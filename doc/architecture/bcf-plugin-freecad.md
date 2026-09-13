# Architecture notes — BCF Plugin for FreeCAD (`podestplatz/BCF-Plugin-FreeCAD`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read: `README.md`, `release-notes.md`, `package.xml`, and source
across `bcfplugin/rdwr/` (`topic.py`, `modification.py`,
`interfaces/state.py`, `writer.py` excerpts) and
`bcfplugin/programmaticInterface.py` (excerpts). No git commands were
run for this pass.

This project is a structurally different animal from every other repo
read so far in this census: it doesn't version or lock CAD geometry at
all — it implements a **standardized, pre-existing interchange format**
(BCF, the buildingSMART BIM Collaboration Format) for exchanging
*issues/comments/viewpoints about* a model, not the model itself. It's
the first entry in this series where the "identity/versioning" and
"conflict/concurrency" questions are largely **already answered by an
external standard**, and the interesting architecture is in how
faithfully and robustly this plugin implements that standard's own data
model, not in any novel collaboration mechanism of its own.

## 1. Repo & basic facts

- **Language**: Python, a proper FreeCAD Addon-Manager plugin
  (`package.xml`, version `1.0.0`) with a genuinely substantial, well-
  organized codebase: a full BCF reader/writer library
  (`bcfplugin/rdwr/`), a documented "programmatic interface" (a
  non-GUI, scriptable API — `programmaticInterface.py`) separate from
  the GUI (`bcfplugin/gui/`), and a real test suite (`bcfplugin/tests/`,
  with dedicated fixture directories for reader/writer/interface/search/
  viewpoint tests).
- **Status**: **explicitly unmaintained** — the README's very first line:
  "❗Not actively maintained❗ Maybe some of the forks are better
  maintained than this repo. I unfortunately don't have the time to
  support it." `doc/ecosystem/graph.yaml` recording `status: dead` as of
  2024-02-09 is consistent with this self-assessment (though the repo
  clearly had a substantial, serious development period before that —
  see the design-doc evidence in §5).
