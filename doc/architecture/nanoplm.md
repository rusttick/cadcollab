# Architecture notes — nanoPLM (`alekssadowski95/nanoPLM`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read: `README.md`, `nanoplm/__init__.py` (the actual data model —
`models.py` itself is an empty 3-line stub, a leftover from an earlier
refactor), `nanoplm/freecad.py`, `nanoplm/open_doc.py`, and a routes.py
grep pass. No git commands were run for this pass.

Per `doc/ecosystem/graph.yaml`'s own top-of-file note, this project
previously received only "a lighter pass" (stub-level, mostly `unknown`
fields) as part of the `alekssadowski95` cluster — the same author
behind `OpenPartsLibrary` (already given a full architecture note in
this series), `FreeBOM`, `we-have-pdm-at-home`, and `PyPDM`. This pass
fills that in with an actual source read, per this technical-
convergence phase's own methodology of reading code rather than relying
on prose census summaries.

## 1. Repo & basic facts

- **Language**: Python, Flask + Flask-SQLAlchemy, SQLite, Bootstrap-
  templated server-rendered pages — the same general stack shape as
  `OpenPartsLibrary` by the same author, but noticeably smaller
  (~1,000 lines across `nanoplm/*.py` vs. OpenPartsLibrary's ~4,600).
- **README is heavily marketing/SEO-oriented**, unusually so for this
  census: long generic sections on "Understanding PLM Software,"
  aerospace/automotive case studies (Boeing, Ford, GE, Caterpillar,
  Siemens) using commercial PLM tools nanoPLM has no relationship to,
  and a "Challenges in Traditional PLM Systems" section reading like
  landing-page copy. This is a real, legitimate open-source project
  (confirmed by reading the actual working code below) with unusually
  promotional public-facing material layered over it — worth noting as
  a style choice distinct from every other project in this census's
  READMEs, not a red flag in itself (contrast the `sabi137032` cloud-
  browser repo, where marketing tone accompanied a genuinely absent
  feature set — here the described features are real, just verbosely
  sold).
- **A real, named author and business**: Aleksander Sadowski, "ALSADO,"
  Sankt Augustin, Germany — a legitimate contact address given in the
  README, and the project was presented at FOSDEM 2025. `doc/ecosystem/graph.yaml`
  currently records `origin_country: unknown`; corrected in this pass
  (see §6).
- **License**: MIT, confirmed directly from `LICENSE`'s header
  ("Copyright (c) 2024 Aleksander Sadowski") — `graph.yaml` currently
  records `license: unknown`; corrected (see §6).
- **A stray personal debug script left in the repo**
  (`nanoplm/open_doc.py`) hardcodes an absolute Windows path
  (`C:/Users/Aleksander/Documents/GitHub/nanoPLM/...`) and calls
  undefined names (`FreeCAD` used without import) — clearly a
  throwaway local scratch file accidentally committed, not part of the
  application's real code path (`routes.py` never imports it). A small
  but concrete "single-developer project, not much repo hygiene around
  what gets committed" signal, consistent with the project's overall
  small scale.

## 2. Identity/versioning model

**Minimal to the point of near-absence — the real substance of this
project lies elsewhere (§4), not in PDM-style identity/versioning.**

- The actual data model lives in `nanoplm/__init__.py` itself (not
  `models.py`, which is a dead 3-line stub — a leftover from what was
  probably an in-progress refactor to split models out of the app
  factory file). Four SQLAlchemy models: `Component` (uuid, name,
  `component_number`, a `status` string defaulting to `"draft"` with no
  enforced state machine anywhere in the code read, `files` as a bare
  `VARCHAR(200)` — not a relation, a raw string), `Instance` (a
  serialized/manufactured unit of a component — `serial_number`,
  `component`, `client`, again `files` as a plain string), `Client`,
  and `File` (uuid, name only — no path, hash, or size column at all).
- **No revision, version, or history concept anywhere in this schema or
  in `routes.py`** (grepped directly for `lock`/`version`/`revision`
  across the whole routes file — zero matches). `status` is the only
  lifecycle field, and it's a free string with only one observed
  default value (`"draft"`) — no transitions, no enforcement, no audit
  trail. This is the thinnest identity/versioning model of any
  database-backed project in this census, thinner even than
  OpenPartsLibrary's (which at least has a `revision`/`lifecycle_state`
  pair, even if unenforced).
- **This is not a gap so much as a sign the project's actual center of
  gravity is somewhere else** — see §4.

## 3. Conflict/concurrency strategy

