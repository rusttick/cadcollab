# Architecture notes — Taack PLM workbench for FreeCAD (`Taack/taack-plm-freecad`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
The client-side repository was read in full: `README.md`, `Intranet.py`
(194 lines — the entire application logic), `freecad_plm_pb2.py` (a
generated Protocol Buffers module, not hand-written). No git commands
were run for this pass.

This is a **thin transport client** for a separate, undownloaded
server repository (`github.com/Taack/plm`, a "Taack PLM Intranet"
application) — most of the actual PDM behavior (versioning, comments,
status changes, history browsing) this project's README describes
happens server-side, in a web UI this client only launches the browser
into, not in the FreeCAD-side code itself. This note is scoped and
sized accordingly: it documents what the client does and infers what
it implies about the server's design, without claiming to have read
the server.

## 1. Repo & basic facts

- **Language**: Python, a small FreeCAD workbench (`Intranet.py`, 194
  lines is effectively the whole application) plus a generated
  Protocol Buffers module (`freecad_plm_pb2.py`, 50 lines, produced
  from a `.proto` schema not included in this repo — presumably defined
  and versioned in the separate server repository).
- **License**: GPLv2, confirmed directly from `LICENSE`'s header —
  `doc/ecosystem/graph.yaml` currently records `license: unknown`;
  corrected in this pass (see §6).
