# Architecture notes — Ondsel-Lens-Addon (`FreeCAD/Ondsel-Lens-Addon`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read: `README.md` (in full), `models/file_version.py`,
`Workspace.py` (the core sync-state and upload/download logic, read
in full for the relevant sections), `WorkspaceView.py` (the
upload-confirmation flow). No git commands were run for this pass.

This is the FreeCAD-side client for `Ondsel-Server`, already given a
full architecture note in this series
(`doc/architecture/ondsel-server.md`). That note flagged a real,
verified gap: the backend has no conflict-prevention mechanism at all,
only a linear version-list-plus-"set active" model with no locking or
staleness check on write. **This client turns out to close a real
fraction of that gap at the UI layer, even though the backend itself
still does not** — a genuinely important nuance the backend-only read
could not have found.

## 1. Repo & basic facts

- **Language**: Python (PySide/Qt), a proper FreeCAD Addon-Manager
  workbench with a substantial, mature codebase (`Workspace.py` and
  `WorkspaceView.py` alone run into the thousands of lines combined),
  REUSE-compliant SPDX licensing throughout, CI (`.github/workflows/`:
  `black.yml` formatting check, `main.yml`, `reuse.yml` license-
  compliance check), and pre-commit hooks.
- **License**: LGPL-2.0-or-later — confirmed directly from the SPDX
  headers present in every file read (`README.md`, `models/file_version.py`,
  `Workspace.py`). `doc/ecosystem/graph.yaml` currently records
  `license: unknown`; corrected in this pass (see §6).
- **A real institutional history, documented candidly in the README's
  own "History" section**: this addon shipped with "Ondsel ES" (a
  custom FreeCAD flavor) to integrate with the `lens.ondsel.com` cloud
  service; when Ondsel the company ceased operations, both the addon
  and the Ondsel-Server backend were released under open licenses
  (AGPL for the server) and folded into the official `FreeCAD` GitHub
  organization. Development now continues under two named funding
  vehicles: the "Ondsel Onward Fund" (for the server) and an NLnet
  grant, "Lens/FreeCAD integration" (for this addon specifically) —
  i.e. this is not an abandoned or orphaned project despite its
  origin company's shutdown; it has active, named, external funding
  as of this snapshot. This corroborates and extends
  `doc/ondsel_server_issue_48.md`'s existing research thread with a
  fresher, addon-side data point.

## 2. Identity/versioning model

**A direct, faithful client-side mirror of the backend schema already
documented in `doc/architecture/ondsel-server.md`** — worth confirming
explicitly rather than re-deriving, since the match is exact:
`models/file_version.py`'s `FileVersion` dataclass
(`_id, createdAt, uniqueFileName, userId, message, fileUpdatedAt,
lockedSharedModels, additionalData`) reproduces the backend's
`fileVersionSchema` field-for-field, including the `lockedSharedModels`
field whose actual purpose (pinning a public share link to a specific
version, not an edit lock — see that note's §3) is preserved
unchanged here. This is a clean instance of a **schema mirrored
faithfully across a language boundary** (JavaScript/TypeBox on the
server, a Python dataclass on the client) — a concrete, working
example of the kind of "shared version-record shape" idea flagged as
worth generalizing in the CADBase and Ondsel-Server notes' own §8
sections.
- **A five-state file-sync status model**, computed purely client-side
  by comparing timestamps (`Workspace.py`'s `FileStatus` enum:
  `SERVER_ONLY` → "Not downloaded", `SERVER_COPY_OUTDATED` → "Local
  copy newer", `LOCAL_COPY_OUTDATED` → "Lens copy newer", `SYNCED`,
  `UNTRACKED`) — computed in `updateFileFound()` by a simple
  `serverDate`/`localDate` comparison on every model refresh. This is
  the most legible, well-labeled staleness-detection UI found in this
  entire census — clearer to a user than any of the raw hash/hidden-
  flag comparisons seen in CADBase's or the standalone cloud-browser
  clients.
- **Versioning itself remains exactly what the backend note already
  described**: uploading creates a new version and makes it "active";
  there is no branch/merge concept, consistent with Ondsel-Server's own
  strictly-linear model.

## 3. Conflict/concurrency strategy — a real, partial mitigation the backend alone doesn't have

**This is the most important, and most surprising, finding in this
note.** `doc/architecture/ondsel-server.md` concluded the backend has
*no* conflict-prevention mechanism — an unconditional last-write-wins
upload path with nothing checking whether the remote version changed
since the client last saw it. Reading this client directly shows that
conclusion is correct **for the backend**, but **incomplete for the
system as actually used**, because this addon adds a real, if soft,
safeguard on top:

- `WorkspaceView.py`'s `upload()` method explicitly branches on the
  computed `FileStatus` **before** calling the upload API:
  - `FileStatus.LOCAL_COPY_OUTDATED` (the server has a newer version
    than what the local copy was based on) → shows a confirmation
    dialog with the message **"The local copy is outdated compared to
    the active version... Uploading will override the server
    version."** — the user must explicitly confirm before the upload
    proceeds (`confirmUpload`/`confirmFileTransfer`).
  - `FileStatus.SERVER_COPY_OUTDATED` (local is ahead, the normal case)
    and `FileStatus.UNTRACKED` (first upload) proceed directly,
    prompting only for a commit message.
  - `FileStatus.SYNCED` short-circuits with a log message and does
    nothing.
- **This is a genuine, working instance of the "warn before you'd
  silently clobber someone else's change" pattern** — softer than
  EasyPDM's hard lock (the user *can* still override and upload
  anyway) but meaningfully stronger than the **zero** protection found
  in the standalone `freecad-cloud-browser` clients or in CADBase's
  FreeCAD/Blender clients (both of which upload unconditionally with no
  staleness check at all, confirmed directly in their own architecture
  notes). This is the best "sync-and-warn" implementation found
  anywhere in this census's cloud-sync-on-save family of tools.
