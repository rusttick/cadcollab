# Architecture notes — FreeCAD-Omniverse Connector (`Metaverse-Colab-for-Fusion-Energy/FreeCAD-Omniverse`)

Read against `doc/technical_convergence_plan.md`'s 8-field template. Source
read: `README.md`, `docs/source/system/system.rst`, `docs/source/how_to/how_to_all.rst`,
`LICENSE.txt`, `src/FreeCAD-Omniverse/omniConnectorGui.py`, `file_utils.py`,
`utils.py`, `omniConnect/source/pyOmniFreeCAD/connectLiveTools.py`,
`session_toml_util.py`. No git commands were run for this pass; dates below
come from in-repo prose and `doc/ecosystem/graph.yaml`, not `git log`.

## 1. Repo & basic facts

- **Language(s)**: Python throughout — the FreeCAD workbench itself
  (`omniConnectorGui.py`, `file_utils.py`, `utils.py`, ~1,300 + 650 + 130
  lines) plus a bundled copy of NVIDIA's own **Omniverse Connect Sample**
  toolkit (`omniConnect/source/pyOmniFreeCAD/`, `connectLiveTools.py` at
  988 lines, `connectSampleLib.py`, `session_toml_util.py`, etc.) — the
  README says outright this includes sources from "Omniverse Connect
  Sample v202.0," MIT-licensed. Roughly 6,700 lines of Python total across
  `src/`.
