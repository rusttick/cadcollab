# Architecture notes — freecad-workbench-git (`cadracks-project/freecad-workbench-git`)

Read against `doc/technical_convergence_plan.md`'s 8-field template. The
repository contains no code — its entire content is a one-line
`README.md`, a `LICENSE` file, and one design document,
`specs/freecad_workbench_git_specs.odt` (a LibreOffice/OpenDocument
file), extracted and read in full for this pass (unzipped as an ODT is
a zip container; its `content.xml` was decoded to plain text — see
method note below). No git commands were run for this pass.

`doc/ecosystem/graph.yaml`'s existing note calls this "Empty scaffold" —
accurate in that there is zero implementation, but the repo is not
actually empty of content: it holds a real, dated, single-author design
specification that never got built. This note is proportionately brief,
matching the amount of actual substance, but the spec itself contains
one point directly relevant to Cluster 2 worth surfacing precisely (§3).

## 1. Repo & basic facts

- **Content**: a design spec only, no implementation — `README.md` is
  literally one line ("Git version management of CAD projects"); the
  substance is entirely in the ODT spec.
- **Author/date**: "Guillaume Florent," dated August 23rd 2018, per the
  spec document's own revision-history table (title page: "PyOSVFreeCAD
  Workbench - Git," a single-row changelog reading "August 23rd 2018 /
  Guillaume Florent / Creation"). Matches `doc/ecosystem/graph.yaml`'s
  `status_as_of: 2018-08-23` exactly.
- **License**: LGPL-3.0 (`LICENSE`, full GNU LGPLv3 text) —
  `doc/ecosystem/graph.yaml` currently records `license: unknown`; a
  straightforward correction (see §6).
- **Part of a named cluster**: the existing graph note already correctly
  places this within "the cadracks-project cluster" (§10.7 of the
  census) — the same GitHub org (`cadracks-project`) responsible for
  `cadracks-freecad-workbench-plm`, `cadracks-openplm`, and `cadracks-opm`
  elsewhere in this census's repository list, though those weren't
  independently cross-checked against this repo's own content in this
  pass.

## 2. Identity/versioning model

**Not designed in any detail — the spec is entirely commands/UI-focused,
not data-model-focused.** The one identity-relevant design decision
stated is structural, matching a theme already surfacing repeatedly in
this census: "Git Init by considering the containing folder of a FCStd
file as the git root" — i.e. one git repo per project folder, identity
derived from filesystem layout, no separate ID scheme of any kind. No
revision numbering, no metadata schema, nothing beyond what plain git
commits already provide (matching `HistoryWorkbench`'s and GitPDM's own
"git commit is the version" baseline, though this spec predates both by
roughly seven years and was never implemented to compare against).

## 3. Conflict/concurrency strategy

**This is the one place the spec goes beyond "wrap git in a UI" and
states a real architectural thesis, worth quoting precisely** (§1.2,
"History," of the spec, lightly reformatted from the extracted text):

> "The possibility to create an assembly from elements defined in
> individual files opens the possibility to manage project versions
> with Git. It was possible with monolithic files but lacked interest
> because of cumbersome/impossible [merge] conflict resolution. Atomic
> parts are easier to handle regarding to merging conflicts (i.e. use of
> `--theirs` or `--ours` in Git)."

This is **the same core insight `freecad_git_tryout` (levity0815)
independently demonstrated empirically roughly six weeks earlier** (that
project's `first_seen` is 2018-07-08; this spec is dated 2018-08-23) —
decompose an assembly into one file per atomic part, and git's ordinary
file-level conflict resolution becomes sufficient, because conflicts
only arise when two people touch the *same* part rather than merely the
same containing assembly. No evidence either project was aware of the
other (consistent with `doc/ecosystem/graph.yaml`'s general finding
across this cluster of near-simultaneous, cross-unaware reinvention —
see §7). Beyond restating that structural principle, the spec's own
conflict-handling design is thin and explicitly provisional: a
**"Merge" command** proposes "Dialog to select branch to merge into and
branch to merge from / Display of geometric diffs (reuse difference
macro) / Merge conflicts → dialog to choose either version with diff
display" — i.e. **manual, either/or conflict resolution with a visual
diff aid**, not automatic merging, not locking, and not the
"construct a side-by-side assembly of both versions" idea
`freecad_git_tryout`'s README independently floated (§8, `freecad_git_tryout`'s
own architecture note) — a narrower, more conventional "pick one side"
UI than that unbuilt idea, for whatever that comparison is worth given
neither was ever implemented.

## 4. File format / serialization touchpoints

- **No format-level work specified at all** — no mention of FCStd
  internals, diffing strategy for the zip container, or any equivalent
  of Zippey/PR-28312's cache-dir/HistoryWorkbench's snapshot extraction.
  The one diff-related line ("Display of geometric diffs (reuse
  difference macro)") explicitly defers to an existing, unnamed
  "difference macro" rather than proposing new diffing machinery of its
  own — the spec's scope is a **command/UI wrapper around git**, not a
  file-format innovation.
- **History visualization** is specified in more UI-design detail than
  anything else in the document: "Subway map -like display of the
  commits history" plus a plain-text fallback ("git log output with
  default options") — an explicit design goal for visual commit-graph
  browsing that neither `HistoryWorkbench` nor GitPDM's read source (in
  this census's other notes) documents having built in quite this form,
  though both have some form of history browsing.

## 5. Dependencies & integration points