- **Author**: Patrick Podest (`@podestplatz`), per copyright/author
  headers throughout the source (e.g. `topic.py`, `modification.py`:
  "Copyright (C) 2019 PODEST Patrick... Author: Patrick Podest, Date:
  2019-08-16"). `doc/ecosystem/graph.yaml`'s `first_seen: 2019-05-11`
  is consistent with this.
- **License**: LGPL-2.1, confirmed directly from `Licence.txt` (full GNU
  LGPLv2.1 text) — already correctly recorded in `doc/ecosystem/graph.yaml`;
  no correction needed.
- **Origin context**: per the file-level docstrings and `README.md`'s
  framing ("integrate collaboration in the BIM space"), this reads as
  serious, likely academic/thesis-grade software engineering — the level
  of design-pattern discipline visible in the reader/writer layer (§3-4)
  is unusual for a solo hobby project and closer in spirit to a
  university software-engineering project or dissertation artifact
  (not independently confirmed in this pass, but worth flagging as a
  plausible origin given the code's own character).

## 2. Identity/versioning model

**Identity and versioning here are not invented by this plugin at all —
they're inherited wholesale from the BCF XML schema itself**, which this
plugin implements faithfully rather than extends:

- Every meaningful BCF object (Topic, Comment, Viewpoint,
  DocumentReference, etc.) is a `UUID`-identified entity per the BCF
  spec (`from uuid import UUID` used throughout `rdwr/topic.py`) —
  identity is the standard's `Guid`/`Uuid` attribute, not anything this
  plugin's own code decides. The README itself warns the user directly
  about this: "If you stumble upon a member `id` in any object you
  retrieved from the plugin, don't modify it. The plugin uses this
  member to uniquely identify objects in the data model!" — a
  user-facing, load-bearing identity invariant stated in plain language.
- **BCF's own versioning concept is a simple, closed, monotonic
  chronology per comment/topic, not a general revision system**: every
  `Topic`/`Comment` carries a `CreationAuthor`/`CreationDate` pair and,
  separately, a `ModifiedAuthor`/`ModifiedDate` pair
  (`modification.py`'s `ModificationAuthor`/`ModificationDate` classes,
  parameterized by a `ModificationType` enum distinguishing `CREATION`
  from `MODIFICATION`) — i.e. **exactly one "last modified" slot, not a
  history list**. This is architecturally the simplest identity/version
  model read in this entire census: no revision letters, no embedded
  array of past versions (contrast Ondsel-Server's `fileVersionSchema`),
  no filename convention (contrast EasyPDM) — because a BCF topic/comment
  is conceptually a single, short-lived piece of correspondence (an
  issue or a comment on an issue), not a versioned artifact with a
  meaningful history of its own. The "history" that matters in BCF is
  the **sequence of comments on a topic over time**, each individually
  timestamped and immutable in spirit (a comment can be *modified*, but
  that just updates its one `ModifiedDate`/`ModifiedAuthor` pair, not a
  new entity) — closer to an email thread's model of history than a
  CAD file's.
- **A genuinely interesting, separate in-memory change-tracking layer
  sits on top of the BCF data model itself**: `interfaces/state.py`'s
  `State` class gives every object one of four states — `ORIGINAL`,
  `ADDED`, `DELETED`, `MODIFIED` — tracked purely **client-side, in this
  plugin's own runtime**, to know what needs writing back to disk on
  save. This is not part of the BCF standard at all; it's this plugin's
  own dirty-tracking mechanism, closer in spirit to an ORM's unit-of-work
  pattern than to anything else in this census. `writer.py`'s
  `addElement`/`deleteElement`/`modifyElement` (plus
  `handleAddElement`/`handleDeleteElement`/`handleModifyElement`) apply
  targeted, **incremental XML edits** to the underlying `markup.bcf`/
  `project.bcf` files based on each object's tracked state, rather than
  serializing the entire in-memory model back to XML on every save — a
  real, deliberate engineering choice for efficiency and (per
  `programmaticInterface.py`'s `_handleProjectUpdate`) safety: a failed
  write rolls the in-memory project back to a pre-update backup
  (`curProject = backup`) rather than leaving a half-written state.

## 3. Conflict/concurrency strategy

**Out of scope by the nature of the format, not solved or unsolved by
this plugin — worth stating precisely why.** BCF is fundamentally a
**store-and-forward interchange format**: a `.bcf` file is a self-
contained ZIP archive of one or more "topics" (issues), typically
authored by one tool, emailed or otherwise handed off, and opened by
another tool (or the same tool, later) — not a live, shared, continuously-
synchronized document the way a `.FCStd` file under GitPDM or
`HistoryWorkbench` is. Concretely, in the code and docs read:

- No lock, no owner field, no presence indicator, no session guard
  anywhere in the modules read — there is no concept of "someone else
  has this BCF file open" because the standard doesn't model concurrent
  editing of the *same* BCF file at all; the normal workflow is
  asynchronous exchange of whole files between parties (e.g. an
  architect exports issues from their BIM tool, sends the `.bcf` to a
  structural engineer, who opens it in a *different* tool — the actual
  scenario BCF was designed to standardize).
- **The "conflict" this plugin actually has to handle is schema
  validity, not concurrent edits**: `README.md`'s own documented
  behavior is that a non-standard-compliant node in an opened BCF file
  produces one XSD-validation error message per offending node (logged,
  with a pointer to a more detailed log file) and is **simply excluded
  from the in-memory model** rather than blocking the whole file from
  loading — "every node that does not comply with the standard is
  simply not read into the internal data model. That means you can't
  modify it, but still can add/update/delete to/from the model." This
  is a graceful-degradation strategy for **malformed input**, not a
  concurrency mechanism — a different kind of robustness problem than
  anything else in this census has needed to solve.
- **This is a fifth structurally distinct position on "conflict/
  concurrency strategy" for this census's purposes** (alongside EasyPDM/
  Anchorpoint/the Omniverse connector's locking, `ose-vcs-library`'s
  git-mergeable-text non-problem, OpenPartsLibrary's no-concurrent-
  editing-model, and GitPDM's built-then-rejected locking): **the
  problem doesn't exist at this layer because the artifact being
  exchanged (a BCF file) is designed from the start to be a discrete,
  point-in-time, single-producer snapshot, not a continuously co-edited
  document.** Worth treating as a reminder that not every "collaboration"
  format needs a concurrency story — some collaboration problems are
  legitimately solved by exchange-and-merge-manually at the human level,
  with the tooling's job limited to faithfully reading and writing the
  exchange format.

## 4. File format / serialization touchpoints

- **BCF itself, a ZIP of XML files per a published buildingSMART XSD
  schema** — this plugin validates every read node against the
  corresponding XSD (`xmlschema` dependency, `README.md`'s dependency
  list) and is explicit that a `bcf-examples/` directory of real,
  external sample files (including one literally named "test response
  from BIMServer.bcf.zip") is checked into the repo for testing against
  real-world producer output, not just self-generated fixtures — a
  genuine interoperability-testing discipline, appropriate for a
  standards-implementing library, that most other projects in this
  census (which invent their own formats) don't need.
- **IFC cross-referencing, not IFC parsing**: `getRelevantIfcFiles()`
  (`programmaticInterface.py`) and BCF's own "relevant IFC project"
  concept let a topic point at IFC model files by reference/GUID — this
  plugin doesn't parse or understand IFC geometry itself, it just
  surfaces the *reference*, consistent with BCF's own design as a
  lightweight companion format to (not a replacement for) the full BIM
  model.
- **The incremental-write architecture (§2)** is itself a real,
  non-trivial file-format-touchpoint finding: rather than treating the
  BCF zip as opaque and rewriting it wholesale on every save (the naive
  approach, and the one most other projects in this census take toward
  their own binary formats — e.g. EasyPDM's plain-attachment-replacement
  model), this plugin computes a minimal, targeted XML diff from its own
  `State` tracking and applies just that — closer in spirit to PR
  #28312's goal (minimize what actually changes on disk between saves)
  than anything else read in this census, though arrived at for a
  completely different reason (efficiency/safety of a library API, not
  git-diff-friendliness).

## 5. Dependencies & integration points

- **`xmlschema`, `python-dateutil`, `pytz`, `pyperclip`** — a short,
  standard, well-justified dependency list (`README.md`), each tied to a
  specific, named need (schema validation, date handling, clipboard
  support for the GUI). README also gives real, detailed instructions
  for the common FreeCAD-plus-virtualenv `sys.path` friction point
  (making FreeCAD's embedded Python aware of packages installed in a
  venv) — a practical, well-documented workaround for exactly the kind
  of Python-environment friction GitPDM's own `CLAUDE.md` separately
  documents hitting with FreeCAD's Addon-Manager allow-list.
- **Dual interface, cleanly separated**: a full non-GUI "programmatic
  interface" (`import bcfplugin as plugin; plugin.openProject(...)`,
  `getTopics()`, `getComments()`, etc. — a real, documented, scriptable
  API) plus a separate GUI frontend (`bcfplugin/gui/`, using FreeCAD's
  own Macro system as its launch point, `BCFPlugin.FCMacro`). This
  separation is a genuine, useful design choice: the plugin is usable
  from FreeCAD's Python console with no GUI at all, which is a real
  integration surface for any other project in this census wanting to
  read/write BCF issues programmatically (e.g. an EasyPDM/Ondsel-Server-
  style PDM tool wanting to surface BCF-format review comments alongside
  its own revision history — not built by any of them today, but this
  plugin's programmatic interface is a directly usable building block if
  one wanted to).
- **No server, no database, no network dependency of any kind** — like
  `ose-vcs-library` and OpenPartsLibrary, this is a purely local,
  file-based tool; unlike either of those, its "file" is a standardized
  interchange format meant to travel between organizations/tools, not a
  private local artifact.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:bcf-plugin-freecad` (line
265): category `[real-time-collab, bom]`, `technical_approach: [other]`,
status `dead` as of 2024-02-09, `scope: freecad-native`, license
"LGPL-2.1" (confirmed accurate), `first_seen: 2019-05-11` (confirmed
accurate against source copyright headers). No corrections applied to
license/dates/status.

**A category-tag caveat, not corrected** (matching the judgment-call
precedent set for OpenPartsLibrary's `pdm` tag in that architecture
note): `category: [real-time-collab, bom]` fits this project poorly on
both counts per this source-level read. **`real-time-collab`** is
arguably backwards — §3 above shows BCF is specifically an
*asynchronous*, store-and-forward format, closer in spirit to a
comment/issue-tracker interchange than to any real-time mechanism; **`bom`**
doesn't fit at all — nothing in the code read relates to bills of
materials, only to IFC file *references* and document *references*.
A more accurate categorization would be something like
`[bim-issue-tracking, async-collab]` or simply a new category this
graph doesn't yet have a slot for (BCF-style structured commenting is
genuinely distinct from every other category currently in use). Flagged
here for a taxonomy-level review rather than corrected directly, since
inventing new category values is a schema decision beyond a single
architecture note's scope — same reasoning applied to OpenPartsLibrary's
tag caveat.

## 7. Friction points observed firsthand

- This project demonstrates a collaboration pattern this census hasn't
  seen elsewhere: **standardize the *commentary* layer (issues,
  viewpoints, comments referencing a model) rather than the model or its
  version history.** Every other project in this census tries to solve
  "how do multiple people edit/version the same CAD file" — BCF sidesteps
  that problem entirely for a large, real class of collaboration
  (design review, clash detection, RFIs) by not requiring concurrent
  access to the same file at all. This is a genuinely different, and
  arguably more tractable, slice of "CAD collaboration" than anything
  else read so far, precisely because it doesn't touch the hard geometry-
  merge problem at all.
- The State-tracking + incremental-write architecture (§2, §4) is a
  well-engineered answer to a real, general problem — "efficiently and
  safely persist just the changes to a structured document" — that's
  independent of BCF specifically and could be read as reference
  material by any other project in this census wanting a similar
  dirty-tracking/incremental-save layer for its own data model (e.g.
  Ondsel-Server's MongoDB documents already get this for free from the
  database; a purely file-based project like EasyPDM's attachment
  handling or `ose-vcs-library`'s entries does not, and could learn from
  this pattern).
- The graceful degradation on schema-invalid input (§3) — log per-node,
  exclude from the model, keep going — is a small but genuinely
  reusable robustness pattern for any project in this census that reads
  externally-produced files it doesn't fully control (a real concern for
  any tool accepting BCF, STEP, or other externally-authored interchange
  formats from unknown producers).

## 8. Minimal-patch hypothesis

- **Not a candidate for anything in Cluster 2's locking/versioning
  discussion** — BCF's whole design premise removes the need for that
  discussion for the class of collaboration it addresses (§3, §7).
- **The most directly transferable idea here is architectural, not a
  drop-in shim**: the State (`ORIGINAL`/`ADDED`/`DELETED`/`MODIFIED`)
  plus targeted incremental-write pattern (§2, §4) is a genuinely
  reusable design for any future project in this census needing to
  persist partial changes to a structured, file-based (not database-
  backed) data model efficiently and safely. Cost to adopt: **small to
  moderate** — the pattern itself is simple (four states, a dirty list,
  targeted writes keyed by state), though implementing it against a
  different underlying format (e.g. FCStd's own internal XML, as PR
  #28312 or `HistoryWorkbench` touch) would require real, format-
  specific work, not a copy-paste.
- **A concretely actionable, low-cost integration opportunity worth
  naming explicitly**: since this plugin already has a clean,
  documented, non-GUI programmatic interface for reading/writing BCF
  topics/comments/viewpoints, any PDM-shaped project in this census
  (EasyPDM's revision-comment history, Ondsel-Server's notifications/
  history feed) could plausibly **read BCF files as an import source**
  for design-review comments, using this plugin's own
  `programmaticInterface.py` as a reference or even a dependency —
  rather than each project reinventing its own review/comment-thread
  concept from scratch, standardizing on BCF for that specific slice of
  "collaboration" would let interoperate with the broader BIM tooling
  ecosystem (Revit, ArchiCAD, BIMcollab, etc.) for free. This is a
  genuinely different flavor of "minimal patch" than anything else
  proposed in this series: not a convention to standardize on, but an
  **existing standard already implemented once, worth reusing rather
  than reinventing** — even though the plugin implementing it is itself
  unmaintained, its output format (BCF) is not, and remains a live,
  widely-adopted standard elsewhere in the BIM industry.
