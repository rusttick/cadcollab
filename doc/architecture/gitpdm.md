# Architecture notes — GitPDM (`nerd-sniped/GitPDM`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read: `README.md`, `CLAUDE.md` (in full — an unusually detailed,
changelog-as-architecture-doc file maintained by the project itself),
`freecad_gitpdm/core/checkpoint.py`, `core/presence.py`,
`core/session_lock.py` (excerpt). No git commands were run for this
pass.

This is the second entry in priority group 2 (the git/version-control
cluster), and — like `HistoryWorkbench` — it wraps plain git rather than
inventing a server-side PDM backend. But its scope is broader and its
engineering maturity is the highest of any project read in this entire
census: multi-host provider abstraction (GitHub/GitLab/Bitbucket/
Gitea-Forgejo/SourceHut), OAuth device flow, a full credential-storage
abstraction across three OSes, and — most relevant to this plan — a
**deliberately-non-locking, git-native answer to concurrent-editing
awareness** that directly supersedes an earlier design the project
itself built, tested, and then removed. That removal-with-rationale is
the single most valuable data point in this document.

## 1. Repo & basic facts

- **Language(s)**: Python only, packaged as a FreeCAD Addon Manager
  workbench (`Init.py`/`InitGui.py`/`package.xml`), same distribution
  model as `HistoryWorkbench`.
- **Size/maturity**: `CLAUDE.md`'s own changelog (which the project
  maintains as its primary architecture record, in lieu of a separate
  design doc) shows sustained, active, patch-heavy development through
  versions v0.4.0 → v0.6.5, structured as `Dev_Docs/GITPDM_DEV_PLAN.md`
  phases G1 through G8. This is a real product-development process — CI
  on every push (`ruff` lint/format, pytest across Python 3.11/3.12 on
  Linux/Windows/macOS, and a custom `architecture_guard.py` enforcing
  per-file line-count budgets), tagged releases, and a working GitHub
  Actions release pipeline (build, container-smoke-test, publish).
  `doc/ecosystem/graph.yaml` records `first_seen: 2025-12-26`,
  `status_as_of: 2026-07-26` — both consistent with what `CLAUDE.md`
  itself documents.
- **License**: MIT (`README.md`, `LICENSE`) — matches `graph.yaml`; no
  correction needed.
- **Downstream integration signal**: `CLAUDE.md` mentions a real bug
  (v0.6.4) "reported from a downstream integration project (Outpost,
  which bakes a pinned GitPDM tag)" — i.e. this project is already being
  embedded/consumed by at least one other, unnamed-in-this-census
  project, which is itself a live instance of the kind of
  low-cost/independently-adopted convergence the technical-convergence
  plan is looking for, though "Outpost" doesn't appear in the plan's own
  repository list and wasn't investigated further in this pass.
- **A live, explicit design retraction is documented**: "G3 (storage
  modes) was retired 2026-07-20" — the project built, then deleted, a
  Git-LFS-based file-locking storage mode once it concluded the real
  problem didn't require it (§3 below). This kind of frank
  build-then-retire record is rare in the wild and unusually valuable
  for this census's purposes.

## 2. Identity/versioning model

**Git commits are the version store, exactly as with `HistoryWorkbench`
— no separate numeric/UUID scheme layered on top.** A `.FCStd` file's
identity is its repo-relative path; its history is its git log. GitPDM
adds two things on top of plain git, both worth treating as genuinely
novel contributions to this census rather than restatements of the
git-native pattern:

- **Continuous checkpointing** (`core/checkpoint.py`) — an
  "Onshape-style 'walk away anytime, lose ≤~1 minute' guarantee," via
  debounced auto-commits onto a dedicated `gitpdm/recovery` branch,
  never touching the user's real commit history. The scheduling logic
  (`should_checkpoint`) fires on **whichever comes first**: an idle
  window (45s of no edits, tunable within a documented 30-60s band) or a
  max-interval backstop (180s since the last checkpoint, so continuous
  active editing without ever going idle still gets periodically
  captured). This is a real answer to a problem every project in this
  census implicitly has and none of the others address: **FreeCAD's
  native save is a blocking, whole-file operation**, so per-keystroke
  persistence (the naive "just save more often" answer) is not viable —
  the module's own docstring states this design constraint explicitly,
  citing `Dev_Docs/GITPDM_DEV_PLAN.md`'s R2.5 rationale.