- **Explicitly builds on, rather than replaces, an existing FreeCAD
  addon**: "The Webtools workbench of FreeCAD is a great starting point
  that covers some Git possibilities" — and for Push/Pull specifically,
  the spec literally just says "See Webtools" twice (§2.6, §2.7),
  meaning **push/pull were never even specified here**, only deferred to
  another tool entirely. This is a real, if minor, data point: this
  spec did not intend to be a complete, standalone git-integration
  solution — it framed itself as covering the *gap* Webtools left
  (init/add/commit/branch/merge/diff/history), not as a competing
  full solution. Webtools itself is referenced by name in
  `HistoryWorkbench`'s own roadmap-adjacent README text too
  (`doc/technical_convergence_plan.md`'s repository list does not
  separately list Webtools as its own entry) — worth noting as a
  recurring, uninvestigated reference point across multiple projects in
  this cluster, though Webtools itself falls outside this pass's scope.
- **Explicit non-goal**: "Scriptability — Not required. It is more
  natural to script through the command line" — i.e. the spec
  deliberately did not intend a Python-scriptable API surface, on the
  reasoning that anyone wanting programmatic git access already has the
  command line. A conscious, stated scope-narrowing decision, not an
  oversight.
- **No dependency on anything beyond FreeCAD + git + Webtools** is
  specified — no server, no host integration, no credential management
  of any kind (contrast GitPDM's substantial later buildout of exactly
  those things).

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:cadracks-freecad-workbench-git`
(line 908): category `[version-control]`, `technical_approach: [git]`,
status `dead` as of 2018-08-23 (confirmed exact match to the spec's own
date), `scope: freecad-native`, license `unknown`, `first_seen: "2018"`,
existing note "Empty scaffold, part of the cadracks-project cluster,
§10.7" — accurate as far as it goes; this read adds that the repo is
"empty" of code but not of content (a real, single-author design spec
exists and was read in full for this pass).

**Correction applied to `graph.yaml` in this pass** (see below):
`license: unknown` → `"LGPL-3.0"`, confirmed directly from the `LICENSE`
file (full GNU LGPLv3 text, unambiguous — no internal inconsistency of
the kind found in `freecad_git_tryout`'s license file).

**A likely-missing edge, not yet applied**: the shared "atomic parts
decompose merge conflicts" insight (§3) between this spec and
`freecad_git_tryout` (roughly six weeks apart, no evidence of
cross-awareness) is structurally identical to the existing
`independently_reinvents` edge already recorded between
`freecad-gitproject-reox` and `versioncontrol-workbench-pfriedrich`
(graph.yaml, ~line 2035) — the same edge type, for the same kind of
finding (near-simultaneous 2018 reinvention within this cluster), just
between two different node pairs. Worth adding in a future
graph-consistency pass; not applied here since it would be the second
edge-addition judgment call made in this architecture-note series and
this document's job is to surface it, not silently keep expanding the
graph's edge set without a chance for a broader consistency check.

## 7. Friction points observed firsthand

- This spec is now the **third independent 2018-era attempt** this
  census has traced (alongside `freecad-gitproject-reox`,
  `freecad_git_tryout`) at solving git-plus-FreeCAD collaboration, all
  within roughly a ten-month window (Nov 2017 → Aug 2018), all with no
  documented cross-awareness of each other. Read together with
  `HistoryWorkbench` and GitPDM's own 2025-2026 efforts, this is now a
  **five-project-deep, decade-spanning pattern** of the same problem
  being independently rediscovered and re-attempted — the single
  clearest, most repeatedly-confirmed instance of duplicated effort
  anywhere in this whole census, and squarely the kind of evidence
  `doc/technical_convergence_plan.md`'s opening paragraph is built to
  act on.
- The atomic-parts insight (§3) being independently reached by two
  different people within six weeks in 2018, and then re-appearing as a
  load-bearing design principle in `ose-vcs-library` (2026, directory-
  per-entry from the start) and implicitly validated by PR #28312's
  don't-version-regenerable-content approach, suggests this specific
  idea has now reached a level of repeated, independent confirmation
  that a synthesis document could reasonably treat as **settled
  best practice**, not merely a hypothesis — worth stating plainly
  rather than hedging further, given how consistently it recurs across
  otherwise-unrelated projects and years.
- The explicit deferral to Webtools for push/pull (§5) is a small but
  real illustration of a **healthy** pattern this census should also
  credit, not just flag: a project scoping itself around an existing
  tool's gaps rather than reimplementing everything, is exactly the kind
  of low-duplication behavior the plan is hoping more projects would
  exhibit. That it still didn't get built has more to do with
  single-author bandwidth (the spec is a solo 2018 document with no
  follow-up commits) than a design flaw.

## 8. Minimal-patch hypothesis

- **No code exists here to adopt** — this document's only transferable
  content is the design thesis in §3, which is better treated as
  corroborating evidence for a synthesis document's argument than as a
  patch source in its own right.
- **The "atomic parts, one file per part" convention** (§3, §7) —
  already the standout minimal-patch candidate from
  `freecad_git_tryout`'s own architecture note — gains additional
  weight from this spec's independent arrival at the identical
  conclusion a few weeks later. Nothing new to add to that
  recommendation beyond noting the corroboration; see
  `doc/architecture/freecad-git-tryout-levity0815.md` §8 for the fuller
  treatment.
- **Method note for future repos of this shape**: this repo demonstrates
  that a "no code" repository in this census's remaining list can still
  be worth reading in full when it contains a genuine, extractable
  design document (here, an ODT unzipped and its `content.xml` stripped
  of markup) rather than being skipped as contentless — worth keeping in
  mind for any other remaining entries in the plan's repository list
  that might turn out to be documents-only rather than code-only.
