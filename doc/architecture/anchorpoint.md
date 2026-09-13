# Architecture notes — Anchorpoint (proprietary, no public source)

Read against `doc/technical_convergence_plan.md`'s 8-field template, per
that plan's own note that Anchorpoint "still relevant to Cluster 2...
its architecture doc will have to be written from public documentation,
not source." **No source code exists to read** — Anchorpoint Software
GmbH's product is closed; this note is built entirely from the
company's own public blog posts and its GitHub organization's
repository list (fetched via WebFetch/WebSearch in this session, not
from any local download). Confidence should be read accordingly:
everything here reflects what the vendor states publicly about its own
product, not independent verification of the actual implementation.

Sources consulted:
- `anchorpoint.app/blog/git-with-freecad`
- `anchorpoint.app/blog/using-file-locking-in-git`
- `github.com/Anchorpoint-Software` (organization repository listing)
- A general web search for architecture/locking documentation

## 1. Repo & basic facts

- **Not a FreeCAD-specific tool at all** — confirmed directly by
  checking Anchorpoint Software's own GitHub organization: its public
  repositories are `ap-unreal` and `ap-unity` (game-engine plugins),
  `git-dist` (a bundled Git + a forked `git-lfs`), `ap-actions`
  (pipeline/workflow extensions), `ap-self-hosted-templates`,
  `cloud-drive` (S3-as-network-drive mounting), and `psd` (a forked
  Rust PSD parser) — **no FreeCAD plugin or CAD-specific repository
  exists**. Anchorpoint's "FreeCAD support" is simply that it is a
  general binary-file-aware Git client, and `.FCStd` happens to be one
  of the binary formats it handles without special configuration — the
  blog post the plan cites is a workflow guide, not evidence of a
  dedicated FreeCAD integration. This matches the plan's own framing
  exactly and is worth stating plainly rather than assuming a plugin
  exists.
- **Primary market is game development and creative/VFX teams**, not
  CAD — Anchorpoint's own GitHub presence, its Unity/Unreal plugins,
  and its blog's general framing ("Version Control for Designers &
  Creative Teams") all point at game-asset/DCC pipelines as the core
  product focus, with CAD/FreeCAD support being one workflow among
  several the same generic binary-locking mechanism happens to serve.
- **License**: proprietary/commercial — confirmed, matches
  `doc/ecosystem/graph.yaml`'s existing "proprietary" entry; no
  correction needed.
- **A notable adjacent project in the same GitHub org**: `lore`, described
  as "a next-generation, open source revision control system," forked
  from EpicGames — Epic has been developing its own open-source
  alternative to Perforce/git for large binary game assets. Anchorpoint
  maintaining a fork of this is a signal worth flagging for anyone
  tracking the broader "version control for large binary creative
  assets" space beyond just FreeCAD/CAD — not investigated further in
  this pass, out of scope for this plan's FreeCAD focus.

## 2. Identity/versioning model

**Standard Git identity and history — Anchorpoint does not appear to
replace or extend git's own commit/blob model with anything CAD-
specific.** Files are versioned as ordinary Git objects, with large
binaries routed through Git LFS. The product's own value-add sits
entirely in the *locking* layer (§3) and in developer-experience
polish (automatic thumbnail generation, tailored `.gitignore`/
`.gitattributes` defaults for CAD/creative workflows, a "project
timeline" showing activity including force-unlock events) rather than
in reinventing what a "version" of a file is. This puts Anchorpoint in
the same general category as `HistoryWorkbench` and GitPDM — a
polish/tooling layer over plain git — but, unlike either of those,
Anchorpoint is explicitly multi-DCC (not FreeCAD-specific) and
commercial rather than community-maintained.

## 3. Conflict/concurrency strategy

**Exclusive, preventative locking — the plan's own existing note
already characterizes this correctly, and public documentation
confirms real, specific mechanics beyond the one-line summary**:

- Locking is implemented via a **separate, proprietary "Anchorpoint
  Metadata Server"** — explicitly *not* Git LFS's own native locking
  (`git lfs lock`), which the company's own blog post calls out as "an
  entirely different system" with real limitations Anchorpoint says it
  avoids. The metadata server is described as never touching
  "production data" (i.e. the actual repository contents) — locking
  state lives entirely out-of-band from the git history itself, in
  Anchorpoint's own cloud service, layered on top of an otherwise
  completely standard git repository hosted on GitHub/GitLab/Azure
  DevOps.
- **This is an architecturally different choice from every other
  locking/presence mechanism found with actual source access in this
  census.** GitPDM's advisory presence (`doc/architecture/gitpdm.md`,
  §3) stores its state *inside* the same git repository, on a dedicated
  `gitpdm/presence` branch — visible to anyone who can read the repo,
  with no separate service required. Anchorpoint instead keeps locking
  state in a **vendor-operated, out-of-band service** the git repository
  itself knows nothing about. This is a real tradeoff worth naming
  explicitly: Anchorpoint's approach can plausibly be faster/more
  reliable (a dedicated service purpose-built for real-time lock
  queries, claimed to "lock 1,000 files in under a second") at the cost
  of a hard dependency on that vendor's service staying available and
  trustworthy — a git repository with Anchorpoint's locks stripped out
  reveals nothing about who was editing what, unlike GitPDM's
  self-contained, repo-native approach.