- **The developers' own code candidly documents the deeper race this
  mitigation does not close**, in a comment directly above the
  vestigial, currently-disabled refresh call:
  > "TODO: in a shared setting refreshing is dangerous, suppose another
  > user pushes a file, then the index does not point to the correct
  > file any longer. First we refresh to make sure the file status have
  > not changed. `# self.refreshModel()`"

  The actual pre-upload refresh call is **commented out** — meaning the
  staleness check the whole confirmation flow depends on can itself be
  working from a stale read if enough time passed between the last
  refresh and the click on Upload. This is a specific, developer-
  acknowledged limitation of the mitigation, not a silent gap the
  architecture-note process had to discover independently — a rare,
  valuable case of a project's own source directly documenting the
  exact next race condition a more thorough fix would need to close.
- **Net assessment**: this client demonstrates that a meaningful
  fraction of Cluster 2's friction point can be addressed at the *UI/
  client* layer, without changing the backend's own storage model at
  all — a cheaper, more incremental path than building real locking or
  presence infrastructure server-side, with a known, explicitly-
  documented residual race (the disabled pre-upload refresh) as the
  honest cost of that cheapness.

## 4. File format / serialization touchpoints

- **The addon is entirely content-agnostic about file internals** —
  files are opaque blobs matched by `_id`/`uniqueFileName`; no FCStd
  parsing or diffing happens client-side (contrast `HistoryWorkbench`).
  Thumbnails (`thumbnailUrlCache`) and previews are pre-generated
  server-side (by Ondsel-Server's FC-Worker, per that note's §4) and
  simply displayed here.
- **A genuinely useful, uncommon feature**: an explicit **offline
  workflow** is a first-class, documented capability ("If a user is
  logged out, it is still possible to work on the files of workspaces
  that are currently on disk. As soon as a user logs into Lens, the
  user has the ability to upload the new versions") — i.e. the local
  filesystem is always the actual working copy, and Lens sync is
  explicitly optional/best-effort per session, not a hard requirement
  to keep working. This is a deliberate design value shared implicitly
  with GitPDM's "desktop user is sacred" principle (per that project's
  own `CLAUDE.md`), independently arrived at here.
- **Reloadable-file integration** (`integrations/reloadablefile/`) —
  not read in depth in this pass, but its presence suggests a mechanism
  for FreeCAD to detect and reload a document that changed on disk
  underneath an open session (relevant to the same staleness problem
  §3 addresses, from the opposite direction — local file changing
  under an open FreeCAD document rather than remote changing under a
  local copy). Worth a closer read in any future pass focused
  specifically on this addon.

## 5. Dependencies & integration points

