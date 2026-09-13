# Architecture notes — EasyPDM (`pawelcel/EasyPDM`)

Read against `doc/technical_convergence_plan.md`'s 8-field template. Source
read: `README.md`, `TECHNICAL.md`, `EasyPDM.FreeCad/README.md`,
`db/schema.sql`, `EasyPDM.Api/Endpoints/ItemEndpoints.cs`,
`EasyPDM.Api/EasyPDM.Api.csproj`, `EasyPDM.FreeCad/*.FCMacro`,
`CHANGELOG.md`. No git commands were run for this pass (repo history not
consulted); dates below come from in-repo prose (CHANGELOG, README status
notes), not `git log`.

## 1. Repo & basic facts

- **Language(s)**: C# (.NET 10, ASP.NET Core minimal API, ~8,500 lines
  across `EasyPDM.Api*`) for the backend; TypeScript/React 19 (~18,400
  lines) for the frontend (`EasyPDM.Web/`, Vite + Tailwind v4 + shadcn/ui);
  Python for the FreeCAD macros (`EasyPDM.FreeCad/*.FCMacro`, ~3,100 lines
  combined — unusually large for "macros," effectively a small client
  application); VBA for the SolidWorks counterpart (`.bas`).
- **Size**: mid-sized single-maintainer project. 47 sequential SQL
  migrations (`002`–`048`) plus a from-scratch `schema.sql`; 19 endpoint
  files under `EasyPDM.Api/Endpoints/`; a real integration test suite
  (`EasyPDM.Api.Tests`, xUnit + `WebApplicationFactory` against a live
  Postgres schema).
- **Last real activity**: CHANGELOG's newest entry is `[0.3]`; the FreeCAD
  macro README's own status note is dated 2026-08-27, and the project's
  `first_seen` in `doc/ecosystem/graph.yaml` is 2026-08-04 — this is a very
  recent, still fast-moving project (not consulted via `git log` per this
  session's constraint; these are the dates the repo's own docs state).
- **License**: MIT (`LICENSE`, copyright Paweł Celmer, 2026).
- **Notable provenance**: per its own README, "the whole application was
  written for me by Claude (an AI model from Anthropic)" — the author
  describes themselves as a mechanical design engineer, not a programmer,
  who supplied requirements/descriptions rather than code. Relevant
  context for judging the code's idioms and for the ecosystem census note
  already on this node (`doc/ecosystem/graph.yaml`, `project:easypdm`).
- **Dependencies are deliberately minimal**: backend has exactly two
  NuGet packages (`Npgsql` — raw SQL, no ORM/EF Core at all — and
  `Microsoft.Extensions.Hosting.WindowsServices` for the Windows service
  wrapper). No message queue, no cache layer, no auth framework (sessions
  and PBKDF2 password hashing are hand-rolled in `PasswordHasher.cs` using
  only `System.Security.Cryptography`).

## 2. Identity/versioning model

Single global identity: every Part/Assembly gets a **UUID primary key**
plus a **human-facing sequential integer `item_number`** from one
Postgres sequence (`item_number_seq`), shared across the whole database,
never reused after deletion (a "reclaim the tail" admin tool exists but
only works below the lowest number already in use). No per-project or
per-type numbering — one number space for everything.

**Revisions** are a single integer column (`items.revision_number`) that
only increments when an item cycles back from `wydany`/released or
`anulowana`/cancelled to `w_pracy`/in-progress — not on every file
upload. The letter form (A, B, C…) shown everywhere is a pure display
convention (`revision_label()`, duplicated — deliberately, per the code
comments — in both the frontend `api/types.ts` and the FreeCAD macro) over
that same integer; the database never stores the letter.

Critically, **there is no separate "file version" entity**. The CAD file
itself is just an `item_attachments` row named
`number (name).REVISION.extension` by convention. A new revision doesn't
replace the attachment, it adds another one alongside it — so revision
history is reconstructed by pattern-matching filenames, not by a real
version table (`EasyPDM.FreeCad/README.md`, "Where it gets files from").
A repeat upload of the *same* revision (no status change in between) does
overwrite that one file. This is a fragile-by-design identity scheme:
FreeCAD has no persistent custom property to carry the item ID (unlike
the SolidWorks side, which the README says uses a `EasyPDM_ItemId`
custom property), so the FreeCAD macro recognizes an already-uploaded
document purely from its on-disk **label string** matching
`number (name).REVISION` — acknowledged in the README as "can in rare
cases be wrong" (e.g. FreeCAD's own Save As can copy a label onto an
unrelated file).

