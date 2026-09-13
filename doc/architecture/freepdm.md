# Architecture notes — FreePDM (`grd/FreePDM`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read: `README.md`, `doc/FreePDM_02-Workflows/02-CheckoutFile.md`,
`internal/domain/models/{lock,version}.go`, `internal/ports/locks/*`,
`internal/adapters/README.md`, `internal/db/item.go`,
`internal/db/init.go`, `internal/vault/localfs/file_index.go`,
`internal/server/router.go`, and a size/content check of
`addons/FreeCAD_Addon/`. No git commands were run for this pass.

FreePDM is the most extensively **designed** project in this entire
census — a dedicated `doc/` tree of numbered design chapters
(requirements, workflows, attributes, DB versioning, architecture,
roadmap) rivaling a real engineering spec — but, per its own README's
opening disclaimer ("ATM only the server is working... an alpha
prototype GUI"), a project whose actual implementation lags well behind
its design maturity, especially at exactly the layer this plan cares
about most: locking and versioning are fully specified as clean
interfaces, with essentially no concrete implementation behind them yet.

## 1. Repo & basic facts

- **Language**: Go (79 `.go` files), a proper hexagonal/ports-and-
  adapters architecture (`internal/ports/{locks,sync,vaults,vaultfs,params}`
  define interfaces; `internal/adapters/` is explicitly documented to
  hold their concrete implementations — see §3) plus a Fyne-based
  desktop GUI prototype (`apps/fpg`, "FreePDM GUI") and a near-empty
  FreeCAD addon skeleton (`addons/FreeCAD_Addon/freecad/FreePDM/`, ~50
  lines total across all Python files).
- **Self-aware participant in this exact census's subject matter**: the
  README's own "Previous attempts" section explicitly cites OpenPLM
  (abandoned), a German FreeCAD-forum PDM thread, Taack PLM, **Ondsel**
  ("A very promising design. Unfortunately it shut down."), and
  NanoPLM — i.e. FreePDM's authors have independently surveyed almost
  exactly the same landscape this technical-convergence plan and its
  predecessor census documents map, and are building explicitly in
  response to Ondsel-Server's shutdown. Worth flagging as a live,
  first-party confirmation that the fragmentation problem this whole
  research series documents is recognized *from inside* the ecosystem,
  not just observed from outside it.
- **License**: MIT — confirmed, matches `doc/ecosystem/graph.yaml`.
- **Copyright headers mix 2023 and 2025** across different files
  (`internal/vault/localfs/file_index.go`: "Copyright 2023"; the
  `internal/domain/models/` and `internal/ports/locks/` files: "Copyright
  2025") — consistent with `doc/ecosystem/graph.yaml`'s `status: revived`
  (as of 2025-02-18): an older filesystem/vault layer from 2023 appears
  to have been kept, with a newer, more deliberately-architected
  ports/domain layer added on top starting 2025.
- **A striking artifact confirming an even earlier Python prototype**:
  `internal/db/item.go`'s stub method bodies contain comments like
  `// raise NotImplementedError("Function update_item is not implemented
  yet")` — `raise`/`NotImplementedError` is Python syntax, not Go,
  strongly suggesting this file is a line-by-line port of an earlier
  Python version of FreePDM that was never finished being translated
  into working Go logic, just scaffolded with the original comments
  intact.

## 2. Identity/versioning model

**Two layers at very different levels of completeness: a working,
primitive filesystem index, and a well-designed but unimplemented
content-hash version model.**

- **What actually works today**: `internal/vault/localfs/file_index.go`
  maintains a global, incrementing `ContainerNumber` (an EasyPDM-style
  sequential ID) recorded in a plain-text `FileList.csv` (colon-
  delimited, hand-rolled CSV I/O) mapping container numbers to
  filenames and directory paths, with **one level of rename/move
  history** (`PreviousName`/`PreviousPath` fields, overwritten on each
  subsequent rename — not a full history, just "what was it called
  before this one change"). This is a real, working, if primitive,
  identity layer — closer in spirit to EasyPDM's item-number sequence
  than to any database-backed scheme, but persisted as a flat CSV file
  rather than a real table.
- **A real, configurable relational layer also exists**
  (`internal/db/init.go`): GORM ORM over either SQLite (dev/test,
  confirmed via `db_test.go`'s in-memory SQLite setup) or PostgreSQL
  (production, via a DSN). `doc/ecosystem/graph.yaml`'s
  `technical_approach: [custom-filesystem-permissions]` undersells this
  — a real SQL backend is wired up, alongside the CSV-based vault index
  (see §6).
- **The designed-but-unimplemented version model**
  (`internal/domain/models/version.go`) is clean and, notably, **content-
  hash-based rather than filename-based**: `Version{ID, Vault, RelPath,
  ContentHash, Size, Author, CreatedAt, Label}` — identity of a version
  is tied to its content hash, with a free-text `Label` field
  (`"WIP"`, `"Released"`) rather than a fixed status enum. This is a
  cleaner, more modern design than either EasyPDM's filename convention
  or Ondsel-Server's array-of-versions, but **there is no concrete
  adapter implementing storage or retrieval of `Version` records
  anywhere in the codebase read** — it exists purely as a domain type
  with no service, repository, or persistence behind it yet.
- **The actual PDM check-in/check-out/release-state logic
  (`internal/db/item.go`'s `OwnerStates`/`ReleaseStates` types) is
  entirely unimplemented** — every method (`CheckIn`, `CheckOut`,
  `CheckInCheckOut`, `New`, `Prototype`, `Release`, `Depreciated`) is an
  empty function body with only descriptive comments of what it should
  eventually do. The `Item` struct itself has no version/revision field
  at all. This is the clearest evidence that FreePDM's actual
  identity/versioning *behavior* — as opposed to its *design* — does
  not yet exist in working code.

## 3. Conflict/concurrency strategy

**The most thoroughly *designed* pessimistic-locking model in this
entire census — genuinely synthesizing features from both sides of the
"lock, don't merge" debate this project's own README implicitly engages
with — but, like §2, entirely unimplemented as of this snapshot.**

- `internal/ports/locks/locks.go` defines a `Service` interface with
  five methods: `Status`, `Checkout(vault, rel, who, ttlMinutes)`,
  `Heartbeat(vault, rel, who)`, `Checkin(vault, rel, who)`,
  `ForceUnlock(vault, rel, admin, reason)`. The accompanying
  `internal/domain/models/lock.go` defines `Lock{Vault, RelPath, Holder,
  Since, ExpiresAt, Status, Note}` with `Status` one of `"locked"` /
  `"stale"` / `"released"`.
- **This single interface design synthesizes elements from across the
  whole census's concurrency-strategy spectrum**: a TTL-based lease
  (`ExpiresAt`) and explicit `"stale"` status echo GitPDM's
  `STALE_PRESENCE_SECONDS`/`STALE_LOCK_SECONDS`; a `Heartbeat` method
  matches GitPDM's presence-heartbeat mechanism almost exactly; but
  unlike GitPDM (which explicitly built and then *tore out* real
  locking in favor of advisory-only presence), FreePDM's design commits
  to genuine **pessimistic, exclusive locking** — `Checkout` is
  documented as "get write-access," matching EasyPDM's/Anchorpoint's/
  the Omniverse connector's approach — while also including
  `ForceUnlock(admin, reason)`, matching EasyPDM's admin-override-a-
  stuck-lock pattern precisely. No other project in this census
  combines a lease/heartbeat/staleness model (GitPDM's contribution)
  with genuine pessimistic exclusivity and admin override (the EasyPDM/
  Anchorpoint cluster's contribution) in one design.
- **`internal/ports/locks/README.md` is explicit that this is
  interface-only**: "Does NOT contain: Lock storage/DB/ACL mechanics."
  `internal/adapters/README.md` lists `memorylocks` (an in-memory dev/
  test implementation) and a DB-backed implementation as **"(future)"**
  examples — i.e., as of this snapshot, **no concrete implementation of
  `locks.Service` exists anywhere in the repository**, confirmed by a
  repo-wide search finding zero non-interface references to the type.
- **The design documentation itself shows live, unresolved debate about
  whether/when to implement this at all** — `doc/FreePDM_02-Workflows/02-CheckoutFile.md`
  includes an inline exchange between two contributors: one guesses
  "I assume this is not something we implement in one of the first
  states (probably state >4)," while the other sketches a concrete UI
  (per-item checkboxes, Check Out/Check In buttons, an
  owner-indicator icon) as a specific proposal. This is a rare,
  directly-observable instance of a design conversation this census
  otherwise has to infer entirely from finished code — worth treating
  as first-hand evidence of how contentious/deprioritized the locking
  question is even among a team explicitly building a PDM system.
- **What's actually reachable today, per the server's own route table**
  (`internal/server/router.go`): authentication (login/logout),
  session-gated admin user management, and **read-only** vault browsing
  (`VaultsListGet`, `VaultBrowseGet`, `VaultPathBrowseGet`). Upload and
  delete routes exist only as commented-out placeholders
  (`// r.Post("/admin/vault/{vaultID}/upload", ...)`). There is,
  functionally, no live check-out/check-in/lock endpoint anywhere in
  the running server as of this snapshot — matching the README's own
  "ATM only the server is working" framed narrowly (auth + browse), not
  broadly (the actual PDM workflow).

## 4. File format / serialization touchpoints

- **No FCStd-aware logic exists in the FreeCAD addon** — it is
  essentially an empty skeleton (`init_gui.py`: 45 lines of workbench
  registration boilerplate; `version.py`: empty; a placeholder
  `my_numpy_function.py`). There is nothing here comparable to
  EasyPDM's or the Omniverse connector's assembly-walking macros to
  analyze.
- **`doc/TestFiles/`** bundles real sample `.FCStd`/`.FCStd1` files
  (sequentially numbered `0001.FCStd` through `0006.FCStd`, plus a
  library part and a manufacturer's SKF bearing model with an attached
  datasheet PDF) — evidence of deliberate, realistic test-fixture
  planning even though the corresponding upload/versioning code isn't
  built yet.
- **Design docs go deep on the *intended* file-handling model**
  (`FreePDM_02-Workflows/01-FileStoringDuringEditing.md`,
  `06-DbShape.md` — not read in full detail in this pass, titles alone
  indicate real design intent around where working copies live during
  editing and how the DB schema should shape file relationships) —
  worth a closer read in a future pass if this project's design
  documents ever become a citation source for a synthesis proposal,
  since the documentation appears more complete than the code across
  this entire area.

## 5. Dependencies & integration points

- **GORM** (ORM) over SQLite or PostgreSQL, configurable per
  deployment — a real, working piece of infrastructure independent of
  the unimplemented domain logic layered on top of it.
- **Chi router** (inferred from `r.Get`/`r.Post`/route-group syntax in
  `router.go`) for the HTTP server.
- **Fyne** (cross-platform Go GUI toolkit) for the desktop client
  prototype (`apps/fpg`) — a genuinely different GUI technology choice
  from every other project in this census (all Qt/PySide-based, since
  every other GUI-having project is a FreeCAD/Blender addon).
  Deliberately chosen per the README ("multi platform app") as a
  standalone application rather than embedding inside FreeCAD itself.
- **Docker Compose deployment** (`docker-compose.yml`, `Dockerfile`)
  and dedicated CLI tools (`cmd/createvault`, `cmd/removevault`,
  `cmd/pdmserver`, `cmd/role-migration-tool`) — real, if modest,
  operational tooling for a project still short on core PDM behavior.
- **CodeQL security scanning** configured in CI
  (`.github/workflows/codeql-analysis.yml`) — a real, if standard,
  security-hygiene practice not observed in most other hobby-scale
  projects in this census.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:freepdm` (line 283): category
`[pdm]`, `technical_approach: [custom-filesystem-permissions]`, status
`revived` as of 2025-02-18 (consistent with the 2023/2025 copyright-
header split found in this read — see §1), `scope: freecad-native`,
license "MIT" (confirmed), `first_seen: 2022-04-28`.

**Correction applied to `graph.yaml` in this pass** (see below):
`technical_approach: [custom-filesystem-permissions]` →
`[custom-filesystem-permissions, sql-database]` — the existing tag
correctly captures the working `FileList.csv`/container-number vault
index (§2), but the read also confirmed a real, separately-configured
GORM/SQLite/PostgreSQL backend (`internal/db/`) that the single
existing tag doesn't reflect; both are genuinely present, not
alternatives.

No correction to `status: revived` — this read's own evidence (mixed
2023/2025 copyright headers, an unimplemented-but-well-designed
ports/domain layer added on top of an older working vault layer)
directly supports that characterization rather than contradicting it.

## 7. Friction points observed firsthand

- **FreePDM is, to date, the single clearest illustration in this
  census of the gap between designing a concurrency model and building
  one.** Every other project read either has a working (if sometimes
  flawed) concurrency mechanism (EasyPDM, GitPDM, CADBase, the
  Omniverse connector) or has made a deliberate, evidenced choice not
  to need one (`ose-vcs-library`, OpenPartsLibrary, BCF). FreePDM has
  neither yet — it has a *specification* good enough that it could
  plausibly serve as a shared interface contract for this whole
  cluster, sitting on top of no implementation at all. This is a
  genuinely distinct position worth naming explicitly rather than
  folding into either "solved" or "doesn't need it."
- **The README's explicit citation of Ondsel-Server's shutdown as
  direct motivation** is a first-party data point corroborating this
  census's own framing (`doc/ondsel_server_issue_48.md` and related
  documents) that the FreeCAD PDM ecosystem is aware of its own
  fragmentation and repeated-reinvention pattern — FreePDM isn't a
  naive fifth reinvention, it's a self-described attempt to learn from
  the specific prior attempts this whole research series has been
  mapping.
- **The `locks.Service` interface (§3) is, on its own design merits,
  arguably the single most reusable *design artifact* (not working
  code) surfaced anywhere in this census for the plan's Cluster 2
  question** — a five-method contract combining lease/heartbeat/
  staleness (GitPDM's contribution) with exclusive checkout and admin
  override (the EasyPDM/Anchorpoint/Omniverse-connector cluster's
  contribution). Its value here is entirely as a *specification* to
  evaluate or adopt, precisely because it has no implementation
  baggage yet to work around.

## 8. Minimal-patch hypothesis

- **Not a source of working code to adopt** — almost everything
  directly relevant to Cluster 1/2's questions in this repository is
  either a primitive-but-working piece (the CSV file index, §2) too
  narrow to generalize, or a well-designed interface with zero
  implementation (`locks.Service`, `Version`) that would need to be
  built, not borrowed.
- **The `locks.Service` interface shape itself is worth treating as a
  candidate shared specification**, independent of this project's own
  implementation status: `Status/Checkout/Heartbeat/Checkin/ForceUnlock`
  with a `Lock{Holder, Since, ExpiresAt, Status, Note}` record is a
  clean, minimal, storage-agnostic contract that synthesizes this
  census's two competing concurrency philosophies (advisory-with-
  staleness vs. exclusive-with-override) into one design. If a future
  synthesis document proposes a shared locking convention for Cluster
  2, this interface — evaluated on its design merits, not as "adopt
  FreePDM's code" — is the strongest single reference point found in
  this entire series. Cost to *use* it as a reference: **zero** (it's
  already written down); cost to *implement* it for real: unknown,
  since FreePDM's own team hasn't done so yet either.
- **A concrete, low-cost recommendation for FreePDM itself** (not a
  cross-project patch): given how much design effort already exists
  around the lock/version domain models, the highest-leverage next
  step for this specific project would be building the `memorylocks`
  adapter explicitly flagged as "(future)" in `internal/adapters/README.md`
  — a working, in-memory reference implementation of `locks.Service`
  would let the rest of the design (heartbeat, staleness, force-unlock)
  actually be exercised and validated before committing to a
  DB-backed version, and would close the single largest gap between
  this project's design maturity and its implementation maturity.
