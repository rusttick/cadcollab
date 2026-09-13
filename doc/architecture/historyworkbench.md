# Architecture notes — HistoryWorkbench (`eblanshey/HistoryWorkbench`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read: `README.md`, `package.xml`, `pyproject.toml`, and source
across `freecad/history_wb/{domain,application,infrastructure}/` —
specifically `domain/snapshots/models.py`, `domain/git/git_service.py`,
`domain/git/ports.py`, `domain/diff/engine.py`,
`infrastructure/git/git_port_adapter.py`. No git commands were run for
this pass (ironic given the subject, noted for completeness).

This is the first repo in **priority group 2** (the git/version-control
cluster) rather than group 1, and it is already independently flagged in
`doc/ecosystem/graph.yaml` as "by far the highest-star project in the
whole census (144 stars)." That signal is corroborated, not just
repeated, by this source-level read: this is a substantially more
mature, better-architected codebase than any of the priority-group-1
repos except possibly Ondsel-Server.

## 1. Repo & basic facts

- **Language(s)**: Python only (~16,500 lines under
  `freecad/history_wb/`, excluding tests) — a proper FreeCAD Addon
  Manager package (`package.xml`), not a bare macro. YAML is used
  as the snapshot serialization format (PyYAML is the addon's one
  declared dependency, `package.xml`).
- **Architecture**: explicit **hexagonal/ports-and-adapters** layering —
  `domain/` (pure logic: git service, diff engine, snapshot models, tree
  comparison — no FreeCAD or subprocess imports), `application/actions/`
  (use-case orchestration, one file per action —
  `create_document_snapshot_commit.py`, `restore_documents.py`,
  `stage_documents.py`, etc.), `infrastructure/` (concrete adapters:
  `GitPortAdapter` shells out to the `git` CLI via `subprocess`;
  `infrastructure/freecad/` adapts real FreeCAD document/GUI objects to
  the domain's `DocumentLike`/port protocols), `entrypoints/` (Qt UI /
  FreeCAD command registration). This is a noticeably more disciplined
  software-engineering approach than any other project read in this
  census — dependency injection via a `container.py`, `Port` protocol
  classes (`domain/git/ports.py`, `domain/freecad_ports.py`) with
  fake/test-double implementations (`tests/fakes/fake_git_port.py`,
  `fake_freecad_port.py`) enabling **domain and application logic to be
  unit-tested with no real FreeCAD or git process at all** — confirmed
  by `tests/unit/` existing alongside a much smaller `tests/integration/`
  and `tests/freecad/` (which does bundle a real `BasicFile.FCStd` fixture
  and `.snapshots/` golden files for FreeCAD-dependent tests).
- **Maturity/activity**: `package.xml` declares `version 0.1.0`, `date
  2026-06-09`, `freecadmin: 1.1.0` (requires the very recent FreeCAD
  1.1 release — a hard version floor none of the other projects in this
  census impose) — an early-stage but clearly not experimental project;
  `doc/ecosystem/graph.yaml` records `first_seen: 2026-05-05`. The
  README's own Roadmap section (with explicit done/not-done checkboxes)
  and a dedicated Astro-based documentation site
  (`docs-static-site/`, published via `.github/workflows/deploy.yml`)
  both signal an unusually well-organized solo/small-team open-source
  project for something at v0.1.0.
- **License**: LGPL-2.1-or-later (`package.xml`, `LICENSE`) — matches
  `graph.yaml`'s recorded "LGPL-2.1"; no correction needed here.
- **Distribution**: FreeCAD Addon Manager only (`package.xml` is the
  addon-manager manifest FreeCAD itself reads) — install is "search
  'History' in FreeCAD's addon manager," a materially lower-friction
  path than EasyPDM's manual macro-folder install or the Omniverse
  connector's manual `Mod/` copy.

## 2. Identity/versioning model

**Git itself is the version store; the workbench's own contribution is
a derived, comparable "Snapshot" representation layered on top —
deliberately not a replacement for git, a translation layer over it.**
This is the cleanest, most direct instance in this whole census of "use
git as the actual mechanism, just make it CAD-shaped for the user":

