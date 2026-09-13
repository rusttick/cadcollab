# Architecture notes — freecad-cloud-browser (`hitclawagent/freecad-cloud-browser`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read: `README.md`, `package.xml`, `LICENSE`,
`core/sync_manager.py` (in full), `core/file_cache.py` (in full),
`providers/base.py` (interface only), `core/auth_manager.py` (excerpt).
No git commands were run for this pass.

This project is a remote-storage file browser for FreeCAD — open/save
`.FCStd` and other CAD files directly against S3/FTP/WebDAV, with a
local cache and automatic re-upload on save. It is **not** a PDM system
in any of the senses the rest of this census uses the term: no
versioning, no revision concept, no numbering, no metadata schema
beyond a filesystem-like directory listing. Its main value to this
census is a clean, concrete illustration of what "collaboration" looks
like when a project doesn't engage with the conflict problem at all —
useful as a sharp negative baseline for Cluster 2, and as evidence for
`doc/ecosystem/graph.yaml`'s already-recorded "duplicated effort within
days" finding (see §6).

## 1. Repo & basic facts

- **Language**: Python, a FreeCAD Addon-Manager workbench (`package.xml`
  `version 1.0.0`, `freecadmin 1.0.0`) with a clean, small, well-
  organized module layout (`providers/`, `ui/`, `core/`) and a real
  (if modest) test suite (`tests/test_auth_manager.py`,
  `test_config_store.py`, `test_file_cache.py`).