- **`pyjwt`, `requests`, `tzlocal`** — a short, explicit dependency
  list (README), installed via FreeCAD's own Addon Manager dependency
  installer, with a documented common pitfall (`pip install jwt`
  installs the wrong package; must be `pyjwt`) — a small, concrete
  piece of practical FreeCAD-addon-packaging knowledge worth noting
  alongside GitPDM's own documented Addon-Manager dependency gaps.
- **JWT-based auth** against the Ondsel-Server backend, with a
  dedicated `TokenRefreshThread` — real, working session-refresh
  handling, not a login-once-and-hope model.
- **Sharing links** — the addon surfaces Ondsel-Server's share-link
  feature (already covered in that backend's own architecture note,
  §"Version Control..."/"Share-Link Workflow") directly in the FreeCAD
  UI, with "fine-tuned" permission controls (download formats, etc.)
  mentioned in the README but not independently verified in code in
  this pass.
- **No dependency on any other project in this census** beyond
  Ondsel-Server itself — a clean one-to-one client/server pairing,
  unlike CADBase's shared-module-across-two-clients pattern.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:ondsel-lens-addon` (line
388): category `[cloud-sharing]`, `technical_approach: [other]`,
`scope: freecad-native`, license `unknown`, `first_seen: unknown`.

**Corrections applied to `graph.yaml` in this pass** (see below):
- `license: unknown` → `"LGPL-2.0-or-later"`, confirmed directly from
  SPDX headers throughout the repository.
- `technical_approach: [other]` → `[server-checkin-checkout]`, matching
  the tag already used for `project:ondsel-server` itself — this addon
  is precisely a check-in/check-out client against that backend, and
  the existing `[other]` tag understates that relationship.

Not corrected: `first_seen: unknown` — SPDX copyright years in the
files read are uniformly "2024," which is suggestive but not
conclusive for a precise first-seen date; left as-is.

## 7. Friction points observed firsthand

- **This is the single best real-world instance in this census of
  "warn before overwrite" as a lighter-weight alternative to full
  locking**, sitting exactly between GitPDM's advisory-presence system
  (warns before you even start editing) and EasyPDM's hard exclusive
  lock (blocks the edit outright). It warns at the *moment of write*,
  after editing has already happened, which is a meaningfully different
  and arguably more practical point to intervene for a workflow that's
  fundamentally "edit offline, sync when convenient" rather than
  "always-connected collaborative editing."
- **The disabled pre-upload refresh (§3) is a small, precise, already-
  diagnosed bug/gap** — rare to find a project that has already located
  and commented its own next concurrency problem this clearly. Worth
  citing directly in any synthesis document discussing what "good
  enough" client-side conflict mitigation looks like and where its
  edges are.
- **This note materially changes how the earlier Ondsel-Server
  architecture note's concurrency conclusion should be read**: that
  note's §3/§7 correctly describe the *backend* as having zero
  conflict-prevention, but a reader should not conclude the *whole
  Ondsel/Lens system* offers users no protection at all — this client
  meaningfully improves on that at the point of actual use. Worth
  flagging as a general methodological note for this series: a
  backend-only or client-only read of a client/server PDM system can
  under- or over-state the system's real-world safety, and pairing
  both reads (as done here) is more reliable than either alone.

## 8. Minimal-patch hypothesis

- **The `FileStatus`-gated confirm-before-overwrite pattern is a
  small, concrete, directly transferable shim** any other cloud-sync-
  on-save FreeCAD addon in this census could adopt with minimal
  backend cooperation (only needs a last-modified timestamp per file,
  already present in every such system read so far). Cost: **small** —
  a client-side timestamp comparison plus one confirmation dialog; this
  is a strictly weaker requirement than building real server-side
  locking or presence, and would immediately close the exact silent-
  overwrite scenario flagged as a live risk in both the standalone
  `freecad-cloud-browser` architecture note and CADBase's own upload
  path. This is arguably the single most practical, lowest-effort
  patch surfaced anywhere in this census for the "naive cloud sync"
  failure mode specifically.
- **Fixing the disabled pre-upload refresh (§3, §7)** is an even
  smaller, already-diagnosed fix specific to this addon: re-enable
  `self.refreshModel()` immediately before checking `fileItem.status`
  in `upload()`, closing the TOCTOU gap the code's own comment
  describes. Cost: **trivial** — the commented-out call already exists;
  someone would need to resolve whatever concern caused it to be
  disabled (likely UI/index-invalidation side effects, per the
  surrounding comment) rather than write new logic.
