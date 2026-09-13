# Architecture notes — CADBase (`gitlab.com/cadbase`, mirrored to GitHub as `cadbaselibrary-freecad`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read across **five repositories**, considerably more than the
plan's own repository list anticipated (which named only
`cadbaselibrary`, dev home `gitlab.com/cadbase`, GitHub mirror
`mnnxp/cadbaselibrary-freecad`): `cadbaselibrary-freecad` (the FreeCAD
workbench client), `cdbs-migrations` (PostgreSQL/Diesel schema),
`cdbs-back` (the Rust/Actix/GraphQL backend), `cadbaselibrary-blender`
(the Blender add-on client), and `three-libs` (static 3D-rendering
assets for the web frontend). No git commands were run for this pass.

CADBase turns out to be the most substantial, most mature, and most
directly PDM-relevant find in this entire census — a real, actively
developed, multi-client commercial platform with a proper relational
backend, not a hobby workbench. It is also the first project read here
that ships **two independent CAD-tool clients (FreeCAD and Blender)
sharing the same identity/versioning model against the same backend**,
which makes it a uniquely strong data point for the plan's central
question: what does a genuinely portable, cross-tool PDM convention
actually look like once someone has built one for real.

## 1. Repo & basic facts

- **A five-repository product**, not a single addon: `cdbs-back` (Rust,
  Actix-web, `async-graphql`, Diesel ORM — the API server),
  `cdbs-migrations` (PostgreSQL schema, Diesel migration format,
  PostgreSQL License, "synthetic test data only" per its own README),
  `cadbaselibrary-freecad` (LGPL-3.0, requires FreeCAD ≥0.21.0),
  `cadbaselibrary-blender` (LGPL-3.0, requires Blender ≥4.2.0), and
  `three-libs` (vendored static assets, license not independently
  checked — mostly third-party code, see §4).
- **First real migration is dated 2021-06-29** (`cdbs-migrations`) —
  `doc/ecosystem/graph.yaml`'s `first_seen: "2018"` is earlier than
  anything found in any of the five repos' own content; this may
  reflect the company/domain's actual founding date (unverifiable from
  source alone) rather than when this codebase existed. Flagged, not
  corrected (see §6).
- **License is genuinely mixed across the product, not a single
  value**: `cdbs-back`'s README states contributions are "licensed
  under the AGPL v3 license"; both CAD-tool clients display an
  LGPL-3.0 badge; `cdbs-migrations` uses the PostgreSQL License.
  `doc/ecosystem/graph.yaml` currently records `license: unknown` —
  corrected in this pass to reflect the mix (see §6).
- **Real production engineering discipline throughout**: `cdbs-back`
  has `.pre-commit-config.yaml`, `rustfmt.toml`, a Jest-based
  integration-test suite (`tests/*.test.js` — `component`, `discussion`,
  `rbac`, `relate`, `service`, `standard`, `user`, `api_key`) run
  against the real GraphQL API rather than mocks, and a
  repository/service/model layered architecture applied uniformly
  across every domain object. This is the most disciplined backend
  codebase read in this entire census.
- **Confirmed Russian origin** (per `doc/ecosystem/graph.yaml`'s
  existing note, via a 2023 Habr.com article) — not independently
  re-verified in this pass, but consistent with Russian-language code
  comments found directly in `cdbs-back`'s access-control logic (§3)
  and the bundled `cadbaselibrary_ru.qm`/`.ts` translation files in the
  FreeCAD client.

## 2. Identity/versioning model

**The richest, most deliberately layered identity/version scheme in
this entire census — a real four-level hierarchy with an explicit
revision chain, independently confirmed at both the database-schema
and backend-logic layers, and reused nearly verbatim across two
different CAD-tool clients.**