**None — and, like OpenPartsLibrary, this isn't really a gap because
the tool isn't built for concurrent multi-user editing at all.** It's a
locally-run Flask app over a single SQLite file (the README's own
"Run locally without an internet connection" and "Data Privacy through
Local Deployment" sections make this an explicit design goal, not an
oversight) — a single-workstation tool for one small manufacturer's
internal use, not a shared multi-user server. No auth/login system was
found in the code read (contrast OpenPartsLibrary, which likewise has
none, or EasyPDM/GitPDM/CADBase, which all do) — consistent with a
tool meant to run on one machine for one user or a small trusted team
sharing a desk, not a networked collaboration platform. This is now
the **fourth or fifth project in this census** for which "no
concurrency model" is a legitimate design consequence of scope rather
than an oversight (alongside `ose-vcs-library`, OpenPartsLibrary, and
BCF) — worth noting as further confirmation that a meaningful fraction
of the FreeCAD-adjacent ecosystem simply isn't building for the
concurrent-multi-editor scenario this plan's Cluster 2 centers on.

## 4. File format / serialization touchpoints — the actual interesting part

**This is where nanoPLM earns its place in this census: it is a
database-driven parametric-CAD configurator, not primarily a PDM tool,
and no other project read in this series attempts this.**

- `nanoplm/freecad.py`'s `set_product_data_in_spreadsheet(dir_path,
  product)` opens a **shared FreeCAD template document** headlessly
  (`FreeCAD.open`) and writes named values directly into a FreeCAD
  `Spreadsheet` object's cells (`spreadsheet.set("name", ...)`,
  `set("stammblattbreite", ...)`, `set("plattensitzhoehe", ...)`, etc.
  — German engineering terms for blade width/seat height/cutting width
  on what the bundled sample `SBHH.FCStd` suggests is a saw-blade-
  holder product line), then calls `recompute()` on both the
  spreadsheet and the document. This is a direct, deliberate use of
  FreeCAD's well-known spreadsheet-driven-parametric-design technique
  (linking model dimensions to named spreadsheet cells), automated
  entirely from a database row's field values — **the PLM database
  record doesn't just describe a design, it actively parameterizes and
  regenerates one.**
- **A cascade of derived-file generation functions follow the same
  pattern**: `create_preview_3d` (mesh export to `.stl` via FreeCAD's
  `Mesh` module, for a web-viewable preview), `create_generic_3d`
  (STEP export via `Import`), `create_technical_drawing` (PDF export of
  a TechDraw `Page001` object via `FreeCADGui.export`), and
  `create_manufacturing_file` (a currently-trivial open/save/close, a
  clear placeholder for future manufacturing-format export). Each
  function opens the same parameterized document, exports one derived
  artifact, and closes it — a genuine, if unglamorous, working headless-
  FreeCAD automation pipeline.
- **This is architecturally distinct from every other CAD-automation
  approach in this census**: EasyPDM's and the Omniverse connector's
  macros export *from* an already-authored, human-modeled document;
  Ondsel-Server's FC-Worker converts *whatever file was uploaded*;
  CADBase's `three-libs` parses STEP *for browser viewing only*.
  nanoPLM instead treats one parametric template document as a
  **product family generator** — a single `.FCStd` file plus a
  database of parameter sets can produce an unbounded number of
  distinct product variants' previews/STEP/drawings on demand. This is
  a genuinely different point in the design space: closer to a
  configurator (as sold by commercial CPQ/PLM vendors) than to a
  version-control or file-sharing tool, and it's the only project in
  this whole census attempting it.
- **The mechanism is fragile in ways typical of a small, single-author
  tool**: hardcoded object names (`doc.getObject('Spreadsheet')`,
  `doc.getObject("Body")`, `doc.getObject("Page001")`) mean every
  product template must use these exact internal names — there's no
  discovery or configuration of what a given template's spreadsheet/
  body/page is actually called. This works for one team's own
  standardized templates but wouldn't generalize to arbitrary FreeCAD
  documents without convention discipline baked into every template
  file.

## 5. Dependencies & integration points

- **FreeCAD itself, imported directly as a Python module inside the
  Flask process** (`from nanoplm import FreeCAD`, `__init__.py`'s
  commented-out `sys.path.append(FREECAD_ABS_PATH)` block shows the
  intended mechanism: point Python at FreeCAD's own `bin/` directory to
  import `FreeCAD`/`FreeCADGui`/`Part`/`Mesh`/`PartDesign` as regular
  modules) — i.e. nanoPLM **embeds FreeCAD inside its own server
  process** rather than shelling out to a separate FreeCAD invocation
  or macro. This is a real architectural commitment (and constraint):
  the Flask app and FreeCAD share one Python interpreter and one
  process lifetime, which is why `create_technical_drawing` ends by
  calling `FreeCADGui.getMainWindow().close()` — the GUI subsystem gets
  spun up and torn down within a single web request's handling.
  `__init__.py`'s `app.config['SUPPORTED_FREECAD_VERSIONS']` /
  version-autodetection logic (checking `C:/PROGRA~1/FreeCAD 0.21` and
  `0.20` paths) confirms this is Windows-first, hardcoded-path-based
  integration, consistent with the README's "Compatible with Windows
  10" claim and the `.bat`/`.spec` PyInstaller packaging files at the
  repo root (`create_dist.bat`, `run.spec`, `run_local.spec`).
- **No external services, no auth provider, no cloud dependency** —
  entirely local, matching the stated privacy/offline design goal.
- **Same author, same general Flask/SQLite/Bootstrap architecture as
  `OpenPartsLibrary`** — `doc/ecosystem/graph.yaml` already records
  this correctly via the existing `same_author` edge
  (`person:alekssadowski95` → nanoplm, FreeBOM, we-have-pdm-at-home,
  OpenPartsLibrary, PyPDM, line 1633) — confirmed consistent by this
  read; no correction needed to that edge.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:nanoplm` (line 413): category