- **License**: MIT — stated directly in `README.md`'s own "License: MIT"
  line and confirmed by the `LICENSE` file header ("MIT License,
  Copyright (c) 2026 hitclawagent"). `doc/ecosystem/graph.yaml` currently
  records `license: unknown`; straightforward correction (see §6).
- **Status/lifecycle**: `doc/ecosystem/graph.yaml` records `first_seen:
  2026-05-04`, `status: dead` as of `2026-05-07` — a **three-day
  lifespan**, the shortest of any project read in this census. The code
  itself, however, shows real engineering care disproportionate to that
  lifespan: `file_cache.py`'s docstring comments reference specific,
  numbered internal fix identifiers ("PERF-3 fix" appearing twice, for a
  read-only cache-path resolution avoiding an unnecessary `os.makedirs`
  syscall on a cache hit) — evidence of at least one real
  review/iteration pass, not a single unreviewed commit, despite the
  short overall window.
- **A same-named sibling project exists and is already captured in the
  graph**: `project:freecad-cloud-browser-sabi137032`, created three
  days later (2026-05-07), independently, with no visible relationship
  between the authors — `doc/ecosystem/graph.yaml` already records this
  correctly as `independently_reinvents` (line 2065), explicitly framed
  as "the third confirmed instance of 'duplicated effort within days,
  same name, no cross-awareness.'" Not independently re-verified in this
  pass (the sabi137032 variant wasn't downloaded/read), but this
  project's own existence and short lifespan are fully consistent with
  that framing.

## 2. Identity/versioning model

**None — files are addressed purely by remote path, with no concept of
a version, revision, or history at all.** This is the simplest possible
"identity" model in this census: a `RemoteItem` (per `providers/base.py`'s
interface) is just a path/name on a remote filesystem-like store (S3
bucket, FTP server, WebDAV share); there is no separate ID, no
UUID, no numbering scheme layered on top. `FileCache`'s own local
caching key (§4) is a SHA-256 hash of `provider_type + remote_path` — a
derived cache key, not an identity the user or any other part of the
system ever sees.

## 3. Conflict/concurrency strategy — the core finding

**There is none, and — unlike every other "no concurrency model" project
read so far in this census — this one is the first where that gap is
directly, structurally dangerous, not merely inapplicable:**

- `SyncManager` (`core/sync_manager.py`) implements a genuinely
  reasonable-sounding feature: register a locally-cached file with its
  remote provider/directory on open, hook FreeCAD's own
  `DocumentObserver.slotSaveDocument` signal, and **automatically
  re-upload the file to the cloud in a background thread on every
  save** — a real, working "edit locally, sync to cloud" loop, similar
  in spirit to Dropbox/OneDrive's local-sync-folder model.
- **The upload is a plain, unconditional overwrite** — `upload_file()`
  (checked in `providers/base.py`'s abstract interface and confirmed
  absent of any ETag/version parameter in `providers/s3.py`'s concrete
  signature) takes only a local path and a remote directory; there is
  no conditional-write mechanism anywhere in the code read (no S3 ETag
  precondition, no WebDAV `If-Match`, no "has the remote file changed
  since I downloaded it" check of any kind).
- **`FileCache.is_cached()`'s own staleness check (§4) only compares
  timestamps to decide whether to *download* a fresh copy before
  opening — it plays no role in the *upload* path at all.** So the
  actual failure scenario is concrete and easy to construct: two users
  open the same remote file (each gets their own local cache copy);
  User A saves, and `SyncManager` silently overwrites the remote copy;
  User B, still editing their own now-stale local cache, later saves,
  and `SyncManager` silently overwrites the remote copy *again* — with
  no warning to either user, and **User A's changes are gone with no
  record they ever existed.** This is a genuine, silent data-loss
  scenario, not merely an inconvenience or a "you'll notice and fix it
  manually" situation like `ose-vcs-library`'s git-mergeable-text
  approach or OpenPartsLibrary's single-writer model.
- **This is the sharpest possible negative baseline for Cluster 2 in
  this whole census.** Every other "no concurrency control" project read
  so far had a structural reason the gap was benign: `ose-vcs-library`
  (git text merges fine), OpenPartsLibrary (no concurrent-editing loop
  exists at all), BCF (the format is inherently single-producer/
  store-and-forward). This project has the *exact* edit-then-sync loop
  that makes conflicts a live, recurring risk — the same loop GitPDM
  built continuous checkpointing and advisory presence specifically to
  make safe — and simply doesn't address it. Worth treating as a
  concrete illustration of **why** GitPDM's presence/checkpoint design
  (§3 of that architecture note) exists, rather than as a defect unique
  to this one small project: naive cloud-sync-on-save is the natural,
  easy-to-build first approach, and it silently loses data the moment
  two people actually collaborate through it.

## 4. File format / serialization touchpoints

- **Fully format-agnostic** — the browser lists and transfers whatever
  files exist remotely; FreeCAD-format-awareness is limited to a
  filter/allowlist (`filter_freecad_files()`,
  `is_freecad_compatible()` in `providers/base.py`) used purely for
  **display filtering** (show `.FCStd`/`.step`/`.stp`/`.iges`/etc. in the
  browser panel), not for any content-aware processing. No FCStd
  internals are read or written by this project at all.
- **`FileCache`'s staleness model** (§3) is a real, if narrowly-scoped,
  serialization-adjacent design: it normalizes a wide variety of
  remote-provider timestamp formats (Unix epoch as string, ISO-8601
  with `Z`, with a numeric offset, or with no timezone at all) into UTC
  for comparison — a small but genuinely careful piece of defensive
  parsing (`file_cache.py`'s `is_cached()`, with an explicit fallback
  to "keep the cached file" if the timestamp can't be parsed at all,
  logged at debug level rather than raising). This kind of
  multi-provider timestamp-format normalization is a reusable, narrow
  utility any other project in this census integrating with multiple
  cloud/remote-storage backends would independently need to solve.

## 5. Dependencies & integration points

- **Three remote-storage protocols behind a common `CloudProvider`
  abstract base class** (`providers/base.py`): S3 (and S3-compatible
  services — MinIO, Wasabi, Backblaze B2), FTP/FTPS/SFTP, and WebDAV
  (Nextcloud/ownCloud-compatible) — a real, if narrower, echo of
  GitPDM's own multi-provider abstraction pattern (`ProviderCapabilities`/
  `BaseProvider`), applied to raw file storage rather than git hosting.
- **Credential storage follows the same layered pattern GitPDM
  independently built**: system keyring first (Windows Credential
  Manager / macOS Keychain / libsecret via `keyring`), falling back to
  a `cryptography` (Fernet symmetric encryption) locally-stored key when
  no keyring is available — `auth_manager.py` even documents a **legacy
  migration path** (an old on-disk `.credential_key`/`secret.key` file
  migrated into keyring on first run, then deleted) — evidence of at
  least one real breaking-change iteration in this project's short
  lifetime, consistent with the "PERF-3" fix-numbering seen in
  `file_cache.py` (§1). This keyring-then-Fernet-fallback pattern is now
  confirmed independently arrived at by two projects in this census
  (this one and GitPDM) — worth treating as a settled, low-risk default
  for any future FreeCAD addon needing to store credentials locally.
- **No server, no database** — purely a client-side workbench talking
  directly to whatever remote storage the user configures; no
  cloud-browser-specific backend service of its own.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:freecad-cloud-browser-hitclawagent`
(line 1323): category `[cloud-sharing]`, `technical_approach: [other]`,
status `dead` as of 2026-05-07, `scope: freecad-native`, license
`unknown`, `first_seen: 2026-05-04`. Already correctly linked via an
`independently_reinvents` edge (line 2065) to the sabi137032 sibling
project, framed accurately per this read as duplicated effort within
days with no cross-awareness — no correction needed to that edge.

**Correction applied to `graph.yaml` in this pass** (see below):
`license: unknown` → `"MIT"`, confirmed directly from both `README.md`'s
own stated license and the `LICENSE` file header.

## 7. Friction points observed firsthand

- **This project is the clearest, most concrete illustration in this
  entire census of the "naive cloud sync" failure mode that motivates
  needing something like GitPDM's presence/checkpoint design or the
  EasyPDM/Anchorpoint/Omniverse-connector locking cluster in the first
  place.** It's not a strawman — it's a real, working, shipped (if
  short-lived) addon with the exact overwrite-on-save loop that silently
  destroys concurrent work. Worth citing explicitly in any synthesis
  document as the "what happens if you don't solve this at all" baseline
  Cluster 2's other, more careful projects are implicitly guarding
  against.
- **The keyring-then-Fernet-fallback credential pattern (§5) being
  independently arrived at by two unrelated projects (this one and
  GitPDM) is a genuine, small piece of converged-on best practice**
  worth flagging as settled rather than open — a concrete instance of
  exactly the kind of "small, independently-adoptable convention" this
  whole technical-convergence phase is looking for, just for a
  credential-storage problem rather than a CAD-versioning one.
- **The three-day lifespan alongside real signs of iteration (§1)**
  is worth a brief methodological note for how this census's "status:
  dead" field should be read going forward: a short `first_seen`-to-
  `status_as_of` window does not necessarily mean an unreviewed,
  low-effort commit — this repo was abandoned quickly, but not before
  at least one documented internal review pass. "Dead" and "immature"
  are not synonyms, and this project is a clean example of that
  distinction.

## 8. Minimal-patch hypothesis

- **The single most concrete, actionable finding for this specific
  project is a defensive fix, not a shared convention**: adding even
  the simplest possible conflict guard to `SyncManager`'s upload path —
  e.g. fetch the remote file's current modification time/ETag
  immediately before uploading and compare it to what was downloaded,
  refusing (or warning) on mismatch — would close the silent-data-loss
  scenario in §3 at very low cost, without requiring anything as
  elaborate as GitPDM's full presence/checkpoint system. This is a
  smaller, more surgical version of the same insight GitPDM's own
  architecture note already generalizes (§8 there): a lightweight,
  advisory conflict signal goes a long way further than the
  literally-nothing this project currently has.
- **Not a source of a transferable *positive* pattern for Cluster 2** —
  its value to this census is entirely as a cautionary baseline (§7),
  not as an architecture worth propagating.
- **The credential-storage pattern (§5, §7) is the one piece of this
  project genuinely worth citing as a mature, reusable convention** —
  independently validated by GitPDM, low-cost to adopt (a documented
  fallback chain: keyring → encrypted local file → plain-text last
  resort with a loud warning), and unrelated to the CAD-versioning
  problem entirely, making it a clean, separable recommendation for any
  future FreeCAD addon handling credentials, regardless of what else
  that addon does.