- **Hierarchy**: `Component` (a part/design, UUID, can itself have a
  `parent_component_uuid` for sub-part/variant relationships) →
  `Modification` (a named design variant/revision of the component,
  UUID, its own `parent_modification_uuid` chain) → `Fileset` (one per
  `program_id` — i.e. FreeCAD and Blender each get their **own**
  fileset under the same modification, `UNIQUE(modification_uuid,
  program_id)`) → `File` (UUID, `sha256_hash`, filename, size,
  presigned download URL). Every root-level component/modification
  points at a configured system UUID that self-references rather than
  using a nullable parent column (`ROOT_COMPONENT_UUID`, etc., set in
  `cdbs-back`'s `.env` and cross-checked against the DB) — a slightly
  unusual but consistent way to avoid nullable foreign keys for tree
  roots.
- **File-level versioning has a real linked chain, not just a
  counter**: `file_ref.revision INTEGER` (a display counter) *and*
  `file_ref.parent_file_uuid` (a self-referencing FK, `ON DELETE
  CASCADE`, pointing at the file row this one supersedes) *and* a
  `commit_uuid` FK into a small separate `commit_ref` table holding
  just a `commit_msg`. This combines EasyPDM's flat revision counter,
  Ondsel-Server's per-version commit message, and something closer to
  git's parent-pointer model, all in one schema — the most complete
  synthesis of this census's various version-record shapes found
  anywhere.
- **"Active revision" is a boolean flag flip, not a separate pointer
  column** — confirmed directly in `cdbs-back`'s
  `set_active_revision_by_uuid` (`src/models/relate_ref/file/service/update.rs`):
  a file's row has `is_hidden`; the *active* revision for a given
  object is the one with `is_hidden = false`, and every other revision
  of the same filename is hidden. Reverting to an older revision is
  literally: unhide the target, then hide whatever was previously
  unhidden. **This is functionally identical to Ondsel-Server's
  `currentVersionId` pointer-move pattern**, independently arrived at
  on a completely different stack (Rust/Postgres/boolean-flag vs.
  Node/MongoDB/explicit-pointer-field) — a genuine, strong
  cross-project convergence worth treating as validated best practice
  for "what does reverting to a past version mean" in any future
  synthesis.
- **A real, if narrow, transactional gap in that revert path**: the
  unhide-then-hide pair in `set_active_revision_by_uuid` is two
  separate `UPDATE` statements, not wrapped together in an explicit
  database transaction in the code read — a crash or concurrent write
  between them could transiently leave zero or two "active" revisions
  for the same filename. Narrow, easy to fix (wrap in a transaction),
  but a concrete, findable correctness note worth surfacing.
- **A genuine, systematic audit trail**: `component_history_list`
  (and matching tables for `standard`/`service`/`user`) logs
  `type_of_change_id`, `user_uuid`, `old_data` (the pre-change value —
  an undo-log-style record), and `changed_at` on every tracked
  mutation. More systematic than any audit mechanism elsewhere in this
  census, including BCF's own creation/modification-date-only model.
- **The identity/versioning model is confirmed shared, not just
  similarly designed, across two CAD-tool clients**: diffing
  `cadbaselibrary-freecad`'s and `cadbaselibrary-blender`'s
  `CdbsModules/QueriesApi.py` files directly shows they are **identical
  except for two lines** (an import-style difference and the
  `programId` constant — `42` for FreeCAD, `53` for Blender). Every
  GraphQL query and mutation string — `component_modifications`,
  `target_fileset`, `fileset_files`, `upload_files_to_fileset`
  (carrying a `commitMsg`), `upload_completed`,
  `delete_files_from_fileset` — is byte-for-byte the same. This is the
  single strongest piece of evidence in this whole census that a
  cross-CAD-tool identity/versioning convention is not just
  theoretically possible but has already been built, shipped, and
  proven to generalize with a one-constant change per additional tool.

## 3. Conflict/concurrency strategy

**A genuinely sophisticated, multi-tier *authorization* model sits on
top of the exact same last-write-wins upload gap found in every other
"no real concurrency control" project in this census — confirmed
independently at both the client and the schema/backend layers.**

- **Client-side upload logic** (`cadbaselibrary-freecad`'s
  `CdbsStorage.py`, read in full): `processing_manager()` fetches the
  cloud's current fileset listing once, `define_files()`/
  `parsing_duplicate()` diff that snapshot against local disk by
  SHA-256 hash, and `processing_update()`/`delete_old_files()` then
  delete/upload accordingly — with no check anywhere that the cloud
  fileset is still in the state it was in when first fetched. Two
  users editing the same modification's fileset concurrently would
  each independently diff against a stale snapshot and silently
  overwrite/delete each other's changes, exactly the pattern already
  flagged as a live risk in the standalone `freecad-cloud-browser`
  projects (§3 of that architecture note) — except here it sits
  underneath a considerably more capable product.
- **No lock/session/checkout table anywhere in the database schema**
  (`cdbs-migrations`, every migration searched) — confirmed
  independently of the client-side reading. The schema has `is_hidden`/
  `is_delete` flags and a full audit trail, but nothing resembling
  EasyPDM's `owner_id`/`owner_locked`, GitPDM's presence branch, or any
  other check-out mechanism found elsewhere in this census.
- **A real, elaborate RBAC layer governs *who may write*, not
  *whether concurrent writers collide*** — `cdbs-back`'s
  `check_access_component_for_user` (`src/models/component/access/util.rs`)
  cascades through three tiers: component ownership
  (`component_ref.user_uuid`), direct per-user grants
  (`user_access_to_component`), and company-membership-plus-role
  grants (`company_access_to_component` joined against
  `company_member_list` and `role_access`). This is the most elaborate
  authorization model in the whole census — multiple individual users
  *and* whole companies can simultaneously hold write access to one
  component — which makes the absence of any conflict-prevention
  mechanism more consequential here than in a single-owner system like
  EasyPDM's: **the more people who can legitimately write to the same
  fileset at once, the more this gap matters in practice.**
- **Net assessment**: CADBase demonstrates that a product can build a
  genuinely mature, production-grade *authorization* story
  (who-can-write) while still leaving the *concurrency* story
  (what-happens-when-two-writers-collide) essentially unaddressed —
  the two problems are more separable than this census's earlier
  entries made them look, and CADBase is the clearest illustration yet
  that solving one doesn't imply solving the other.

## 4. File format / serialization touchpoints

- **The FreeCAD/Blender clients never parse CAD geometry themselves**
  — files are opaque blobs, matched by SHA-256 hash and filename;
  identity/versioning is entirely a metadata-layer concern (§2), the
  same design choice OpenPartsLibrary made for a very different reason
  (a parts catalog, not a version-controlled workspace).
- **The web frontend does real, custom client-side CAD parsing that no
  other project in this census attempts**: `three-libs`'s
  `STEPLoader.js` wraps `occt-import-js` (a WebAssembly build of the
  real OpenCascade Technology kernel) running inside a Web Worker, so
  STEP files are parsed into renderable geometry **entirely in the
  browser**, with no server-side conversion step for the viewer. This
  is architecturally distinct from Ondsel-Server's FC-Worker (a
  server-side FreeCAD/Celery conversion pipeline) and from the
  Omniverse connector's STEP→USD tessellation-at-upload-time approach
  — CADBase pushes the actual geometry-kernel work to the client's own
  browser, presumably to reduce server load and support the "air-gapped
  / on-premises / offline" deployment goal `three-libs`'s own README
  states as its purpose.