`[plm]`, `technical_approach: unknown`, status `active` as of
2025-07-21, `scope: freecad-native`, origin unknown, license unknown,
`first_seen: unknown` — consistent with the file's own top-of-document
note that this cluster received "a lighter pass" (stub-level
treatment) rather than full write-up.

**Corrections applied to `graph.yaml` in this pass** (see below):
- `technical_approach: unknown` → `[sql-database]`, confirmed
  (Flask-SQLAlchemy over SQLite, `__init__.py`).
- `license: unknown` → `"MIT"`, confirmed (`LICENSE` header).
- `origin_country: unknown` → `"DE"`, confirmed (README's own contact
  block: "ALSADO Inh. Aleksander Sadowski, Liebfrauenstraße 31, 53757
  Sankt Augustin, Germany").

Not corrected: `first_seen`/`status_as_of` — no strong internal
evidence found to pin an exact date beyond the `LICENSE` file's 2024
copyright year, which is suggestive but not conclusive for a
"first_seen" field.

## 7. Friction points observed firsthand

- **nanoPLM barely engages with this census's core PDM friction
  points (identity/versioning, concurrency) at all** — it has almost no
  version-control story and no multi-user story. Its real contribution
  is a different, adjacent idea: **using FreeCAD's spreadsheet-
  parametric mechanism as a database-driven configurator backend**,
  which is a genuinely useful, underexplored pattern for the specific
  niche it targets (small manufacturers with parametric product
  families, e.g. a range of blade holders differing only in a handful
  of dimensions) rather than for general CAD collaboration.
- **This is worth flagging as a reminder that "PDM/PLM for FreeCAD" is
  not one problem** — this census has now found tools solving file
  versioning (EasyPDM, Ondsel-Server, CADBase), git-native diffing
  (HistoryWorkbench, GitPDM), locking/presence (the whole Cluster 2
  debate), BOM/parts-catalog management (OpenPartsLibrary), design-
  review commentary (BCF), and now parametric product configuration
  (nanoPLM) — all under the same loosely-applied "PDM"/"PLM" label in
  the census's own category taxonomy. Worth a note for whoever
  eventually synthesizes this series: the category tags in
  `graph.yaml` (`pdm`, `plm`, `bom`) may be doing less discriminating
  work than the actual variety of problems being solved would justify.

## 8. Minimal-patch hypothesis

- **Not a source of any transferable identity/versioning/concurrency
  pattern** — its relevant contribution to this plan is architectural
  variety (§7), not a reusable mechanism for Cluster 1/2's questions.
- **The spreadsheet-driven configurator pattern (§4) is a genuinely
  portable idea, independent of nanoPLM's own thin PDM layer**: any
  FreeCAD-adjacent tool wanting to generate a family of derived
  artifacts (previews, STEP, drawings) from parameter sets stored in an
  external database could adopt the same technique — open a shared
  template headlessly, write named values into its Spreadsheet object,
  recompute, export. Cost: **small**, the mechanism itself is only a
  few dozen lines per export target; the harder part (as nanoPLM's own
  fragility shows) is template-authoring discipline, not code.
- **A concrete, low-cost recommendation for nanoPLM itself** (not a
  cross-project convention): the `files` columns being bare strings
  rather than a proper `File` relation is a real, easily-fixed data-
  modeling gap — `File` already exists as its own model; wiring
  `Component`/`Instance`/`Client` to it via a real foreign key or
  association table would cost little and would remove a class of bugs
  (e.g. no way to represent "zero or many files" cleanly with a single
  string column) without touching anything else in the architecture.