- The literal unit of history is a **git commit containing one or more
  whole `.FCStd` files** (`GitService.commit`, `get_committed_files`,
  `get_all_fcstd_paths` — all operate in terms of git commits and FCStd
  paths). There is no separate numeric revision or UUID scheme
  layered on top — a file's identity *is* its git-relative path
  (`domain/git/paths.py`, `is_fcstd_path`), and its version history *is*
  its git log, full stop. This is the most git-native identity model of
  any project in the census — even `ose-vcs-library` (§ own architecture
  note) treats git only as an implicit backing store for plain text; this
  project actively models git concepts (commits, staging, working tree)
  as first-class domain objects (`GitCommit`, `DirtyFile`, `GitRepository`
  in `domain/git/models.py`).
- **Vocabulary is deliberately translated for CAD users**, exactly as
  the README states: "History Workbench uses Git internally... but
  replaces Git terminology with CAD-focused terms." Concretely (from
  README's Quick Start and the `application/actions/` module names):
  commit → **"Save Iteration"**, staging → **"Reviewed Area"** (moving a
  file into the staged/reviewed state before finalizing), working tree →
  **"Current Files Area"**. This is a **UI/vocabulary-layer solution**
  to the "git is unfamiliar to CAD users" adoption barrier, not a
  different underlying mechanism — worth flagging as a directly portable
  idea independent of this project's specific code.
- **The `Snapshot` model** (`domain/snapshots/models.py`) is not a
  replacement identity/version scheme — it's a **derived, structured
  extraction of one FCStd file's document tree at one git ref**, built
  purely to make diffing and comparison possible (a binary FCStd diffs
  as opaque bytes in raw git). `SnapshotObject` (name, numeric id
  retained only "for deterministic YAML ordering," type_id, a properties
  dict) plus `SnapshotOccurrence` (a tree-path occurrence of an object,
  supporting the same object appearing at multiple tree positions — e.g.
  a shared component reused across an assembly, structurally analogous
  to EasyPDM's `item_relations` many-parents model but derived
  per-snapshot rather than stored persistently) together reconstruct
  "what did the document look like," extracted fresh from git blobs on
  demand — not stored as a parallel database, matching the README's
  "local-first storage" claim.
- **Restore is explicitly non-destructive to history**: "Safe restore
  workflow: Restore individual files or batches without rewriting
  project history" (README) — confirmed by
  `GitService.restore_paths_from_ref`/`application/actions/git_workflow/restore_documents.py`,
  which checks out file content from a given ref into the working tree
  (a `git checkout <ref> -- <path>`-style operation) rather than
  performing a `git reset`/history rewrite. This mirrors EasyPDM's own
  explicit design choice (§3 of that architecture note: reverting a
  status bumps the revision forward, never deletes old attachments) —
  an independently-arrived-at shared value across this census: **never
  destroy prior versions to satisfy a "go back" request**.

## 3. Conflict/concurrency strategy

**None of its own — this project inherits git's answer wholesale and
adds nothing on top**, which is a meaningfully different position from
every project read in priority group 1:

- No lock, no owner field, no check-out state anywhere in the domain
  models read (`GitRepository`, `GitCommit`, `DirtyFile`, `GitIdentity`
  — all pure git-mirroring value objects). Concurrent edits between two
  users are handled exactly as two people editing the same git
  repository normally are: whoever pushes/merges second deals with
  whatever git conflict resolution their remote workflow requires (the
  README's roadmap explicitly lists **"Push project to GitHub or other
  git remote services" as not-yet-implemented** — this is presently a
  **local-only, single-user history tool**, so true multi-user
  concurrent-edit conflicts are, today, entirely out of scope rather
  than solved).
- This makes HistoryWorkbench the **third clean instance in this census
  of a project that simply doesn't need (or, here, doesn't yet
  encounter) the "lock, don't merge" pattern** — alongside
  `ose-vcs-library` (mergeable text) and OpenPartsLibrary (no
  concurrent-editing model at all). Here the reason is temporal rather
  than architectural: multi-user git remotes are on the roadmap but not
  built, so the binary-CAD-merge-conflict problem this cluster's other
  members solve via locking hasn't been reached yet, not solved
  differently. Worth flagging as a **fourth data point**, but a
  qualitatively different one from the other two counter-examples — this
  one is "not yet a multi-user tool," not "solved without locking."