- **Locking is automatic and default-exclusive for binary files**:
  "binary files are automatically locked as soon as they are modified,"
  with no explicit "check out" action required — a stronger, more
  automatic version of the exclusive-lock pattern than EasyPDM's or
  Anchorpoint's own FreeCAD-workflow-guide framing ("check out to get
  write access") might suggest at a glance; the lock is opportunistic
  (triggered by the edit itself) rather than a deliberate, upfront user
  action.
- **Locks release on push, but propagate to others only on pull**: "the
  rest of the team must pull the files before the read-only protection
  is removed" — the phrase "read-only protection" strongly implies a
  real client-side filesystem enforcement (the locked file made
  read-only on a collaborator's disk), not merely a UI badge — a
  materially stronger, more automatic enforcement than the confirm-
  before-overwrite *dialog* pattern found in `Ondsel-Lens-Addon`'s
  client (which still lets the user click through), though this
  detail is stated by the vendor's marketing material and was not
  independently verified against any implementation.
- **Force-unlock exists and is audited**: "you can unlock the file by
  right-clicking," and the action "will be recorded on the project
  timeline" — the same admin-override-with-audit-trail pattern already
  confirmed independently in EasyPDM (admin lock takeover), OdooPLM
  (checkout chatter messages), and CADBase (component-history
  old-data log) — now a fifth confirmed instance of this exact
  convergent design choice across completely unrelated products and
  business models.
- **No timeout/staleness mechanism is documented anywhere found in this
  pass** — unlike GitPDM's explicit `STALE_PRESENCE_SECONDS`/heartbeat
  design or FreePDM's designed (if unimplemented)
  `ExpiresAt`/`Heartbeat` lock interface, Anchorpoint's own public
  documentation says nothing about what happens if a lock-holder goes
  offline without pushing — force-unlock appears to be the only stated
  remedy, which is a real, if unconfirmed, potential gap (a lock held
  by someone who has gone on vacation with unpushed changes would, per
  the documentation read, require a teammate to notice and manually
  force-unlock, rather than any automatic staleness detection).

## 4. File format / serialization touchpoints

- **Format-agnostic by design** — Anchorpoint's own claim is that it
  "handles binary files without prior configuration," meaning `.FCStd`
  gets no special parsing or treatment beyond being recognized as
  binary (hence lockable, hence excluded from any merge attempt). No
  FreeCAD-specific format knowledge (assembly structure, BREP caches,
  etc.) is claimed or implied anywhere in the sources read.
- **Automatic thumbnail generation** is mentioned for asset management/
  browsing, presumably format-aware for common types, but no detail on
  how (or whether) this extends to `.FCStd` specifically was found in
  this pass.

## 5. Dependencies & integration points

- **Git + Git LFS as the storage substrate**, with Anchorpoint's own
  forked `git-lfs` (per its GitHub org) and a bundled portable Git
  distribution (`git-dist`) — the company controls its own Git/LFS
  toolchain rather than depending purely on the system's installed
  git.
- **A proprietary cloud metadata service** for locking/tagging/
  centralized Git configuration — a hard external dependency, the same
  general shape as CADBase's or Ondsel-Server's cloud backends, except
  here the *locking* function specifically is what's centralized, not
  the file storage itself (file storage stays on whatever standard git
  host the team already uses).
- **Self-hosted deployment option exists** (`ap-self-hosted-templates`
  in the GitHub org) — suggesting the metadata-server dependency is not
  necessarily a hard requirement to use *some* Anchorpoint-hosted
  service specifically, though whether self-hosting includes the
  locking metadata server or only other components wasn't confirmed in
  this pass.
- **No FreeCAD-specific dependency of any kind** (§1) — integration is
  entirely at the "generic binary file in a git repo" level.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:anchorpoint` (line 1567):
category `[version-control, pdm]`, `technical_approach: [git]`
(accurate but incomplete per §3 — a custom, out-of-band metadata
service is the actual locking mechanism, not git or git-lfs locking
itself; not corrected, since `[git]` isn't wrong, just underspecified,
and this census's `technical_approach` vocabulary has no tag for
"proprietary companion service" to add instead), `scope:
multi-cad-platform` (confirmed, and per §1 probably undersells how
non-CAD-specific this product actually is — "multi-DCC-platform" would
be more accurate but isn't an existing vocabulary value), license
"proprietary" (confirmed), `first_seen: unknown`, existing note
correctly identifying the "lock, don't merge" pattern shared with
EasyPDM and the Omniverse connector's checkpoint model.

**A wording issue spotted, not corrected**: the existing note's closing
phrase, "the 'lock, don't prevent' pattern," appears to be a
transcription slip — every other node in this cluster (EasyPDM,
`freecad-omniverse-connector`) uses the phrase **"lock, don't merge"**
for this same pattern, and "lock, don't prevent" is essentially the
opposite of what Anchorpoint's own documentation describes (locking
*is* how it prevents conflicts). Flagged for a maintainer to fix
directly rather than corrected here, since this note only has web
research to go on and a one-word wording fix to another node's note is
a low-risk, easily-verified edit better made by whoever last touched
that census section.

## 7. Friction points observed firsthand

- **The out-of-band metadata-server locking design (§3) is a genuinely
  distinct fourth architecture for "how do you coordinate exclusive
  access to a binary file" in this census**, alongside EasyPDM's
  database-column lock, GitPDM's in-repo presence branch, and OdooPLM's
  database-constraint-enforced checkout table. Each keeps the lock
  state in a different place (a PDM database, the git repository
  itself, a relational database, and now a vendor's separate cloud
  service) — worth citing together as a spectrum of options for any
  future proposal, since the "where does lock state live" question
  turns out to have several genuinely different, defensible answers
  across this census, not one obvious right place.
- **The claimed client-side read-only enforcement** (§3, "read-only
  protection") — if accurate — is a meaningfully stronger deterrent
  than any confirm-before-overwrite *dialog* pattern found with source
  access in this census (Ondsel-Lens-Addon's warning is still
  override-able by clicking through; a genuinely read-only file on
  disk is harder to accidentally edit at all). Flagged as unverified
  vendor-stated behavior, but worth treating as an aspirational
  benchmark: a real filesystem-level read-only flag on a locked file,
  automatically applied and lifted on lock/unlock, would be a concrete,
  implementable step beyond what any FreeCAD-native tool with source
  access in this census currently does.
- **This note's own epistemic limitation is itself worth stating
  plainly**: everything here is a vendor's public description of its
  own product, not verified behavior. Where this note says
  "documentation states" or "claims," that qualifier is load-bearing —
  any future synthesis citing Anchorpoint's design should carry the
  same caveat forward rather than treating vendor blog-post claims as
  equivalent in reliability to this census's source-verified findings
  for the other ~20 projects.

## 8. Minimal-patch hypothesis

- **Not a source of adoptable code** (no source exists), but a real
  source of a **design pattern worth naming explicitly**: exclusive,
  automatic (not deliberately-checked-out), out-of-band locking with
  client-side read-only enforcement and an audited force-unlock escape
  hatch. Any future FreeCAD-native tool wanting the *strongest*
  possible deterrent against accidental concurrent binary-file edits
  (stronger than a warning dialog, short of OdooPLM's database
  constraint, which requires a real backend database Anchorpoint's
  target audience may not have) could look at "make the locked file
  read-only on disk automatically" as a cheap, OS-level enforcement
  layer — no server changes needed on the *reading* side, only on
  whatever already tracks lock state.
- **The "keep lock state in a separate service rather than in the
  repository itself" choice (§3, §7)** is worth flagging as a real
  design fork for this plan's eventual synthesis to weigh explicitly:
  it trades self-containment (GitPDM's repo-native presence branch
  needs no extra service) for potentially better performance/reliability
  at scale (Anchorpoint's claimed 1,000-locks-under-a-second figure) —
  neither choice is strictly better, and a shared convention proposal
  for this census's Cluster 2 question should probably accommodate
  both rather than mandate one.
