# Architecture notes — FreeCAD_gitproject (`reox/FreeCAD_gitproject`)

Read against `doc/technical_convergence_plan.md`'s 8-field template. The
entire codebase was read in full: `README.md`, `GitWrapper.py` (68
lines), `InitGui.py` (63 lines) — ~115 lines total, the smallest project
in this census by a wide margin. No git commands were run for this
pass; no corrections needed to `doc/ecosystem/graph.yaml`'s existing
node (all fields — LGPL-2.1, status `dead` as of 2018-01-06, `first_seen`
2017-11-08 — confirmed accurate against the source).

Given the size, this note is proportionately brief: several template
fields have essentially nothing to report, which is itself the finding.

## 1. Repo & basic facts

- **Language**: Python, a single FreeCAD workbench module
  (`InitGui.py` + `GitWrapper.py`), no package structure, no tests, no
  CI, no build system of any kind.
- **Author**: Sebastian Bachmann (reox), 2017, per `InitGui.py`'s header
  comment — matches the census's attribution.
- **Status**: explicitly self-labeled proof-of-concept —
  `README.md`'s opening line: "BEWARE: This is just a proof of concept
  at the moment! ... USE WITH CAUTION AT YOUR OWN RISK!" Every listed
  feature beyond autocommit-on-save is marked `// TODO` and none appear
  implemented in the code (tag creation, autocommit toggle, custom
  commit messages, a version-control status indicator — all stubs or
  absent). `doc/ecosystem/graph.yaml` recording `status: dead` since
  2018-01-06 is consistent with a repo that never grew past this state.
- **License**: LGPL-2.1 (`LICENSE`), consistent with `graph.yaml`.
- **Discussion**: linked to a FreeCAD forum thread (2017,
  `forum.freecadweb.org` — the old pre-freecad.org domain, itself a
  small dating signal) rather than GitHub Issues/Discussions — the
  earliest and most informal engagement channel of any project in this
  census.

## 2. Identity/versioning model

**None beyond "each save is a commit."** There is no numbering, no
UUIDs, no revision field, no metadata schema of any kind — `commitchanges()`
(`GitWrapper.py`) does exactly `index.add([f]); index.commit("[FreeCAD
autocommit]")` on the single active file, with a **hardcoded, identical
commit message every time** (`"[FreeCAD autocommit]"` — the `TODO`
"create own commits with messages" was never built). This is the
simplest possible instance of the "git commit = version" pattern also
seen in `HistoryWorkbench` and GitPDM, stripped of every refinement both
of those projects later built (no debouncing, no idle/interval
scheduling, no distinction between a real commit and a recovery
checkpoint, no vocabulary translation for non-git users) — useful
mainly as a **historical baseline** showing how far the git-native
approach has actually developed since 2017, rather than as an
independent design contribution.

## 3. Conflict/concurrency strategy

**None whatsoever, not even considered.** No lock, no presence
indicator, no session guard, no multi-user awareness of any kind
anywhere in the 115 lines. This is unsurprising for a single-evening
proof of concept but is worth stating plainly as a data point: three
projects now sit at increasing levels of concurrency sophistication
along the same git-native lineage — this one (nothing), `HistoryWorkbench`
(nothing yet, but architecturally positioned to add it), and GitPDM
(built real locking, then replaced it with advisory presence +
checkpointing after hands-on experience). Read together, they trace a
plausible maturity curve for what a "just use git" FreeCAD tool
eventually needs to grow, rather than three unrelated data points.

## 4. File format / serialization touchpoints

- **No FCStd-aware logic of any kind** — `commitchanges()` treats the
  active document's file exactly like any other file git can add/commit;
  there is no attempt to understand, diff, or filter FCStd internals
  (contrast `HistoryWorkbench`'s snapshot/diff engine or PR #28312's
  cache-dir mechanism). The entire "integration" with FreeCAD is reading
  `FreeCAD.ActiveDocument.FileName` to know what to commit.
- **The save-hook mechanism is a genuine, if fragile, technical
  finding**: `InitGui.py`'s workbench `__init__` **monkey-patches
  `Gui.SendMsgToActiveView`** at the module/class level (not per-document,
  not via a documented signal), wrapping it to detect when the string
  `"Save"` appears in the call's `args` and firing `commitchanges()`
  afterward. This predates FreeCAD's more structured signal API
  (compare GitPDM's use of `slotChangedObject`/document-level signals,
  or PR #28312's `signalStartSaveDocument`) — a real, if crude, prior
  attempt at "detect a save event" that later, more mature projects in
  this same git-native lineage solved more robustly. Worth noting as a
  concrete illustration of how much FreeCAD's own extension surface for
  this exact use case (observing saves) has matured since 2017.

## 5. Dependencies & integration points

- **GitPython** (`import git`) — the one external dependency, wrapping
  the git CLI at a higher level than GitPDM's/HistoryWorkbench's direct
  `subprocess` calls to the `git` binary. No indication in the README of
  why GitPython was chosen over shelling out directly; not itself a
  finding worth over-interpreting given the project's proof-of-concept
  status.
- **No server, no database, no auth, no multi-host support** — the
  simplest possible integration footprint in this census, smaller even
  than `ose-vcs-library`'s.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:freecad-gitproject-reox` (line
865): category `[version-control]`, `technical_approach: [git]`, status
`dead` as of 2018-01-06, `first_seen: 2017-11-08`, license "LGPL-2.1" —
all confirmed accurate by this full-source read. No corrections needed.

## 7. Friction points observed firsthand

- This project is best read not as an independent data point but as the
  **earliest ancestor in a visible lineage** this census has now traced
  across three points in time: this project (2017, single hardcoded
  autocommit, no concurrency thought at all) → `HistoryWorkbench` (2026,
  full snapshot/diff engine, vocabulary translation, still single-user)
  → GitPDM (2025-2026, multi-host, checkpointing, presence, and a
  documented locking-then-retraction cycle). The "git for FreeCAD"
  approach didn't arrive fully formed — it accreted the specific
  refinements (debounced auto-save, structural diffing, presence over
  locking) one hard-won lesson at a time, each documented in the later
  projects' own commit history/changelogs. Worth citing this lineage
  explicitly in any synthesis document discussing Cluster 2, since it
  demonstrates the maturation path rather than just the current
  end-state.
- The monkey-patched save-hook (§4) is a small, concrete illustration of
  a friction point specific to *building* FreeCAD tooling rather than
  *using* it: in 2017, there was no clean, documented way to observe "a
  save just happened," so this project resorted to intercepting a
  low-level message-dispatch function by string-matching its arguments.
  That gap appears to have been closed by the time of GitPDM/PR #28312
  (both use proper signals) — a small, unglamorous but real piece of
  FreeCAD-core-API history relevant to anyone assessing how mature
  FreeCAD's extension points for this use case have become.

## 8. Minimal-patch hypothesis

- **Not a source of any transferable code or convention** — the project
  is too minimal to contain a reusable idea beyond what `HistoryWorkbench`
  and GitPDM have already independently and more robustly implemented.
  Its value to this technical-convergence phase is entirely historical/
  contextual (§7), not as a patch candidate.
- **No action recommended** beyond noting its place in the lineage when
  the plan's eventual synthesis discusses Cluster 2 — citing it
  alongside `HistoryWorkbench` and GitPDM as evidence of an evolving,
  self-correcting design consensus (autocommit → structural diffing →
  presence-over-locking) is more useful than treating it as a fourth
  independent architecture to weigh on its own terms.