## 3. Conflict/concurrency strategy

**Pessimistic, not optimistic** — despite the plan's template using
"optimistic locking" as a reference point, EasyPDM implements old-school
check-out/check-in locking, structurally the same pattern already flagged
in `doc/ecosystem/graph.yaml` as independently reinvented by Anchorpoint
and the Omniverse connector ("lock, don't merge"):

- `owner_id` + `owner_locked` on `items`. Creating a Part/Assembly
  immediately locks it to its creator; while locked, **only the owner can
  edit it — administrators do not bypass this** (a deliberate design
  choice repeated in both README and code comments).
- The race between two simultaneous `/lock` calls is closed with a single
  conditional `UPDATE ... WHERE (owner_locked = false OR owner_id =
  @ownerId OR @isAdmin)` (`ItemEndpoints.cs:961-962`) rather than a
  read-then-write check — this is the closest thing in the codebase to
  optimistic-concurrency style (a compare-and-swap via the `WHERE` clause,
  checked by `rowsAffected`), but it guards *lock acquisition*, not
  arbitrary field edits, and there is no version/`xmin` column guarding
  the rest of the row.
- `/release` has the symmetric race-close (`WHERE (owner_id = @userId OR
  @isAdmin)`), with an explicit code comment explaining why a stale,
  just-rejected release request must not silently clobber a lock someone
  else has since legitimately taken over.
- An item in `wydany`/released status is always unlocked/ownerless and
  uploads to it are rejected outright (not merged, not queued) — the
  FreeCAD macro's changelog explicitly calls out a fixed bug where it used
  to silently flip a reviewed item back to editable and overwrite it.
- No merge tooling anywhere — for the actual 3D file content, conflicts
  are prevented (via the lock), never resolved. This mirrors exactly the
  friction point the technical-convergence plan is looking for evidence
  of in Cluster 2.

## 4. File format / serialization touchpoints

- **Storage**: files sit as plain blobs on disk under a configured
  `StorageRoot` (no format-aware processing) — `item_attachments` records
  point at them. `file_hash`/`file_size` columns exist on `items` but per
  `TECHNICAL.md`'s "Known limitations," there is no upload validation of
  file type or size at all.
- **CAD-native touchpoints are all in the client macros, not the
  server.** The FreeCAD macro:
  - Exports STEP directly from FreeCAD's shape API (`Part.Shape`/exportBrep
    path), explicitly bypassing `Part.export()` because it silently drops
    `App::Link`/assembly containers ("is not a shape" — a fixed bug noted
    in the README).
  - Exports PDF via `Gui.export(...)` on visible objects — explicitly
    *not* going through FreeCAD's TechDraw drawing pipeline, flagged as
    best-effort.
  - As of CHANGELOG `[0.3]`, detects an actual TechDraw drawing page
    (activating it if needed) to distinguish a real technical drawing from
    a rendered 3D snapshot, and separately handles SolidWorks `.SLDDRW`
    files by resolving them to the Part/Assembly they document via their
    views' own model references.
  - Detects assembly structure by walking `App::Link` objects
    (`isDerivedFrom("App::Link")`, deliberately not an exact type match)
    and building a quantity-summed BOM from link/pattern counts.
- **API surface for macros vs. web app is identical** — both go through
  the same REST endpoints; the only CAD-macro-specific mechanism is the
  browser-bridge ticket system (`/api/auth/browser-bridge-ticket`,
  `/api/create-tickets/{ticket}`) that hands off the "new item / duplicate
  / attach existing" decision from the macro to a system-browser tab
  already logged in via a one-time token exchange, so the decision UI is
  never duplicated between the two frontends.
- No embedded/custom schema format of its own (JSON `properties` column
  is the only semi-structured extension point, keyed by ad hoc string
  fields like `properties.rodzaj`, `properties.client`).

## 5. Dependencies & integration points

- **Backend**: ASP.NET Core minimal API + `Npgsql` only, no ORM — every
  query is hand-written SQL. Migrations are embedded resources applied
  automatically on startup (`MigrationRunner.cs`), tracked in
  `schema_migrations`.
- **Frontend**: React 19, Vite, TypeScript, Tailwind v4, shadcn/ui
  ("base-nova" style over Base UI), served as static files from the
  backend's own `wwwroot/` (no separate frontend server in production).
