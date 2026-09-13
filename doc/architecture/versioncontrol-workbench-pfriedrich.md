# Architecture notes — versioncontrol-workbench (`p-friedrich/versioncontrol-workbench`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
The repository was read in full: `README.md`, `structure.py`,
`assembly.py`, `import_json.py`, `scripts/open_from_folder.py`,
`scripts/open_from_loco.py`, `Init.py`/`InitGui.py`. No git commands
were run for this pass. No suspicious binaries or download links were
found — a plain, small, legitimate-looking source repository (unlike
the previous entry in this series).

This project is already a confirmed, load-bearing dependency of
`freecad_git_tryout` (`doc/architecture/freecad-git-tryout-levity0815.md`
added the corresponding `cites_as_prior_art` edge in this same pass) —
this note reads the actual script that dependency runs, closing the
loop on what that earlier note could only describe from the outside.

## 1. Repo & basic facts

- **Language**: Python, a small FreeCAD workbench (~200 lines total
  across `structure.py`/`assembly.py`/`import_json.py`/the two
  `scripts/`) plus unmodified FreeCAD workbench-template boilerplate.
- **A genuinely two-tiered repo, worth distinguishing clearly**: the
  actual **data/import logic** (`structure.py`, `assembly.py`,
  `import_json.py`) is real, working, non-trivial code — but
  `InitGui.py` is **the literal, unedited FreeCAD "create a new
  workbench" tutorial template** ("A description of my workbench,"
  "MyCommand1, MyCommand2," "An existing Menu," a placeholder icon
  comment reading "paste here the contents of a 16x16 xpm icon"). The
  workbench-registration/UI layer was never actually built out — every
  real capability in this repo is invoked via the standalone
  `scripts/` entry points and `import_json.py`, run from the command
  line or via FreeCAD's file-open/importer hook, not through any custom
  toolbar or menu command. This is a meaningfully different flavor of
  "unfinished" than `cadracks-freecad-workbench-git` (spec, zero code)
  or `freecad-gitproject-reox` (small but complete) — here the
  *backend* is complete and was actually used by a third party
  (`freecad_git_tryout`), while the *frontend* is a copy-pasted stub.
- **License**: no `LICENSE` file exists anywhere in the repository —
  `doc/ecosystem/graph.yaml`'s "none declared" is accurate; confirmed,
  no correction needed.
- **Status**: `doc/ecosystem/graph.yaml` records `status: dead` as of
  2018-09-26, `first_seen: 2018-07-20` — plausible given the code's
  clearly-abandoned-mid-tutorial state, but see §6 for a specific date
  inconsistency worth flagging.

## 2. Identity/versioning model

**A filename-embedded ID convention, walked into a hierarchy purely
from folder nesting — no database, no metadata file beyond the derived
JSON, no git integration of its own despite the repo's name.** This is
the exact mechanism `freecad_git_tryout`'s sample data directory names
(`body_in_white___________________________PID0`, etc.) are built to be
consumed by:

- `Structure.from_folders()` (`structure.py`) walks a directory tree
  (`os.walk`) collecting every `.fcstd` file, and for each one plus
  each containing folder, calls `parse_name()` to split a filename like
  `body_in_white___________________________PID_1` into a **PID token**
  (the part's identity — literally "Part ID," extracted as the
  substring from `PID` onward) and a **human-readable label** (every
  underscore-joined segment before it). The long runs of underscores
  visible in `freecad_git_tryout`'s actual filenames exist specifically
  to visually/lexically separate the label from the PID in a plain
  filename — a real, if unusual, encoding convention: **identity and
  display name are both crammed into one filesystem-legal string**,
  because the tool has no other metadata channel to put them in.
- **The folder hierarchy itself is the assembly hierarchy** — nesting a
  part's `.fcstd` file inside subfolders directly encodes its position
  in the assembly tree (each folder becomes an `App::Part` group with a
  parent derived from the folder above it, per `assembly.py`'s
  `create_groups()`). This is the structural mechanism underpinning the
  entire "atomic parts, one file per part" pattern this census has now
  traced through `freecad_git_tryout` and the `cadracks-freecad-workbench-git`
  spec (§7 of both those notes) — this repo is the actual **tooling**
  that made that pattern practical to work with in FreeCAD, rather than
  something either of those two other projects built themselves.