- **`three-libs` is mostly vendored, not custom** — worth a clean
  correction to how this repo's own GitLab description characterizes
  it ("Shared 3D graphics core and model parsers submodule"): the bulk
  of its content is unmodified third-party code (three.js builds,
  web-ifc/OpenBIM WASM bindings, Draco decoders, GLTF/STL/GCode
  loaders) bundled locally specifically for offline/air-gapped
  deployment, not custom parsing logic. The two genuinely
  CADBase-authored pieces found are small: a 51-line LRU-style
  in-memory model cache (`model-cache.js`, evicting the oldest fetched
  model buffer once 20 are held) and the thin Web Worker wrapper
  around `occt-import-js` in `STEPLoader.js`.
- **Presigned S3 URLs are the actual upload/download transport**
  (`presigned_url_ref` table in the schema; `uploadUrl` returned by the
  `uploadFilesToFileset` mutation) — files never pass through the
  GraphQL API's own request/response bodies, only their metadata and
  temporary storage URLs do.

## 5. Dependencies & integration points

- **Backend**: Rust, Actix-web, `async-graphql`, Diesel ORM over
  PostgreSQL, JWT auth (RS256, keys generated locally per the README's
  own setup instructions), S3-compatible object storage. A genuinely
  modern, well-chosen stack for this kind of service — no other
  project in this census uses Rust at all.
- **Two independent CAD-tool clients sharing one backend and one
  Python module layout** (§2) — the strongest existing proof in this
  census that a PDM-style backend can be genuinely CAD-tool-agnostic in
  practice, not just in aspiration.
- **A dedicated migrations repository, separate from the backend code
  that consumes it** (`cdbs-migrations`) — a clean separation of
  concerns (schema evolution vs. application logic) not seen elsewhere
  in this census, where every other project keeps its schema/migrations
  inside the same repo as its server code.
- **The web frontend's rendering stack is fully self-hosted for
  offline/air-gapped deployment** (§4) — a deliberate infrastructure
  choice distinguishing CADBase from every other cloud-shaped project
  in this census, none of which mention air-gapped operation as a
  design goal.