- **CAD integrations**: FreeCAD (Python `.FCMacro`, no workbench/package —
  run directly from FreeCAD's macro menu) and SolidWorks (VBA `.bas`
  macros). Both talk to the same HTTP API and share the session/ticket
  mechanism described above. No plugin/add-on manager integration (e.g.
  not distributed via the FreeCAD Addon Manager).
- **External services**: none beyond PostgreSQL (18) and the filesystem.
  No cloud storage, no external auth provider, no telemetry. Deployment
  is self-hosted-only (Docker images on GHCR, a Linux systemd installer,
  a Windows Inno Setup installer) — there is no SaaS/hosted offering.
- **CI**: seven GitHub Actions workflows, several of which actually
  install the produced artifact on a clean runner (Windows/.exe, Linux
  tarball) rather than only building it — comparatively rigorous CI for a
  project of this size and stated single-author, AI-assisted origin.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:easypdm` (line 647): category
`[pdm, bom]`, `technical_approach: [other]`, status `active`,
`scope: multi-cad-platform`, license MIT, `first_seen: 2026-08-04`. One
edge already recorded: `independently_reinvents` →
`project:anchorpoint` (line 2028), shared idea = "lock, don't merge" as
the concurrency strategy — confirmed directly by this reading (§3 above).
That edge is itself one hop from a second `independently_reinvents` edge,
`project:anchorpoint` → `project:freecad-omniverse-connector` (line 2039),
so all three projects (EasyPDM, Anchorpoint, the Omniverse connector) sit
in the same "lock, don't merge" cluster the census had already flagged
from prose alone — this source-level read corroborates it rather than
revising it.

## 7. Friction points observed firsthand

Confirms the census's framing and adds detail prose alone couldn't
surface:

- The "lock, don't merge" edge is real and specific: it's implemented as
  a single boolean+owner pair with admin override, not a distributed lock
  service or anything more elaborate — i.e. genuinely small, portable
  logic, which matters for judging patch cost in §8.
- The deeper friction the prose census likely undersells: **there is no
  real file-versioning system**, only a filename convention
  (`number (name).REVISION.ext`) layered on an attachments table that was
  never designed as a version store. Any two PDM-ish tools that want to
  interoperate on "what is revision B of this part" would need to agree
  on more than a locking API — they'd need to agree on what a "version"
  even *is*, and EasyPDM's answer today is informal enough (parsed from a
  filename string) that it could not itself be a reliable interchange
  point.
- Identity portability is weak by construction, not by oversight: the
  FreeCAD side has no durable per-document identifier at all (unlike the
  SolidWorks side's custom property) and says so plainly in its own
  README. Any shared convention proposal for "which PDM item does this
  file correspond to" would need to introduce what FreeCAD lacks — a
  persistent, renaming-proof identifier — which is a bigger ask than a
  shim on top of existing state.
- The concurrency model is intentionally non-collaborative at the file
  level (exclusive lock, no merge, no CRDT, no field-level diffing) — so
  a shared "conflict resolution" shim across projects in this cluster
  would really be a shared *locking protocol/API*, not a merge algorithm.
  That's a narrower, more tractable target than the census phrasing
  ("concurrency strategy") might suggest.

## 8. Minimal-patch hypothesis

Two separable opportunities, different costs:

- **A shared "lock/check-out" HTTP convention** (trivial–small cost): the
  actual state machine here is tiny — `owner_id`/`owner_locked` plus two
  endpoints (`/lock`, `/release`) with a conditional-`UPDATE` race guard.
  A one-page convention (endpoint shape, response codes for
  already-locked/insufficient-permission, an "admin override" flag) that
  EasyPDM, Anchorpoint-likes, and the Omniverse connector's checkpoint
  model could each implement against their own storage would cost each
  adopter roughly what EasyPDM's own `/lock`/`/release` pair already
  costs — a handful of endpoints and one boolean+owner column. This is
  the most realistic minimal patch in this cluster: it requires no shared
  library, no shared server, just a documented shape.
- **A shared file-identity/version convention** (moderate–large cost, not
  realistic as a "tiny shim"): EasyPDM's own identity model is too
  informal to export as-is (filename parsing, no durable ID on the
  FreeCAD side) — proposing it as a cross-project standard would mean
  fixing EasyPDM's own weakest point first (giving FreeCAD documents a
  real persistent identifier, e.g. via `App::Document` custom properties
  or a sidecar file) before it could interoperate with anything else.
  Worth flagging as the harder, second-order problem this cluster
  actually has, separate from the locking convention above — matches the
  plan's expectation that field 8 sometimes reveals the census's
  "concurrency" framing was really bundling two different problems
  together.