- A closely related, already-solved local-conflict problem *is* handled:
  **"noise control"** (README) — excluding object types/properties from
  diffs, tunable float precision (`domain/config.py`:
  `EXCLUDED_PROPERTIES`, `EXCLUDED_PROPERTIES_BY_TYPE`, `EXCLUDED_TYPES`,
  `FLOAT_PRECISION`, all settings-repo-overridable per
  `domain/diff/engine.py`) — this addresses **spurious diff noise from
  FreeCAD's own recompute non-determinism** (e.g. floating-point jitter,
  internal bookkeeping properties changing without a meaningful design
  change), a real, specific friction point distinct from concurrent-user
  conflicts but squarely in the "what does a meaningful CAD version diff
  even look like" territory this whole cluster cares about.

## 4. File format / serialization touchpoints

- **FCStd is the only artifact actually versioned** — stored as
  ordinary git blobs, no format transformation applied to the committed
  file itself (`is_fcstd_path` filtering everywhere confirms `.FCStd` is
  the sole first-class tracked extension today; README's roadmap lists
  "Track and compare non-FCStd files in the project" as not yet done).
- **The diff/comparison layer is where real FreeCAD-specific
  serialization work happens**: `domain/snapshots/gui_extractor.py` (name
  implies pulling structured data from FreeCAD's live GUI/document
  objects) produces `Snapshot`s, and
  `infrastructure/persistence/snapshot_yaml.py` /
  `snapshot_yaml_deserializer.py` round-trip them to/from YAML — i.e. the
  actual git-committed content is the FCStd binary, and the YAML
  snapshot is either a cached derived artifact or regenerated on demand
  for comparison (not fully disambiguated in this pass; worth a
  follow-up read of `snapshot_yaml.py` if the caching-vs-regenerate
  distinction becomes load-bearing for a patch proposal).
- **3D geometry comparison** (`infrastructure/freecad/freecad_visual_diff_creator.py`,
  `domain/diff/visual_diff.py`) renders modified geometry between two
  snapshots in distinct colors directly in FreeCAD's 3D view — a live
  FreeCAD-API integration (shape comparison/coloring), not just
  property-table diffing. This is a genuinely more sophisticated
  CAD-aware diff than any comparison mechanism in priority group 1
  (none of EasyPDM/the Omniverse connector/Ondsel-Server do 3D visual
  diffing at all).