- **A second, parallel identity source exists for a proprietary tool**:
  `Structure.from_fatxmls()` extracts the same group/part hierarchy from
  `.fatxml` files (an XML metadata format belonging to
  [LoCo](https://www.scale.eu/en/products/loco), a commercial
  "Simulation Data Management" product by SCALE GmbH) via a regex
  extracting a `<loco_tree_path>` XML tag's value and splitting it on
  `/`. This is a real, if narrow, integration with **actual commercial
  PDM/SDM software** — the only such integration found anywhere in this
  census's git/version-control cluster — extracting LoCo's own
  already-existing tree-path metadata rather than duplicating a new
  identity scheme for that half of the tool.
- **No revision/version field of any kind** — a part's PID is a stable
  identity, not a version; nothing in this codebase tracks "this is
  revision 2 of this part," consistent with this tool's actual job
  being structural translation (folder-tree ↔ JSON ↔ FreeCAD assembly),
  not PDM record-keeping.

## 3. Conflict/concurrency strategy

**None — and, unlike every other project in this cluster, this one
isn't even adjacent to the question.** This tool has no notion of
"open," "save," "commit," or "sync" at all — it's a **one-shot
converter**: given a folder of `.fcstd` files (or a set of LoCo
`.fatxml` files), produce a `structure.json` describing their hierarchy,
launch FreeCAD pointed at that JSON, and (per both `scripts/`
entry points) **delete the generated `assembly.fcstd`/`structure.json`
files again once FreeCAD exits** (`open_from_folder.py`'s and
`open_from_loco.py`'s trailing cleanup loops). The assembly document
this tool builds is explicitly disposable, regenerated fresh from the
folder structure every time it's invoked — there is nothing here to
version or lock, because the "assembly" itself isn't a persistent
artifact this tool asks anyone to keep. Actual version control (the
stated purpose in the README — "explore the possibilities of using
FreeCAD in conjunction with different version control software," naming
Git and LoCo specifically) is left **entirely external**: the README
never claims this tool itself performs git operations, only that it
produces a *folder layout* (one file per part) that's easy for an
external VCS — git, or LoCo — to manage well. This is a precise,
narrow scope, worth stating plainly: **this project's contribution to
version control is indirect** — it's a structural-translation utility
that makes the "atomic parts" convention practically usable in FreeCAD,
which is what actually helps git (or LoCo) do its job, not a version-
control mechanism in its own right.

## 4. File format / serialization touchpoints

- **A minimal, ad hoc JSON schema** — `export_to_json()`
  (`structure.py`) writes exactly `{"parts": [...], "groups": [...]}`,
  each entry a flat dict of `name`/`label`/`parent`/`filename` — no
  formal schema file, no versioning of the JSON format itself, entirely
  private to this tool's own read/write round-trip (`import_json.py`
  reads back exactly what `structure.py` wrote).
- **`App::Link`-based assembly construction** (`assembly.py`'s
  `create_part()`) — opens each part's own `.fcstd` document and links
  its root object into the generated assembly document via
  `doc.addObject('App::Link', ...).setLink(obj)`, the same underlying
  FreeCAD mechanism EasyPDM's and the Omniverse connector's macros both
  walk for their own assembly-detection logic, and the same one
  `freecad_git_tryout`'s experiments depended on. `assembly.py` imports
  `freecad.asm3.assembly` directly — confirming (independent of
  `freecad_git_tryout`'s own README) that this tool, too, was written
  against the third-party **Assembly3** workbench specifically, not
  FreeCAD's later native Assembly workbench.
- **A real, if minor, correctness note**: `create_part()` prints "More
  than one root object!" if a linked part's document has more than one
  root object, then silently proceeds using only `RootObjects[0]` — a
  known, acknowledged (if unhandled) limitation for parts that aren't
  single-root documents.

## 5. Dependencies & integration points

- **Assembly3** (third-party FreeCAD fork/workbench, `realthunder`'s,
  per `freecad_git_tryout`'s own README) — a hard dependency
  (`import freecad.asm3.assembly`), not optional.
- **A real external subprocess launch, not just a library** — both
  `scripts/open_from_folder.py` and `scripts/open_from_loco.py` build
  the JSON, then literally shell out (`subprocess.Popen(["freecad",
  jsonpath], ...)`) to open a *separate* FreeCAD process pointed at the
  generated file — this tool is designed to be invoked from **outside**
  FreeCAD (a file manager, or LoCo's own "external application"
  configuration hook, per the README's `open_from_loco` description) as
  much as from within it.
- **LoCo (SCALE GmbH)** — a real, named commercial dependency for half
  of this tool's stated purpose; not independently investigated further
  in this pass (proprietary, no public source to read).
- **No dependency on git itself anywhere in the code** — despite the
  repo's name and stated purpose, nothing here invokes `git` directly;
  the git integration is entirely structural/indirect (§3).

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:versioncontrol-workbench-pfriedrich`
(line 879): category `[version-control]`, `technical_approach: unknown`,
status `dead` as of 2018-09-26, `scope: freecad-native`, license "none
declared" (confirmed accurate — no `LICENSE` file exists), `first_seen:
2018-07-20`.

**Correction applied to `graph.yaml` in this pass** (see below):
`technical_approach: unknown` → `[other]`, since this read confirms the
tool doesn't implement git (or any single VCS) directly at all — it's a
folder-structure/JSON-translation utility that indirectly supports
git *and* a proprietary tool (LoCo) equally, which the existing
`[git]` tag used by sibling nodes in this cluster would misrepresent.

**A date inconsistency worth flagging, not resolved in this pass**:
this node's `first_seen` is recorded as **2018-07-20**, but
`project:freecad-git-tryout-levity0815`'s `first_seen` is **2018-07-08**
— twelve days *earlier* — despite `freecad_git_tryout`'s own README
documenting active use of this workbench's `open_from_folder.py` script
from the start. Either this node's `first_seen` date is wrong, or
`freecad_git_tryout`'s is, or (most likely) both dates reflect each
repo's own GitHub-recorded creation timestamp rather than a single
shared, verified timeline, and the discrepancy is simply an artifact of
using repo-creation dates as a proxy for "when the code was usable" (a
private/local script can be usable before its own repo is created, or a
public repo's creation date can lag well behind when its author started
using the code privately). Flagged for a future graph-consistency pass
rather than corrected here — resolving it would require checking actual
commit history, which this session's git-command restriction rules out
per the plan's own current-session methodology.

## 7. Friction points observed firsthand

- This project is best read, per §2 and per its own confirmed citation
  in `freecad_git_tryout`, as the **actual tooling substrate** behind
  the "atomic parts, one file per part" pattern this census has now
  independently confirmed three times over (empirically in
  `freecad_git_tryout`, as a design thesis in the
  `cadracks-freecad-workbench-git` spec, and now as working
  infrastructure here). None of those other two projects built their
  own folder-to-assembly translation logic — this is the one piece of
  code in that whole cluster that actually does it, quietly, without
  much fanfare, only a stub of a workbench UI wrapped around it.
- The dual git/LoCo scope (README's explicit "two systems will be
  used") is a small, genuine data point that the *problem* this whole
  cluster keeps re-encountering (binary CAD files don't play well with
  version control) is recognized identically whether the version-control
  system in question is free/open (git) or commercial (LoCo) — the
  fix (decompose into linked per-part files) is VCS-agnostic by
  construction, which is a small but real point in favor of proposing
  it as a convention independent of which VCS a given team happens to
  use.
- The unfinished-tutorial-boilerplate `InitGui.py` (§1) is a small,
  concrete illustration of a recurring shape in this census's smaller/
  abandoned repos: **real, working backend logic wrapped in an
  never-finished UI layer** — worth distinguishing explicitly from
  "the whole thing is a stub" (as with `cadracks-freecad-workbench-git`)
  when assessing how much of a given abandoned repo is actually usable
  today. Here, the answer is: all of it, just not through a FreeCAD
  menu — via the command-line scripts directly.

## 8. Minimal-patch hypothesis

- **Not itself a new patch candidate** — its main relevance to this
  plan is as confirmed, working infrastructure for a pattern
  (§7, and the `freecad_git_tryout`/`cadracks-freecad-workbench-git`
  notes) already identified as the strongest minimal-patch candidate
  in this whole census: decompose assemblies into one linked file per
  part. This repo demonstrates that pattern doesn't even require new
  tooling to become practical — a ~200-line folder-walker/JSON-
  translator, of exactly the kind read here, is sufficient scaffolding.
- **A concrete, small addition worth naming**: since this tool already
  proves the "atomic parts" convention is VCS-agnostic (§7), a natural,
  low-cost follow-on would be a thin, direct `git` integration in the
  same spirit as `open_from_loco.py`'s LoCo hook — e.g. an
  `open_from_git_worktree.py` that resolves a structure directly from a
  git-tracked folder rather than a plain filesystem walk (so per-part
  git history/blame becomes directly queryable from within the
  generated assembly). Cost: **small**, given `structure.py`'s
  `from_folders()` already does 90% of the needed work; the addition
  would be substituting `os.walk()` over a plain directory with a walk
  over `git ls-tree` output, an incremental change to existing,
  working code rather than a new architecture.