- **Size**: small-to-mid, single-purpose workbench, not a general
  application — no database, no server component of its own (Nucleus,
  NVIDIA's own asset server, does that job).
- **Last real activity / maturity**: `doc/ecosystem/graph.yaml`
  (`project:freecad-omniverse-connector`) records `first_seen: 2023-10-31`,
  `status_as_of: 2026-06-24`, status `active`; the project is peer-reviewed
  (Journal of Open Research Software, 2025, linked directly from the
  README) and funded by EUROfusion/EPSRC/UKAEA — a government
  fusion-energy research context, not a hobby project. Tested FreeCAD
  versions per README: 0.20 through 1.0.1, **Windows only**.
- **License**: BSD-3-Clause, copyright The University of Manchester, 2024
  (`LICENSE.txt`) — matches `graph.yaml`'s recorded license. The bundled
  NVIDIA sample code under `omniConnect/` carries its own separate MIT
  license header (`session_toml_util.py`: "Copyright 2020 NVIDIA
  Corporation").
- **Has a real (if thin) test file**: `src/FreeCAD-Omniverse/tests/test_integration.py`.

## 2. Identity/versioning model

Fundamentally different from EasyPDM's: there is **no database and no
numeric item ID**. Identity is **path-based**, inside a Nucleus (NVIDIA's
asset-management server) folder tree:

```
omniverse://HOST/Projects/FreeCAD/$PROJECT_NAME/assets/$ASSET_NAME/
omniverse://HOST/Projects/FreeCAD/$PROJECT_NAME/assembly/$ASSEMBLY_NAME.usda
```

(or under `Users/$USERNAME/...` for private projects) — an asset's
identity *is* its Nucleus path, same as a file in a shared drive; there's
no separate identity layer above the filesystem the way EasyPDM's UUID +
`item_number` is layered above `item_attachments`.

**Versioning is entirely delegated to Nucleus's own checkpoint
mechanism** (comparable to Perforce/SVN revisions built into the asset
server) rather than being modeled by this project's own code at all. Per
`system.rst`: every upload/download/assembly action tags the resulting
file's checkpoint with a **unique token** message
(`AddCheckpointToNucleusAsset`, `omniConnectorGui.py:197`) — this token
is the *only* thing tying together the two parallel artifacts this
connector always keeps for one logical asset: a **STEP file (authoritative
geometry, what FreeCAD actually reads back)** and a **USD file (updated in
lockstep, used for visualization/assembly in Omniverse)** — "STP and USD
files which are associated with the same task are identical and as such
can be used as a way to track different versions" (`system.rst`). Notably,
tessellation (STEP→mesh) happens **only once, at upload time** — pulling
back into FreeCAD always re-imports the STEP, never the tessellated USD,
specifically to avoid feeding FreeCAD lossy mesh geometry.

This is a real point of contrast with EasyPDM worth flagging for field 8:
EasyPDM invented its own (fragile) filename-based version convention
because Postgres/plain-file storage has no native versioning; this
project sidesteps that problem entirely by choosing a backing store
(Nucleus) that already versions files, and layering only a thin
"same-token = same version, dual format" convention on top.

## 3. Conflict/concurrency strategy

**Two distinct mechanisms, not one** — this nuances the census's
"lock, don't merge" / checkpoint-token framing already in
`doc/ecosystem/graph.yaml`:

- **Primary, everyday path (asynchronous, checkpoint-based)**: Upload and
  Download are ordinary asset-server operations with no explicit lock —
  Nucleus permission checks (`GetCurrentSTPPermissions`/`GetAuthCheck`,
  `file_utils.py`) gate who can write where, and every write creates a new
  checkpoint rather than overwriting history. This is closer to "commit to
  a version-controlled path" than to EasyPDM's owner/lock-field model —
  there's no `owner_locked` equivalent gating concurrent edits; two users
  uploading to the same asset path in quick succession would simply create
  two checkpoints, the second winning, with no conflict signal raised to
  either user. So calling this "lock, don't merge" is only half right: it's
  really **"neither lock nor merge — last-write-wins with a full history
  trail"** for the normal STEP/USD upload-download flow.
- **Secondary path, "Live assembly mode"**: this is where actual real-time
  multi-user concurrency is handled, and it's wholesale NVIDIA
  Omniverse-sample machinery (`connectLiveTools.py`, `session_toml_util.py`,
  unmodified copyright headers), not something this project's own authors
  built. A live session has a **single owner recorded in a session TOML
  file** (`session_toml_util.py`, `OWNER_KEY = "user_name"`); USD supports
  live collaborative editing natively via a "live layer" that every
  participant's changes are composed into in real time (this is USD/Kit's
  own CRDT-like layer-composition model, not a merge algorithm this repo
  wrote). Only the session owner can `end_and_merge_session()`
  (`connectLiveTools.py:503`, checked in code, not just convention) —
  which flattens the live layer onto the root layer
  (`UsdUtils.FlattenLayerStack` + `Sdf.CopySpec`) and checkpoints both
  before and after, so a merge is itself always reversible via Nucleus
  history.
  - **On the FreeCAD side specifically, this is one-directional** — per
    `how_to_all.rst` §"Connecting with a live session," toggling "Live
    assembly mode" in the FreeCAD Assembly Panel only *streams* component
    transform updates from the Omniverse live session *into* FreeCAD in
    real time; nothing in `omniConnectorGui.py` pushes FreeCAD edits back
    into the live layer. `graph.yaml`'s own note already states this
    ("one-directional live-assembly streaming") — confirmed here by
    reading the actual GUI code, which only calls
    `GetAvailableLiveSessions`/subscribes, never authors into the live
    stage from the FreeCAD process.

## 4. File format / serialization touchpoints

- **STEP is the authoritative interchange format** for actual geometry —
  exported/imported via FreeCAD's own `Import` module
  (`Import.export`/`Import.insert`), not a custom serializer.
- **USD (`.usda`/`.usd`) is the secondary, Omniverse-native proxy format**
  — generated from the STEP on upload (tessellated once), used for
  visualization/assembly composition inside Omniverse/Kit apps, never
  read back into FreeCAD directly.
- Both formats are pushed/pulled through a **PowerShell-invoked batch
  fetcher** (`GetBatchFileName()`/`GetFetcherScriptsDirectory()`,
  `subprocess.Popen(['powershell', cmd], ...)`) wrapping NVIDIA's
  `omniClient` C++/Python bindings — i.e. the actual Nucleus protocol
  client is NVIDIA's, this project only shells out to it with constructed
  CLI flags (`--push_non_usd`, `--pull_non_usd`, `--add_checkpoint_to_usd`,
  etc.). This is a real integration cost/fragility point: cross-process,
  cross-language (Python→PowerShell→NVIDIA client), Windows-only.
- Assembly structure is represented as a `.usda` file per assembly
  (`$PROJECT_FOLDER/assembly/$ASSEMBLY_NAME.usda`) built from USD's own
  prim/reference composition — FreeCAD's own `App::Link` assembly model is
  translated into USD references on upload (inferred from the presence of
  `xform_utils.py` and assembly-panel code in `omniConnectorGui.py`
  handling per-component checkpoints, e.g. line 647,
  `"Add asset to assembly in ..."`).

## 5. Dependencies & integration points

- **Backend/server**: none of its own — NVIDIA Nucleus (a real,
  license-gated NVIDIA Omniverse server product) is a hard external
  dependency. This is a much heavier integration ask for an adopter than
  EasyPDM's self-hosted Postgres+API, since Nucleus is third-party
  infrastructure the project doesn't control or distribute.
  - Note: `README.md`/`system.rst` do not state whether a free/self-hosted
    Nucleus tier exists — worth verifying before treating this as a
    "just self-host it" option in any patch proposal.
- **FreeCAD integration**: a proper Addon-Manager-installable workbench
  (`InitGui.py`/`Init.py`, installed by copying into `Mod/`), unlike
  EasyPDM's bare macros — a materially lower-friction install path for a
  FreeCAD user.
- **Windows-only**, driven by hard subprocess calls to `powershell` and a
  `.bat` fetcher script — no Linux/macOS path shown anywhere in the code
  read.
- **External services**: NVIDIA Omniverse Connect Sample client (bundled
  MIT-licensed source, not a pip dependency), Nucleus server (external,
  not bundled).
- No CI configuration was found under this repo's top level in this pass
  (not confirmed absent, just not encountered — worth a second look if
  this becomes relevant to field 8).

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:freecad-omniverse-connector`
(line 1205): category `[real-time-collab, cloud-sharing]`,
`technical_approach: [server-checkin-checkout]`, status `active`,
`scope: opencascade-generic`, origin GB, license BSD-3-Clause,
`first_seen: 2023-10-31`. Same-author edge to `person:soemantoro` (line
1851-1855, confirmed). No `independently_reinvents` edge is anchored
*from* this node in the excerpt read, but it is the **target** of one
from `project:anchorpoint` (see the EasyPDM architecture note, §6, and
graph.yaml line ~2039) — the three-way "lock, don't merge" cluster
(EasyPDM ↔ Anchorpoint ↔ this project) the census already built. This
reading **refines rather than overturns** that edge: see §3 above — the
"lock, don't merge" label fits this project's Live-session merge gating
(single owner, explicit merge step) reasonably well, but doesn't fit its
default checkpoint-based upload/download path at all, which has no
locking of any kind.

## 7. Friction points observed firsthand

- The census's characterization ("Dual-format storage... Omniverse
  checkpoint-based version tagging, one-directional live-assembly
  streaming") holds up well against the actual code — this is one of the
  more accurate prose-only summaries encountered so far in this series.
- The real friction point this project has that EasyPDM does **not**: it
  depends on a proprietary, licensed external server (Nucleus) it neither
  controls nor ships, whereas EasyPDM is fully self-contained/self-hosted.
  Any shared convention spanning both projects has to work for a party
  that owns its whole stack (EasyPDM) *and* a party that treats its
  version-control backend as someone else's product (this project) — that
  asymmetry matters more than the surface-level "both use checkpoints/
  locks" similarity suggests.
- The dual-format-with-shared-token pattern (STEP authoritative + USD
  proxy, tied together by a checkpoint message string) is a genuinely
  reusable idea independent of Nucleus specifically — it's really "keep a
  human/machine-readable, human-visible tag correlating two parallel
  representations of the same version," which does not require USD or
  Nucleus to implement elsewhere.
- The Live-session code being verbatim NVIDIA sample code (not
  project-authored) means it's not a candidate for a "shared shim between
  independent FreeCAD-PDM projects" at all — it's Omniverse-specific
  plumbing, not a generalizable concurrency algorithm any of the other
  cluster members could realistically adopt.

## 8. Minimal-patch hypothesis

- **A shared "correlated dual-artifact" checkpoint/tag convention**
  (small cost, realistic): formalize the pattern this project already
  uses informally — when a tool keeps more than one file representation
  of the same logical version (e.g. EasyPDM's CAD file + its exported
  STEP/PDF attachments, or this project's STEP + USD pair), tag all of
  them with one shared, opaque version token in a predictable place (a
  checkpoint message here, an attachment-name convention in EasyPDM). This
  is genuinely portable — it doesn't require either project to adopt the
  other's storage model, just a naming/tagging discipline. Directly
  addresses a soft spot identified in the EasyPDM note (§8 there): "what a
  version even is" is exactly what this convention would pin down.
- **A shared locking/session-ownership convention across the "lock,
  don't merge" cluster** (moderate cost here, cheaper for EasyPDM/
  Anchorpoint-likes): this project's session-TOML ownership model
  (single named owner, checked server-side before allowing a merge) is
  structurally similar to EasyPDM's `owner_id`/`owner_locked`, but it's
  bound tightly to USD live-layer semantics and NVIDIA's client library —
  extracting a Nucleus-independent version would mean rewriting the
  ownership check against a generic key-value store, which is a bigger
  lift than EasyPDM's own trivial two-column implementation. Realistic
  framing: EasyPDM's model is the one worth standardizing *from*; this
  project's is worth citing as prior art/validation that the pattern
  generalizes, not as the basis for the shim itself.
- **Not realistic as a shared patch**: anything that assumes a
  self-hostable, project-controlled server — Nucleus is proprietary
  infrastructure this project doesn't own, so a convention requiring
  server-side enforcement (rather than a client-side tagging discipline)
  couldn't be adopted here without NVIDIA's cooperation.