- **Property-level diffing** (`domain/tree/property.py`,
  `domain/diff/comparator.py`) walks FreeCAD object properties
  (dimensions, placements, expressions, constraints, links per the
  README) structurally, not as opaque blobs — this is the part of the
  "why doesn't Git work for CAD" problem
  (`doc/optimistic_locking_research.md`'s literature survey, §1: "CAD
  conflicts are geometric/topological incompatibilities, not line
  diffs") that this project has a real, shipped, partial answer to: it
  doesn't solve semantic merge, but it does solve **semantic diff
  presentation** — showing a human which properties changed, in
  CAD-meaningful terms, even though the underlying git storage is still
  whole-file binary blobs.

## 5. Dependencies & integration points

- **`git` CLI via subprocess** (`GitPortAdapter`) — no GitPython or
  libgit2 binding; a deliberate choice to shell out to the user's own
  git installation (README: "Ensure git is installed on your computer"
  is a listed prerequisite) rather than bundle a git implementation.
  Windows-specific subprocess flags (`CREATE_NO_WINDOW`,
  `STARTUPINFO`/`SW_HIDE`) show real cross-platform testing effort
  beyond the Windows-only Omniverse connector's more minimal approach.
- **PyYAML** — the addon's one declared runtime dependency
  (`package.xml`), used purely for the snapshot serialization format
  described in §4.
- **No server, no database, no network dependency at all** — "local-first
  storage" is an accurate architectural description, not just a README
  claim; every domain/infrastructure file read operates against the
  local filesystem and local git repository only. "Optional remote
  storage and sharing available for advanced users" (README) is not
  elaborated on in the files read in this pass — likely refers to
  ordinary git remotes (GitHub, etc.) rather than a project-specific
  service, consistent with the roadmap item "Push project to GitHub or
  other git remote services" still being unchecked.
- **Real automated testing infrastructure**: unit tests against fakes
  (no FreeCAD/git needed), a separate integration test tier
  (`run_integration_tests.sh`, `run_with_freecad.sh` — implies a real
  FreeCAD process is spun up for at least some tests), and a snapshot
  golden-file mechanism (`tests/freecad/.snapshots/`,
  `scripts/verify_snapshot_extraction.py`). This is the most rigorous
  test setup of any project read in this census.
- **`AGENTS.md` at the repo root** — explicit instructions for AI coding
  agents working in this codebase, a governance/contribution-process
  signal worth noting given this very research project's own use of AI
  agents, though not architecturally relevant to the CAD-collaboration
  question itself.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:historyworkbench` (line 847):
category `[version-control]`, `technical_approach: [git]`, `scope:
freecad-native`, license "LGPL-2.1" (confirmed accurate, no correction
needed), `first_seen: 2026-05-05`, existing note already correctly
identifies it as the highest-star project in the census and reads that
as "the community's actual demand is git-friendly diffing/history, not
server-based PDM." This source-level read supports that conclusion
directly: this is the most substantial, best-tested, most
thoughtfully-architected codebase encountered in the census so far, and
it is a pure git-diffing/history tool with **no server component of any
kind** — a strong structural confirmation of the note's claim, not just
a popularity coincidence.

No corrections needed to this node's fields.

## 7. Friction points observed firsthand

- This project is the strongest evidence yet, across the whole census,
  for a conclusion the plan itself should weigh carefully: **the
  community's revealed preference (144 stars, by far the highest) is
  for tools that make git work better for CAD users, not for tools that
  replace git with a bespoke server-side lock/version system.** Every
  project in priority group 1 (EasyPDM, the Omniverse connector,
  Ondsel-Server) invents its own server-side versioning/locking
  mechanism; this one — clearly the most popular by a wide margin —
  does neither, and instead invests its engineering effort entirely in
  making git's existing history *legible* to non-git-fluent CAD users
  (vocabulary translation, structured diffing, 3D visual comparison).
- The "vocabulary translation over an unmodified mechanism" pattern
  (commit→"Save Iteration", staging→"Reviewed Area") is itself a
  friction point worth naming explicitly: **CAD users' actual objection
  to git is very plausibly UI/vocabulary, not the underlying DAG/commit
  model** — which, if true, reframes Cluster 2's premise. The plan
  frames the friction point as "conflict/concurrency strategy"
  (implying a technical merge problem); this project's success suggests
  a large fraction of the real friction is *presentation*, solvable
  without touching concurrency semantics at all.
- The semantic-diff work (§4) is a real, partial answer to the academic
  literature's core finding (`doc/optimistic_locking_research.md` §1:
  syntactic diffing misses domain-meaningful CAD changes) — not a full
  answer (it doesn't attempt semantic *merge*, only semantic *diff
  presentation*), but a genuinely useful middle ground worth citing as
  prior art if a future proposal in this series engages with that
  literature's gap between "shipped tooling" and "academic state of the
  art."

## 8. Minimal-patch hypothesis

- **Not a fit for a locking/concurrency shim** (§3, §7) — this project's
  entire value proposition argues against needing one, at least for its
  current (single-user, local-first) scope; if/when multi-user git
  remotes are added (roadmap item), the concurrency question becomes
  "how do FCStd binary conflicts get resolved on `git merge`," which is
  a different, harder problem than anything the EasyPDM/Anchorpoint/
  Omniverse-connector locking pattern addresses — worth flagging as a
  genuinely open problem this project will eventually have to face, not
  one it can borrow a ready answer for from this cluster.
- **The single most exportable idea here**: the **snapshot-extraction +
  structured-diff pattern** itself (walk a FreeCAD document's objects/
  properties into a normalized, comparable representation, independent
  of git) is directly reusable by *any* project in this census that
  wants to show "what changed" without needing HistoryWorkbench's git
  integration at all — e.g. EasyPDM's revision-comment history could be
  enriched with an actual structured diff instead of a free-text
  comment, using exactly this project's `Snapshot`/`SnapshotObject`
  shape as a model. Cost: **small to moderate** per adopter (walking
  FreeCAD's object/property model is well-trodden ground, this project's
  `gui_extractor.py`/`tree/property.py` could plausibly serve as
  reference code, license permitting — LGPL-2.1 allows this more freely
  than Ondsel-Server's AGPL would).
- **The vocabulary-translation pattern** (§7) is a trivial-cost,
  purely-presentational idea any git-backed tool in this census
  (`ose-vcs-library` included) could adopt independently: rename git
  concepts in the UI to domain-meaningful terms, without touching
  underlying mechanics. Not really a "patch" in the code-sharing sense
  the plan is looking for, but worth naming explicitly in any synthesis
  document as the single cheapest, highest-leverage UX idea surfaced in
  this entire technical-convergence phase.