- **Origin**: French — `doc/ecosystem/graph.yaml`'s existing note
  ("Announced by its own developer in a French-language LinuxFr.org
  post, 2023-02-09, 'Petit PLM'" — i.e. "Little PLM") is consistent with
  the small, focused scope of the actual code read here.
- **A real forum presence and part of the official Addon collection**:
  the README links a FreeCAD forum thread and states the workbench "is
  part of the FreeCAD addons collection and can be simply installed
  from the Addons manager" — a legitimate, distributed addon, not an
  obscure unlisted repo.

## 2. Identity/versioning model

**The one genuinely distinctive design choice in this whole project**:
identity is FreeCAD's own **native, built-in document UUID**
(`obj.Uid`), not a separately manufactured identifier —
`createDocProtobuf()`: `plmFile.id = obj.Id if obj.Id else obj.Uid`.
Every other project in this census that needs a stable per-document
identity either invents its own (EasyPDM's sequential item number,
Ondsel-Server's Mongo `_id`, CADBase's UUID columns) or relies on a
filename convention (the git-native cluster). This is the only project
found so far that reaches directly for a property FreeCAD itself
already assigns to every document, rather than adding a parallel
identity layer on top. Whether the separate server repository then
does anything further with that UUID (a real revision/version scheme,
a database key, etc.) is not verifiable from this client-only read —
flagged as a gap for a future pass if `Taack/plm` is ever downloaded.

Versioning itself is described only from the outside, via the README's
usage instructions: "Either the file Uid does not exist on the server →
uploaded as new... Either the file Uid does exist → the existing model
will be updated," and separately, downloading offers "the latest
version" or "a previous version" with an access-history view — both
of these interactions are driven entirely by the server's own web UI
(the screenshots referenced in the README are all of the Intranet's
browser pages, not FreeCAD dialogs), not by any code in this
repository. The actual version-storage mechanism is therefore **out of
scope for this repo's own architecture** — it lives entirely in the
undownloaded server.

## 3. Conflict/concurrency strategy

**Not addressed anywhere in this client, and not verifiable for the
server from this pass.** `Intranet.py` contains no lock, staleness
check, or confirmation-before-overwrite logic of any kind (contrast
`Ondsel-Lens-Addon`'s explicit `FileStatus`-gated confirm dialog,
read immediately before this project in the same session) — uploading
simply serializes the active document and its full link tree into one
Protobuf `Bucket` message and POSTs it, unconditionally
(`uploadCurrentActiveDoc()`). Per the README's model/status workflow
("Add comment OR change model status"), some server-side lifecycle
concept clearly exists, but nothing in the client enforces or even
displays it before an upload proceeds. This is consistent with the
client's overall shape as a bare transport layer — whatever
concurrency story exists for Taack PLM as a whole would need to be
enforced server-side, in code not read in this pass.

## 4. File format / serialization touchpoints

- **Protocol Buffers over HTTP, not JSON/GraphQL** — the only project
  in this entire census to use Protobuf as its client/server wire
  format (every other multi-tier project here — Ondsel-Server, CADBase,
  GitPDM's REST calls to git hosts — uses JSON-based REST or GraphQL).
  This is a real, distinct technology choice worth noting for
  completeness, though the generated `.proto` schema itself isn't
  present in this repo to inspect directly.
- **Whole-assembly, single-request upload**: `createBucketProtobuf()`
  walks the active document's `App::Link` objects recursively
  (`createLinkProtobuf`, with an `avoidLoop` list guarding against
  cycles), building one `Bucket` protobuf message containing every
  linked file's metadata (creation/modification dates and authors,
  taken directly from FreeCAD's own document properties) **and raw
  file bytes** (`plmFile.fileContent = open(obj.FileName,
  'rb').read()`) in a single message, POSTed as one multipart upload
  (`plm/uploadProto`). This is architecturally distinct from every
  other assembly-upload flow in this census (EasyPDM, CADBase, and
  `versioncontrol-workbench` all upload files individually, one HTTP
  request per file, sometimes via presigned URLs) — Taack PLM instead
  batches an entire assembly's files into one atomic transport unit.
  Tradeoffs are the usual ones for this shape: simpler client logic and
  atomicity (all-or-nothing for the whole tree in one request), at the
  cost of no per-file progress/resume and a request size that scales
  with the whole assembly rather than just what changed.
- **File content and CAD-native metadata travel together** — unlike
  CADBase or Ondsel-Server (where metadata lives in a separate
  database record pointing at object storage), here FreeCAD's own
  `CreationDate`/`CreatedBy`/`LastModifiedDate`/`LastModifiedBy`
  properties are read directly off the live document object and
  embedded in the same message as the file bytes — a simpler, more
  tightly-coupled design that trusts the CAD file's own embedded
  metadata as the source of truth rather than maintaining a parallel
  record.

## 5. Dependencies & integration points

- **`protobuf` and `requests`** — the only two Python dependencies the
  README lists, both installed via FreeCAD's own dependency mechanism.
- **A separate, undownloaded server application** (`Taack/plm`,
  described as an "Intranet" app) — this client's entire value depends
  on that server; nothing about its identity/versioning/concurrency
  model could be verified in this pass.
- **No sharing/collaboration features visible client-side** — no
  share-link mechanism (contrast Ondsel-Server/Ondsel-Lens-Addon), no
  multi-provider abstraction (contrast GitPDM), no offline-workflow
  design (contrast Ondsel-Lens-Addon) — this is a narrowly-scoped,
  single-purpose upload/download bridge, consistent with the
  developer's own "Petit PLM" ("Little PLM") framing.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:taack-plm` (line 762):
category `[plm]`, `technical_approach: [other]` (left as-is — genuinely
the best fit given the Protobuf transport doesn't match any more
specific existing tag), `scope: freecad-native`, origin FR (confirmed
consistent with the existing LinuxFr.org citation), license `unknown`,
`first_seen: unknown`, existing note about the 2023-02-09 LinuxFr.org
announcement.

**Correction applied to `graph.yaml` in this pass** (see below):
`license: unknown` → `"GPL-2.0"`, confirmed directly from the `LICENSE`
file header.

Not corrected: `first_seen: unknown` — no date evidence found in the
client repository itself beyond the graph's own existing 2023-02-09
citation, which the node doesn't currently surface into `first_seen`
for reasons not verifiable from this repo alone (possibly the
LinuxFr.org post predates this specific FreeCAD-side repo's own
creation) — left as a question for a future pass rather than guessed.

## 7. Friction points observed firsthand

- **The native-`Uid`-as-identity choice (§2) is a small, genuinely
  novel idea in this census's context**, worth citing on its own
  merits even though the rest of this client is thin: every other
  project's "invent a stable identity" problem could, in principle, be
  solved more cheaply by reaching for FreeCAD's own already-assigned
  document UUID first, falling back to a custom scheme only where
  FreeCAD's own identity isn't sufficient (e.g. tracking identity
  across a file being duplicated, which does create a new document
  with a new `Uid` — a real limitation of this approach that would need
  handling, not verified either way from this client-only read).
- **The whole-tree-in-one-request upload model (§4)** is a distinct,
  legitimate alternative to the per-file upload pattern this census's
  other multi-file-assembly projects all converge on — worth flagging
  as a real design-space alternative (simplicity/atomicity vs.
  per-file granularity) rather than a worse or better choice in the
  abstract.
- **This repo is a reminder that a thin, undownloaded server can hide
  most of a system's actual PDM behavior** — the plan's own repository
  list didn't include `Taack/plm` itself, and this read confirms that
  omission means the concurrency/versioning question for this specific
  project genuinely cannot be answered from what's available. Worth
  flagging explicitly rather than guessing: if a future pass wants a
  real answer for Taack PLM's identity/versioning/concurrency model,
  `Taack/plm` itself needs to be added to the download list.

## 8. Minimal-patch hypothesis

- **Not a strong source of a transferable pattern** given how much of
  the real behavior is server-side and unread — the one exception is
  the native-`Uid`-as-identity idea (§2, §7), which is cheap to state
  as a design recommendation independent of this project's own code:
  "check FreeCAD's own `Document.Uid` before inventing a new identity
  scheme" is a one-line piece of advice any future FreeCAD-PDM-tool
  author could act on directly, costing nothing to adopt and
  potentially saving a whole identity-scheme design effort.
- **No other actionable cross-project patch found** — this project's
  main contribution to the census is as a data point (a third wire-
  format choice, Protobuf, alongside REST/JSON and GraphQL) and a
  reminder of the plan's own coverage gap (§7) rather than a source of
  code or convention to propose elsewhere.
