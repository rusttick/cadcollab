# Core vs. Addon: Which Collaboration Features Actually Need a FreeCAD PR?

Purpose: a cross-cutting synthesis question over the technical-
convergence architecture notes (`doc/architecture/*.md`, ~23 projects
read as of this writing, per `doc/technical_convergence_plan.md`) —
of everything this census found any project doing under the banner of
"FreeCAD collaboration," which pieces genuinely require a change merged
into `FreeCAD/FreeCAD` core, and which are already fully achievable as
a third-party addon with zero core changes? This matters directly for
scoping any future minimal-patch proposal: a fix that requires getting
code through FreeCAD's own core review process is a fundamentally
different, slower kind of "small patch" than one any single project
can ship independently.

Methodology: built from source-level findings already recorded in the
individual architecture notes, not from new research. Where a claim
here extends beyond what a specific note stated (e.g. an inference
about what FreeCAD's Python API can or can't do), that's flagged
explicitly rather than presented as separately verified.

## Needs FreeCAD core changes (confirmed, no addon workaround exists)

### 1. Reducing what goes inside the `.FCStd` zip on save

The clearest, best-evidenced "must be core" item found in this whole
census. Per `doc/architecture/pr-28312.md`: `Base::Writer`/`Base::Reader`'s
zip-writing loop and the new `Base::Persistence::canBeCachedForDocument()`
flag are pure C++ engine internals inside `Document::saveToFile()`.
There is no Python/addon hook that lets code intercept "should this
attachment go in the zip or externally" during FreeCAD's own save —
every addon read in this census that wants smaller diffs or externalized
derived-content either reads the FCStd zip *after the fact* as an
external file (`HistoryWorkbench`'s snapshot extraction), or avoids the
problem by never letting FreeCAD create the noisy content inside FCStd
in the first place (`ose-vcs-library`'s never-commit-the-binary
approach). Nobody found a way to do incremental/partial FCStd writes
from Python. **The actionable move for any project caring about this
friction point is tracking/advocating for PR #28312 (or an equivalent),
not reimplementing it at the addon layer** — no addon can substitute
for this.

### 2. True in-memory concurrent editing of one open document (CRDT-style merge)

Nothing in this census attempts this — not even the Omniverse
connector's "Live" mode (`doc/architecture/freecad-omniverse-connector.md`),
which operates on a separate USD layer, never on the FreeCAD document
itself. FreeCAD's own transaction/undo/recompute machinery
(`Document::addRecomputeObject`, property-change notification) isn't
exposed with the fine-grained, network-transparent hooks real-time
merge would need. This is an inference, not something any project
attempted and hit a wall on — but it's the strongest candidate for
"would need deep core work" if anyone ever wanted it, and quite
plausibly *why* every project in this census instead converged on
exclusive locking or async versioning rather than real co-editing (see
`doc/architecture/gitpdm.md` §3 for the most explicit discussion of
that tradeoff any project in this census articulated on its own).

### 3. Shipping something in every install by default, in native menus

Trivial but real: anything that must be available without installing
an addon — e.g. PR #26306's `Collaboration_Topic` command appearing
directly in the standard "Tools" menu next to `Std_AnnotationLabel`
(`doc/architecture/pr-26306.md` §5) — has to go through core by
definition, independent of whether the underlying *capability* could
technically be built as an addon (see the grey zone below).

## Grey zone — cheaper in core, but not strictly required there

**PR #26306's `Topic` annotation object** is the interesting case. It
subclasses the existing C++ `App::AnnotationLabel`/`App::GroupExtension`
and reuses the existing `Attacher` geometry-attachment engine (the same
mechanism Sketcher/PartDesign use for datum attachment) — all core C++
classes, per `doc/architecture/pr-26306.md` §2. But nothing about the
*concept* (a labeled marker, anchored to geometry or floating in
space, grouped under a document group) is fundamentally locked to
core: FreeCAD's existing `Part::FeaturePython` mechanism already lets
an addon define a brand-new document-object type in pure Python with
arbitrary properties, and a Python `ViewProvider` (using `pivy`, the
Python bindings for the same Coin3D scene graph FreeCAD's 3D view
already uses) can render custom 3D-view geometry including floating
text. `BCF-Plugin-FreeCAD` (`doc/architecture/bcf-plugin-freecad.md`)
is indirect proof this general territory is addon-reachable — it
manages topic/comment/viewpoint data entirely from Python, just
without a live, geometry-anchored 3D presence the way `Topic` has.

**So: building `Topic` in core is cheaper (reuse existing C++
machinery) but not the only way this capability could exist.** If
BCF-style external exchange and native in-document `Topic` markers
ever need to meet in the middle, the interoperability point doesn't
have to be a core PR — it could be a plugin translating between the
two, built entirely at the addon layer. (`doc/architecture/pr-26306.md`
§6/§8 flags this exact open question — whether the PR's author
consulted BCF-Plugin-FreeCAD's maintainer — as worth checking before
any further core investment risks duplicating that project's own
`rdwr/` package.)

The one piece of PR #26306 that *is* narrowly core-only **as
implemented** (not as a matter of necessity): the new `Gui::DoubleClick`
helper and the change to `ViewProviderAnnotationLabel::dragStartCallback`
— because it intercepts raw Coin3D mouse events *during an existing
C++ drag manipulator's* event handling, below the level where a Python
`ViewProvider.doubleClicked()` callback normally fires. An addon with
its own custom manipulator could implement equivalent double-click
detection itself in Python/PySide without touching core — it's only
core-only here because this PR chose to extend the existing manipulator
rather than write a new one from scratch.

## Fully addon-implementable today (the large majority of everything found)

Everything else catalogued across this census — which is most of it —
needs zero FreeCAD core changes. Grouped by mechanism, with the
architecture note each is drawn from:

- **All check-in/check-out/locking mechanisms**: EasyPDM's `owner_id`/
  `owner_locked` (`easypdm.md`), GitPDM's presence branch + continuous
  checkpointing (`gitpdm.md`), OdooPLM's database-constraint-enforced
  checkout table (`odooplm.md`), CADBase's `is_hidden`-flag revision
  toggle (`cadbaselibrary.md`), Anchorpoint's proprietary metadata
  server (`anchorpoint.md`). FreeCAD is just told to open/save a file
  in every one of these — the coordination logic lives entirely
  server- or client-side, fully outside FreeCAD's own process model.
- **Assembly/BOM tree walking**: EasyPDM, the Omniverse connector,
  `versioncontrol-workbench-pfriedrich`, `taack-plm` — all built on the
  already-scriptable `doc.Objects` / `App::Link` type-checking API,
  no core hooks needed.
- **Git integration**: `HistoryWorkbench`, GitPDM, the older
  `freecad-gitproject-reox`/`freecad_git_tryout` pair — all shell out to
  `git` or use GitPython from Python. `HistoryWorkbench`'s structural
  diff/snapshot extraction (`historyworkbench.md` §4) reads the FCStd
  zip itself from Python as an external file, entirely outside
  FreeCAD's own save/load pipeline — proof that meaningful structural
  diffing doesn't require core cooperation either, only careful
  external parsing.
- **Export-driven automation**: EasyPDM's STEP-export bypass of the
  generic `Part.export()` (using FreeCAD's own shape/export API more
  directly instead), nanoPLM's Spreadsheet-driven parametric
  configurator (`nanoplm.md` §4), CADBase's thumbnail/STEP client
  uploads — all built on existing, already-exposed export APIs
  (`Import`, `Mesh`, `FreeCADGui.export`, the `Spreadsheet` object's
  `.set()` method).
- **Reusing FreeCAD's own native identity**: `taack-plm-freecad`'s use
  of `Document.Uid` (`taack-plm.md` §2) needs *nothing new* — the
  property already exists; the finding here is "use what's already
  there" rather than "build something new," at either layer.
- **The BCF interchange format itself**: `BCF-Plugin-FreeCAD` is
  entirely self-contained — reads/writes its own `.bcf` zip files with
  no dependency on FreeCAD's own document format at all.
- **The "atomic parts, one file per part" convention**: not even an
  addon — `freecad_git_tryout`'s experiment, the
  `cadracks-freecad-workbench-git` spec, and `versioncontrol-workbench-pfriedrich`'s
  actual tooling all demonstrate this is a pure authoring/workflow
  convention. Zero code required at any layer — the single cheapest
  "patch" surfaced anywhere in this entire census.

## The practical implication

The one item on the "needs core" list that's actually in-flight and
mergeable today is **PR #28312**. It's also the single most
load-bearing item for this whole plan's Cluster 2 question, since it's
the one thing no downstream project — however clever its own addon —
can substitute for on its own. Everything else labeled "collaboration
feature" across this census, including the more elaborate concurrency
and versioning schemes, was built entirely at the addon/server layer.
That's a genuinely useful constraint on the plan's own scope: most of
the friction points this census has catalogued don't need anyone to
get anything past FreeCAD's own core review process at all — the
"small, low-cost, independently-adoptable" patches the plan's opening
paragraph is looking for are, for the most part, already provably
reachable without core cooperation.