- **No CAD-side macro system was found for either client comparable to
  EasyPDM's assembly-tree auto-detection** — both clients' upload flow
  operates on whatever the user has already placed in the relevant
  local fileset folder, not by walking the active document's own
  in-memory link/assembly graph. This is a real, if minor, capability
  gap relative to EasyPDM's and the Omniverse connector's more
  automated macros.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:cadbaselibrary` (line 748):
name "CADBaseLibrary (mnnxp/cadbaselibrary-freecad + gitlab.com/cadbase)",
category `[pdm, bom, cloud-sharing]` (all three confirmed accurate by
this five-repo read — arguably the single project in this census that
most cleanly earns all three tags at once), `technical_approach:
[other]`, `scope: freecad-native`, origin RU (confirmed consistent with
Russian-language code comments found directly in `cdbs-back`), license
`unknown`, `first_seen: "2018"`, existing note correctly identifying
GitLab as the real dev home.

**Corrections applied to `graph.yaml` in this pass** (see below):
- `technical_approach: [other]` → `[sql-database, server-checkin-checkout]`
  — confirmed a real PostgreSQL/Diesel backend (`cdbs-migrations`,
  `cdbs-back`) and an upload/download-to-central-server model (both
  clients), matching the vocabulary already used for Ondsel-Server and
  the Omniverse connector.
- `scope: freecad-native` → `multi-cad-platform` — confirmed two
  independent CAD-tool clients (FreeCAD, Blender) against the same
  backend, the same vocabulary value already used for OpenPartsLibrary
  and the Omniverse connector.
- `license: unknown` → a mixed-license note, since no single SPDX value
  accurately describes the product: LGPL-3.0 for both CAD-tool clients,
  AGPL-3.0 for the backend, PostgreSQL License for the migrations
  repository.

**Flagged, not corrected**: `first_seen: "2018"` predates anything
found in any of the five repositories' own content (`cdbs-migrations`'s
earliest real schema migration is dated 2021-06-29) — this may reflect
the company/platform's founding or domain-registration date from the
prose census source rather than this codebase's actual age; left as-is
pending a check against that source, since a repo-internal read cannot
resolve which date the field is actually meant to track.

## 7. Friction points observed firsthand

- **This is the single best-evidenced example in the entire census of
  a working, production-grade, cross-CAD-tool PDM backend** — every
  other project either serves one CAD tool (EasyPDM, GitPDM,
  HistoryWorkbench, the Omniverse connector) or serves no specific tool
  at all (OpenPartsLibrary, `ose-vcs-library`). CADBase is direct,
  concrete proof that the plan's underlying premise — a shared
  identity/versioning convention that multiple independent CAD-tool
  integrations can adopt with minimal per-tool cost — is not
  speculative; it has already been built and shipped, at the cost of
  essentially one integer constant per additional CAD tool (§2).
- **The "active revision = unhide one, hide the rest" pattern**,
  independently convergent with Ondsel-Server's `currentVersionId`
  pointer, is now confirmed across two unrelated stacks and should be
  treated as settled, low-risk best practice for any future "what does
  a version record need" proposal in this series — alongside the
  already-repeated "never destroy prior state" principle this census
  keeps finding everywhere.
- **The authorization/concurrency separation (§3)** is the most
  important structural lesson from this specific project: CADBase
  proves a team can build a genuinely excellent access-control system
  (multi-tier, company-aware, role-based) without that effort touching
  the concurrent-write-conflict problem at all. Any future proposal
  addressing Cluster 2's friction point should treat these as two
  separate design problems requiring two separate solutions, not
  assume solving one buys progress on the other — CADBase is the
  clearest evidence yet that they don't automatically travel together.
- **The client-side WASM CAD-kernel approach in `three-libs`** (§4) is
  a distinct third strategy for "how does a non-CAD-native environment
  render CAD geometry," alongside Ondsel-Server's server-side FC-Worker
  and the Omniverse connector's upload-time tessellation — worth citing
  as a third option (client-side WASM kernel) if a future synthesis
  ever needs to compare deployment-cost tradeoffs for CAD-in-the-browser
  approaches.

## 8. Minimal-patch hypothesis

- **The shared `QueriesApi.py`-style client module, parameterized by
  one `program_id` constant, is the strongest existing template in
  this entire census for what a genuinely portable, per-CAD-tool
  integration shim looks like.** Cost to add a third CAD tool to this
  exact system: register a new `program_id` in `program_ref`, copy the
  existing client module structure, adjust CAD-specific file-open/save
  hooks. This is a working existence proof, not a proposal — any future
  recommendation in this series arguing "a shared identity/versioning
  convention is cheap to adopt per additional CAD tool" can point
  directly at this as precedent.
- **The revert-to-previous-revision transactional gap (§2)** is a
  small, concretely locatable, low-risk fix: wrap
  `set_active_revision_by_uuid`'s two flag flips in a single database
  transaction. Cost: **trivial** — a scoping change to existing,
  working code, not a new mechanism.
- **The authorization/concurrency separation (§3, §7)** suggests the
  most valuable next step for CADBase specifically (not a
  cross-project convention, a recommendation for this product) would
  be layering a presence-or-lock mechanism analogous to GitPDM's
  advisory-presence design on top of its already-excellent RBAC system
  — the access-control foundation to build on already exists here more
  solidly than in any other project in this census.
- **Not a source of a new concurrency pattern for the plan's own
  cross-project shim proposal** — CADBase's gap here is the same one
  already characterized in the standalone cloud-browser and Ondsel-
  Server notes; its main contribution to that specific question is
  confirming the gap's prevalence, not offering a new way to close it.
