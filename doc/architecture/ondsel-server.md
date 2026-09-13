# Architecture notes — Ondsel-Server / "Lens" (`FreeCAD/Ondsel-Server`)

Read against `doc/technical_convergence_plan.md`'s 8-field template. Source
read: `README.md`, `docs/technical.md`, `docs/workflows.md`,
`docs/macro-examples.md`, `backend/package.json`, `backend/src/services/models/models.schema.js`,
`backend/src/services/file/file.schema.js`, `backend/src/services/file/file.js`,
`backend/src/services/file/file.distrib.js`, `backend/src/services/shared-models/shared-models.schema.js`,
`.gitmodules`, `REUSE.toml`/`LICENSES/`. No git commands were run for this
pass; the `FC-Worker` submodule was **not** fetched (empty directory, per
`.gitmodules` pointing at `FreeCAD/FC-Worker.git`), so §5's FC-Worker
details come from `docs/technical.md` prose only, not source.

This project overlaps heavily with prior research already in this repo:
`doc/ondsel_server_issue_48.md`, `doc/ondsel_48_research.md`,
`doc/ondsel_48_response.md`, `doc/ondsel_project_proposal_research.md`
(issue #48's "PDM vs. not-PDM" scope debate) and
`doc/optimistic_locking_research.md` (an academic-literature survey on
CAD merge/locking, not this repo's code specifically). This note is the
source-level companion to those — it doesn't repeat their content, only
cross-references where relevant.

## 1. Repo & basic facts

- **Also called "Lens"** — the platform's product name (see frontend
  README title "Lens Platform"); `Ondsel-Server` is the repo/org name.
- **Language(s)**: JavaScript/Node.js (Feathers.js v5) backend
  (`backend/`, 37 direct npm dependencies, ESM `.js` throughout — no
  TypeScript despite using `@feathersjs/typebox` for schema/validation);
  Vue.js frontend (`frontend/`); a separate **FC-Worker** service
  (Python, FastAPI + Celery, built on FreeCAD core libraries per
  `docs/technical.md`) as a **git submodule that was not fetched** for
  this pass — its actual code was not read.
- **Size/architecture**: substantially larger and more structurally
  mature than EasyPDM or the Omniverse connector — ~25 Feathers "services"
  under `backend/src/services/` (users, organizations, workspaces,
  directories, file, models, shared-models, groups, org-invites,
  preferences, macros, code-runs, publisher, keywords, notifications,
  agreements, site-config, etc.), each following a documented, consistent
  multi-file convention (`docs/technical.md` §"Backend Services
  Structure": `.class.js`/`.schema.js`/`.distrib.js`/`.curation.js`/
  `.shared.js` per service). A dedicated `.distrib.js` convention exists
  specifically to manage denormalized "summary" sub-documents kept in
  sync across collections — an explicit, named pattern for a
  cross-collection consistency problem the other repos in this cluster
  don't formalize at all.
- **Deployment**: multi-container (`docker-compose.yml`: backend,
  frontend, MongoDB, FC-Worker, Celery, Matomo analytics, optional
  Keycloak for local OIDC testing) — this is SaaS-shaped infrastructure,
  not a single-binary self-host story like EasyPDM.
- **License**: **AGPL-3.0-or-later** (`REUSE.toml`/`LICENSES/`,
  SPDX headers throughout, copyright "Ondsel <development@ondsel.com>" and
  named individual contributors per file) — this is the copyleft-strongest
  license in the cluster read so far (EasyPDM: MIT; Omniverse connector:
  BSD-3-Clause). Matters for field 8: an AGPL codebase constrains what
  code (as opposed to protocol/convention) could actually be shared
  downstream without copyleft obligations.
- **Governance signal**: hosted under the `FreeCAD` GitHub org itself
  (unlike EasyPDM/the Omniverse connector, which are independent
  third-party repos) — this is the closest thing in this cluster to an
  "official" FreeCAD-adjacent PDM effort, which is exactly the tension
  `doc/ondsel_server_issue_48.md` documents (the open debate over whether
  Ondsel-Server should scope itself as a PDM system at all).

## 2. Identity/versioning model

The richest, most deliberately-designed version model read in this
series so far — a proper **linear version list embedded per file**, not
a filename convention (EasyPDM) or an external asset-server's checkpoint
log (Omniverse connector):

- A `File` document (`file.schema.js`) has `versions: [fileVersionSchema]`
  — an **array of full version records embedded directly in the parent
  file document** (not a separate collection) — plus a `currentVersionId`
  pointer selecting which one is "active."
- Each `fileVersionSchema` entry carries `uniqueFileName`, `userId`
  (who created it), `message` (an **explicit, user-authored commit
  message** — the only project in this cluster with anything like a real
  commit-message concept), `createdAt`, and `additionalData`.
- Per `docs/workflows.md` §"Version Control of Model/File Workflow": every
  upload creates a new version; version history is browsable per file
  (message, creator, timestamp, current-version highlighted); a user can
  explicitly "**Set as active**" on any past version, which is described
  as *switching* the file's current version rather than creating a new
  one on top — i.e. this is a **pointer move**, not a revert-as-new-commit
  (Git-style), and not a monotonically-increasing revision number
  (EasyPDM-style).
- **Strictly linear — no branching, no merge.** There is nothing in the
  schema or workflow docs resembling a DAG, a branch, or a three-way
  merge. This matters directly for `doc/optimistic_locking_research.md`'s
  framing: Ondsel-Server doesn't attempt any of the academic
  merge/CRDT/feature-reservation approaches surveyed there — it uses the
  simplest possible model (one linear list per file, explicit "become
  current" pointer), consistent with that research's finding that shipped
  tooling universally lags the academic state of the art.
- A separate `Model` document (`models.schema.js`) is explicitly commented
  as "a snapshot in time for a specific combination of: File Version,
  SharedModel (Link), User Parameters" — i.e. Model is a **derived
  render/export cache** (viewer OBJ, thumbnail, STEP/STL/FCStd export
  flags, all with `isXGenerated`/`latestLogErrorId...` bookkeeping for
  FC-Worker's async job results), not itself a source-of-truth version
  record. This separation (raw versioned file vs. derived, regenerable
  model artifacts) is architecturally cleaner than either EasyPDM
  (attachments serve both roles) or the Omniverse connector (STEP/USD pair
  conflated as "the version").

## 3. Conflict/concurrency strategy

**None, by design — optimistic in the weakest sense (no check at all),
not pessimistic locking.** This is a real point of difference from both
EasyPDM (owner/lock fields, admin-overridable) and the Omniverse
connector (session-TOML ownership gate for its Live mode):

- Uploading a new version does not appear anywhere in the schema or
  workflow docs to check or record what version the uploading client
  started from — there is no "expected current version" field submitted
  with an upload, so two users independently uploading "the next version"
  in quick succession would simply produce two new version-array entries
  in whatever order their requests land, one becoming current after the
  other with no conflict signal to either party. This is weaker than even
  a naive optimistic-concurrency check (e.g. a version-token compare
  before accepting a write) — it's closer to plain last-write-wins, just
  with full history retained (nothing is destroyed, unlike a true
  last-write-wins overwrite).
- The one real "lock" concept in the codebase,
  `fileVersionSchema.lockedSharedModels` /
  `VersionFollowTypeMap.locked` (`shared-models.subdocs.schema.js`,
  `file.distrib.js`), is **not** an edit-conflict lock at all — it governs
  whether a **public share link** stays pinned to the specific version it
  was created against ("locked") or always shows whichever version is
  currently active ("active" following). This is a publishing/citation
  concern (matching this specific rendered snapshot to what a viewer
  sees), unrelated to concurrent-editor conflict prevention. Worth
  flagging explicitly since the field name invites the same
  "lock, don't merge" association the census applied to EasyPDM/
  Anchorpoint/the Omniverse connector — but it is a different mechanism
  solving a different problem, and should not be added to that
  `independently_reinvents` cluster in `graph.yaml`.
- Access control is at the **workspace** level (read/write per
  user/group, `docs/workflows.md` §"Workspace Creation Workflow") — this
  gates *who* may write, not *when/whether two concurrent writers
  conflict*. Coarser-grained than EasyPDM's per-item lock.
- Net assessment: Ondsel-Server's actual concurrency story is "rely on
  workspace-level write permissions to keep the number of concurrent
  editors small, and let a full version history absorb the cost of any
  actual collision" — a pragmatic, low-engineering-cost choice, but a
  materially weaker guarantee than either other project in this cluster
  offers, and worth noting as a genuine gap rather than reading it as
  equivalent to their models.

## 4. File format / serialization touchpoints

- **Format-agnostic at the storage layer** — `File`/version records store
  whatever bytes were uploaded (any file type per `docs/workflows.md`
  §"File Upload Workflow"); CAD-specific processing only activates for
  recognized extensions (`fcstd`, `obj`, `step`, `stp` — §"Model Creation
  Workflow"), which is when a `Model` record and FC-Worker job get
  created alongside the plain `File` record.
- **FC-Worker (not fetched, per `.gitmodules`) is the actual FreeCAD
  integration point** — per `docs/technical.md`, it wraps FreeCAD core
  libraries behind a FastAPI service, offloading conversion/render jobs to
  Celery workers. The backend's `Model` schema fields
  (`isExportFCStdGenerated`/`isExportSTEPGenerated`/`isExportSTLGenerated`/
  `isExportOBJGenerated`, `generatedFileExtensionForViewer`,
  `latestLogErrorIdFor...Command`) describe FC-Worker's job surface
  indirectly: it can generate a web-viewer format (OBJ/BREP), plus
  on-demand exports to FCStd/STEP/STL/OBJ, each tracked as an
  independent async job with its own error-log pointer.
- **Server-side scripted geometry inspection** is a capability none of
  the other repos in this cluster have: `docs/macro-examples.md`
  describes a "Run Script" panel in the model viewer that executes
  user-supplied Python against the opened model (via FC-Worker, almost
  certainly using real FreeCAD Python API objects — `obj.Shape`,
  `Part.OCCError`, `doc.Objects` in the example scripts), with a small
  placeholder-substitution DSL (`<selectedObject:N>`, `<objLabel:NAME>`)
  resolved before execution. This is a live, server-side FreeCAD Python
  console over an uploaded document, not just format conversion.
- No indication in the code read of any custom/embedded schema layered
  onto FCStd/STEP themselves (unlike, say, a project that would enrich
  the file with PDM metadata) — properties like title/description/tags
  live in Lens's own MongoDB documents, entirely separate from the CAD
  file's own content.

## 5. Dependencies & integration points

- **Backend**: Feathers.js v5 (a full-featured Node REST+realtime
  framework — auth, hooks, schema validation, socket.io transport all
  first-class) over MongoDB (`@feathersjs/mongodb`, `mongodb` driver
  directly) — a much heavier framework choice than EasyPDM's
  raw-SQL-over-Npgsql minimalism. OAuth support
  (`@feathersjs/authentication-oauth`), email (`feathers-mailer`,
  `nodemailer`), Swagger docs generation, and both local-filesystem and
  S3 storage backends (`@aws-sdk/client-s3`, `aws-sdk`, `s3-blob-store`,
  `feathers-blob`) are all built in.
- **Search**: a self-implemented RAKE (Rapid Automatic Keyword
  Extraction, `node-rake-v2`) keyword-indexing "curation" system per
  service, described in `docs/technical.md`/`docs/workflows.md` — not an
  external search engine (no Elasticsearch/Meilisearch dependency).
- **FC-Worker**: external Python/FastAPI/Celery service, a genuine
  process/language boundary from the main backend (unlike EasyPDM, which
  keeps everything in one ASP.NET process). Not independently verified in
  this pass (submodule not fetched).
- **Analytics**: Matomo, self-hosted, optional Docker Compose profile.
- **Auth**: supports both local email/password and OAuth, plus an
  optional local Keycloak (OIDC) profile for development/testing — a
  materially more sophisticated auth surface than EasyPDM's hand-rolled
  PBKDF2 sessions.
- **No CAD-side client macro of its own read in this pass** — unlike
  EasyPDM and the Omniverse connector, which each ship a FreeCAD-side
  upload/download macro, Ondsel-Server's primary interaction model
  (per `docs/workflows.md`) is browser-based upload/download plus
  server-side script execution against the uploaded file — the "client"
  doing FreeCAD-specific work is FC-Worker, not anything running inside a
  user's local FreeCAD.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:ondsel-server` (line 351):
name "Ondsel-Server (Lens)", category `[pdm, cloud-sharing,
real-time-collab]`, `technical_approach: [sql-database,
server-checkin-checkout]`, `scope: freecad-native`, license "AGPL-3.0",
`first_seen: unknown`. Five edges reference this node (lines 1580, 1703,
1770, 1885, 1892) — not individually re-derived here; see those line
numbers directly for the relationship prose in a graph-consistency pass.

**Correction surfaced by this reading**: `technical_approach:
[sql-database, ...]` does not match the actual stack — `backend/package.json`
declares `mongodb`/`@feathersjs/mongodb` as the database dependency, and
every schema read (`file.schema.js`, `models.schema.js`,
`shared-models.schema.js`) uses embedded-document/array patterns
(`versions: Type.Array(fileVersionSchema)`) idiomatic to MongoDB, not a
relational schema. This is a from-prose-only census entry that a
source-level read directly contradicts — worth fixing in `graph.yaml` to
`technical_approach: [nosql-database, server-checkin-checkout]` (or
similar) in a follow-up graph-maintenance pass, rather than left standing
as-is.

## 7. Friction points observed firsthand

- The prior research (`doc/ondsel_server_issue_48.md` et al.) frames
  Ondsel-Server's scope question as "should this be a PDM system." Read
  from the code: it already **is** one in the narrow sense of this
  plan's own template — it has a versioning model, access control, and a
  model/file separation — but it deliberately omits the two things every
  other project in this cluster treats as PDM-defining: **no numbering
  scheme** (no EasyPDM-style sequential item number) and **no
  concurrency control** (§3 above). It reads as a *file-sharing and
  visualization platform with version history*, not a *part-management*
  system — which is a more precise way to state the scope tension the
  existing issue-48 research already surfaces in prose.
- The `.distrib.js` denormalized-summary convention is a real, portable
  engineering pattern independent of PDM specifics: "each collection
  keeps a read-only summary copy of related collections' key fields,
  refreshed via hooks on the source's own patch/create events." This
  isn't a CAD-specific idea, but it is a clean, named answer to a problem
  every project in this census with more than one collection referencing
  another one has to solve informally (e.g. EasyPDM's `properties.client`
  string-linking, which explicitly avoids foreign keys and therefore
  never goes stale but also never gets summary fields for free).
- The weak/absent concurrency model (§3) is a genuine, verified gap, not
  a documentation oversight — worth treating as a **counter-example**
  rather than supporting evidence when this cluster's "lock, don't merge"
  pattern gets written up, since Ondsel-Server does neither.

## 8. Minimal-patch hypothesis

- **Not a strong fit for the "lock, don't merge" shared shim** (§8 of
  the EasyPDM and Omniverse-connector notes) — Ondsel-Server has nothing
  to plug a shared locking convention into; adopting one would be a
  bigger, non-trivial addition (a real feature, not a small shim) since
  no lock/ownership field exists on `File`/version records today.
- **A stronger, more portable candidate**: the **linear version-list +
  explicit commit-message + "set as active" pointer** model itself
  (`fileVersionSchema`) is a clean, minimal, storage-agnostic schema that
  is more disciplined than EasyPDM's filename-convention versioning and
  simpler than the Omniverse connector's dual-format-plus-checkpoint-token
  scheme. A shared convention proposal for "what does an interchangeable
  file version record look like" (fields: opaque version id, parent file
  id, author, message, timestamp, "is this the active one" flag) could
  plausibly be modeled directly on this schema — cost: **small**, since
  it only requires each project map its own storage onto that shape, not
  change how any of them actually stores bytes.
- **The `.distrib.js` summary-sync pattern** is a genuinely small,
  documentable convention (a short design doc, not code) that any of the
  more relationally-organized projects in this census (EasyPDM's
  manufacturer/material/client name-linking, in particular) could adopt
  without changing their storage engine — cost: **trivial**, since it's
  a documentation/discipline pattern, not a library.
- **Not realistic without first fixing Ondsel-Server's own gap**: any
  cross-project concurrency-control shim (the more valuable prize per the
  plan's stated Cluster-2 friction point) cannot be meaningfully proposed
  *to* Ondsel-Server yet — it would first need *some* form of
  conflict signal (even a minimal "version you started from" field on
  upload) before a shared protocol could plug into anything on this side.