- **A genuinely subtle bug-and-fix pair, documented candidly in
  `CLAUDE.md`, worth citing in this note because it generalizes**: an
  early version of the checkpoint mechanism gated the actual `doc.save()`
  call on FreeCAD's `Document.isTouched()`, on the assumption that
  "not touched" meant "no unsaved changes." It doesn't —
  `isTouched()` tracks FreeCAD's *recompute dependency graph*, which
  settles almost immediately after an edit (well inside the 45s
  idle-debounce window), so this gate silently skipped the real save on
  "close to every real checkpoint," producing a git commit that looked
  successful but re-snapshotted stale, pre-edit disk content — a bug a
  real user hit and reported, that would have been invisible from
  outside (`CLAUDE.md`: "every checkpoint still committed *something*...
  just a re-snapshot of already-stale disk content"). **This is directly
  relevant to any other project in this census that queries FreeCAD's
  document-dirty state to decide when to save/version/checkpoint**
  (conceivably `HistoryWorkbench`, though not confirmed to use the same
  API in this pass) — a documented, hard-won pitfall worth flagging
  explicitly as prior art before any future project reinvents the same
  mistake.
- **Non-destructive restore, matching a pattern now confirmed across
  three projects in this census** (EasyPDM's revision-bump-not-delete;
  `HistoryWorkbench`'s "restore without rewriting history"; here,
  `restore_recovery_checkpoint`/`export_recovery_snapshot`, the latter
  explicitly built because an earlier in-place-only restore "left no
  visible proof of what, if anything, actually happened" per a real user
  report) — this is now a strong enough cross-project convergence to
  treat as a settled, near-universal design value in this whole census,
  not a coincidence: **never destroy prior state to satisfy a "go back"
  request; always leave a browsable trace.**

## 3. Conflict/concurrency strategy

**This is the single most important section of this note for the
plan's purposes.** GitPDM's current answer to concurrent editing is
**advisory presence, explicitly not locking** — and the project's own
`Dev_Docs/PRESENCE_AND_LFS_REMOVAL_PLAN.md` (referenced repeatedly in
both `core/presence.py` and `CLAUDE.md`, not independently read in this
pass but extensively summarized by both) documents *why* it abandoned
real locking after having actually built toward it:

- **What was tried and removed**: a full "storage mode" system
  including a Git-LFS-backed "lfs" mode whose entire justification was
  **real file locking** (`supports_lfs_locking`). Per `CLAUDE.md`: "Real
  Git LFS file locking... was never actually implemented on any
  provider (`supports_lfs_locking` was `False` everywhere, permanently
  deferred)." On closer analysis, the project concluded **locking's real
  value is preventing *wasted editing effort*, not preventing data
  loss** — because checkpoints/recovery (§2) already make every state
  recoverable, so there is no unrecoverable-data-loss scenario locking
  would actually be preventing. "`.FCStd` conflicts are always manually
  reconcilable (git history is never lost)" (`presence.py` docstring).
- **What replaced it**: `core/presence.py`'s advisory, cross-user "who
  else has this file open" indicator — "warns, never blocks." Built
  entirely on git itself: a dedicated `gitpdm/presence` branch holding
  one small JSON file (`open-files.json`), written via raw git plumbing
  (`hash-object`/`mktree`/`commit-tree`/`update-ref`, the same low-level
  approach `checkpoint.py` uses for its own recovery branch) rather than
  porcelain commands — i.e. **the git remote itself is the presence
  channel**, no external server, no lock-file service, nothing beyond
  what every collaborator already has access to. A CAS-style
  (compare-and-swap) write (`update_ref_cas` with an `expected_old_sha`)
  with one bounded retry handles the read-modify-write race between two
  users announcing/closing at nearly the same time — explicitly *not*
  retried indefinitely, "since this is advisory data, not a guarantee."
- **This is a fourth, structurally distinct data point on the "lock,
  don't merge" question this census keeps returning to** — and the most
  informative one, because it's not a project that never needed locking
  (`ose-vcs-library`, OpenPartsLibrary) or hasn't gotten there yet
  (`HistoryWorkbench`'s roadmap). It's a project that **built real
  locking machinery, then deliberately tore it out** after concluding the
  actual user-facing value (avoid wasted work) didn't require the
  mechanism (exclusive locks) the EasyPDM/Anchorpoint/Omniverse-connector
  cluster converged on. This directly challenges the framing implicit in
  `doc/ecosystem/graph.yaml`'s `independently_reinvents` "lock, don't
  merge" edge: at least one well-engineered, actively-maintained project
  looked at the same problem and concluded the pattern was solving the
  wrong thing.
- **Local-scope concern is handled separately and is explicitly not the
  same mechanism**: `core/session_lock.py` is a *local-filesystem/PID*
  advisory lock (`.git/gitpdm.lock`) guarding against **two GitPDM
  instances on one shared working tree** (e.g. two browser tabs against
  one hosted deployment) writing simultaneously — a different problem
  (single-machine process coordination) from cross-user presence
  (multi-machine awareness), and the module docstring is explicit that
  this too is "advisory, not enforced... it exists to warn, not to
  block. A determined second instance can always override." The project
  applies the same non-blocking philosophy at both scopes, consistently,
  not just at the cross-user one.

## 4. File format / serialization touchpoints

- **`.FCStd` is the only versioned artifact**, treated as an opaque
  binary blob by git itself — GitPDM does not attempt structural
  diffing of FCStd internals anywhere in the modules read (contrast
  `HistoryWorkbench`'s snapshot-extraction/property-diff engine, or PR
  #28312's document-cache-dir approach) — GitPDM's contribution is
  entirely at the *workflow* layer (when/how commits happen, presence,
  recovery), not the *content* layer.
- **A hard, explicitly-documented format constraint drives a real
  architectural guard rail**: "`.FCStd` files are ZIP archives. If the
  working tree changes under an *open* FreeCAD document, the file can
  corrupt" (`CLAUDE.md`, "Key behavioral constraint: branch switching
  safety") — so branch switching/checkout/worktree operations require
  documents closed first, a rule `CLAUDE.md` explicitly tells future
  contributors (or agents) not to relax without understanding why.
  `docs/README.md` has a dedicated explainer, "Why Branch Switching Is
  Tricky with FreeCAD Files" (not independently read in this pass, but
  referenced consistently enough across `README.md` and `CLAUDE.md` to
  treat as a real, documented finding). This is a **specific, concrete
  technical hazard of git-plus-FCStd** that neither `HistoryWorkbench`'s
  notes (not documented in this pass) nor `ose-vcs-library`'s
  never-commit-the-binary approach have to contend with in the same
  way — worth treating as a fifth data point (alongside PR #28312's
  cache-dir goal) on the general theme "binary CAD files and git don't
  naturally cooperate," and a specific enough hazard that any future
  shared convention for "how to git-manage FCStd files safely" should
  probably state this constraint explicitly.
- **Export/preview pipeline** (`export/`: `stl_converter.py`,
  `model_export.py`, `thumbnail.py`, `manifest.py`) produces
  shareable previews (STL mesh, thumbnail PNG) driven by an optional
  `.freecad-pdm/preset.json` — this is presentation/sharing tooling
  (e.g. a committed `preview.png` for a host's repo page), not a version
  or identity mechanism, and reads FreeCAD's own embedded thumbnail
  (`read_embedded_thumbnail()`) rather than rendering a fresh one via a
  custom viewport pipeline (an earlier custom-render pipeline was
  deleted in favor of this simpler approach, per `CLAUDE.md`).

## 5. Dependencies & integration points

- **`git` CLI via subprocess** (`git/client.py`) — the sole, deliberately
  host-agnostic layer every git operation goes through; `CLAUDE.md` is
  explicit that "no provider conditionals belong in here," with one
  documented, narrow exception (asking the active provider for its
  `credential_username` convention, since GitLab/Bitbucket disagree on
  what username to send alongside a PAT over HTTPS).
- **Five git-host providers** behind a `ProviderCapabilities`/
  `BaseProvider` abstraction (`providers/`): GitHub (full OAuth device
  flow, real REST API integration — rate limiting, caching, repo
  creation) as the mature reference implementation, and GitLab/
  Bitbucket/Gitea-Forgejo/SourceHut as PAT-paste-only peers (no
  pre-registered OAuth app exists for any of them, so PAT/SSH is
  documented as "the universal floor"). This is the broadest
  multi-host git integration of any project in this census by a wide
  margin — none of the other projects (including `HistoryWorkbench`,
  which doesn't push to a remote at all yet) integrate with git hosting
  services directly.
- **A real, cross-platform credential-storage abstraction**:
  Windows Credential Manager, macOS Keychain, and Linux
  (`secretstorage`/`keyring`, GNOME Keyring/KWallet) — each behind a
  per-OS module selected by a factory, plus a headless/container path
  (`GITPDM_TOKEN_FILE`/`GITPDM_TOKEN` env vars, resolved through a single
  `credential_chain.py` precedence order) explicitly built and tested
  for non-desktop deployments (`auth/check.py` is described as "the
  keyring-less container smoke test").
- **A genuinely interesting cross-project compatibility finding,
  self-documented**: `CLAUDE.md`'s v0.6.2 entry records that FreeCAD's
  actual Addon Manager Python-package allow-list does **not** include
  `secretstorage` or `keyring` — the exact two dependencies GitPDM's own
  credential storage needs on Linux/macOS — meaning an Addon-Manager
  install of GitPDM on those platforms silently can't auto-install its
  own credential backend, requiring a manual `pip install` into
  FreeCAD's own Python. This is a **real, concrete example of the kind
  of ecosystem-level friction point this whole census is looking for**
  — not between two competing PDM tools, but between a well-behaved
  addon and the FreeCAD Addon Manager's own dependency-management
  ecosystem. Worth flagging as a possible small, independently-useful
  patch target of its own (getting `secretstorage`/`keyring` added to
  FreeCAD's allow-list) — GitPDM's maintainer has drafted, but not yet
  confirmed filing, Package-Addition requests against `FreeCAD/Addons`
  for exactly this.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:gitpdm` (line 653): category
`[pdm, version-control]`, `technical_approach: [git]`, `scope:
freecad-native`, license "MIT" (confirmed), `first_seen: 2025-12-26`,
`status_as_of: 2026-07-26` — both dates consistent with `CLAUDE.md`'s own
detailed timeline. No corrections needed to this node.

Worth flagging for a future graph-consistency pass rather than corrected
here (no existing edge to revise, only an omission possibly worth
filling): this node currently carries **no `independently_reinvents` or
similar edge** to the EasyPDM/Anchorpoint/Omniverse-connector "lock,
don't merge" cluster, despite GitPDM being the one project in the census
that engaged with that exact pattern directly (built it, then reversed
course — §3). An edge capturing that relationship — perhaps a new type
like `evaluated_and_rejected`, or a `cites_as_prior_art` in the opposite
direction from what that edge type currently models — would make the
graph reflect a genuinely distinct and informative stance, not just
silence. Flagged as a suggestion, not applied, since inventing a new
edge type is a graph-schema decision beyond a single architecture
note's scope.

## 7. Friction points observed firsthand

- **The presence-not-locking design and its documented rationale (§3)
  is the most important single finding in this note for the plan's
  Cluster 2 question.** It's not merely a fourth data point against
  "lock, don't merge" — it's a *reasoned rejection*, made by a project
  that got further into building real locking machinery than most, with
  an explicit, load-bearing insight worth quoting directly in any future
  synthesis: locking's actual value is "avoiding wasted editing effort,"
  and that value is achievable through a much cheaper mechanism
  (advisory, best-effort, git-native presence) once recoverability
  (continuous checkpointing) has independently solved the data-loss
  half of the problem locking is usually justified by. This suggests the
  EasyPDM/Anchorpoint/Omniverse-connector cluster's locking may be
  solving a problem (data loss) that a good checkpoint/recovery story
  would solve more cheaply and more generally, while the *actual*
  residual need (avoid wasted effort) is served just as well by a much
  lighter warn-don't-block mechanism.
- **The FCStd-is-a-ZIP-and-corrupts-under-live-checkout hazard (§4)** is
  a concrete, load-bearing engineering constraint that any future shared
  convention for "safely git-manage FCStd" absolutely has to account
  for — it's not hypothetical; it's the reason GitPDM's own branch-
  switching code requires documents closed first, stated as an explicit
  do-not-relax-without-understanding-why rule in `CLAUDE.md`.
- **The FreeCAD-dirty-state pitfalls (§2: `isTouched()` misuse,
  `activeDialog()` returning a bool not an object) are exactly the kind
  of "small, hard-won, non-obvious FreeCAD API gotcha" that's expensive
  to rediscover independently** — worth treating as a reusable knowledge
  artifact for any future project in this census (or beyond) that needs
  to answer "is this FreeCAD document dirty/busy right now," independent
  of whether that project ever adopts anything else about GitPDM's
  architecture.
- **The Addon-Manager dependency-allow-list gap (§5)** is a real,
  specific, fixable piece of FreeCAD ecosystem friction, and one of the
  few friction points in this whole census that's fixable by petitioning
  a single upstream list rather than coordinating across multiple
  independent projects — a genuinely different, lower-effort category of
  "convergence fix" than anything else surfaced so far.

## 8. Minimal-patch hypothesis

- **Not a candidate for adopting the "lock, don't merge" pattern** —
  the opposite: this project is itself evidence *for* a specific
  alternative pattern (checkpoint/recovery + advisory presence) that
  could be proposed as a **lighter-weight substitute** for the locking
  convention other architecture notes in this series (EasyPDM,
  the Omniverse connector) considered proposing. Cost of adapting
  GitPDM's presence approach elsewhere: **small to moderate** — it needs
  (a) a git-backed remote (not applicable to EasyPDM's Postgres/HTTP
  model without an added git dependency) or an equivalent shared,
  low-latency broadcast channel, and (b) a periodic heartbeat mechanism.
  Realistically most directly transferable to other **git-native**
  FreeCAD tools (i.e. `HistoryWorkbench`, once/if it adds multi-user
  remote support) rather than to the server-based projects in priority
  group 1.
- **The continuous-checkpointing pattern (§2)** is a strong, portable
  idea independent of git specifically: "debounce on idle, backstop on
  max-interval, write to a disposable/recovery location, never touch
  the user's real history" is a general architecture any tool
  auto-saving a slow-to-save, easy-to-corrupt document format could
  adopt. Cost: **small**, the scheduling logic itself
  (`should_checkpoint`) is a pure, dependency-free function already
  isolated from FreeCAD specifics in this codebase — directly copyable
  as a reference implementation (MIT license, permissive).
- **Filing FreeCAD's own Addon-Manager allow-list gap (§5, §7)** is a
  concrete, low-effort, high-leverage action item this document
  surfaces that isn't really a "patch to another project" at all — it's
  a one-time upstream fix (`FreeCAD/Addons` Package-Addition requests
  for `secretstorage`/`keyring`) that would remove a real installation
  papercut for GitPDM and potentially any other addon needing OS
  keyring access, independent of anything else in this technical-
  convergence phase. Worth flagging explicitly as an actionable,
  non-code recommendation for the plan's eventual synthesis, distinct
  in kind from every code-shim proposal in the other architecture notes.
