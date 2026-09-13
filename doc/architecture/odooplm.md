# Architecture notes — OdooPLM (`OmniaGit/odooplm`)

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read: `README.md`, `CLAUDE.md` (the repository's own detailed
architecture/contributor guide), `plm/models/plm_mixin.py` (the shared
revision base class, read in full for its header/state-machine
section), `plm/models/plm_checkout.py` (read in full). No git commands
were run for this pass.

This is, alongside CADBase, one of the two most mature, most directly
production-grade finds in this entire census — but arrived at from a
completely different direction: rather than a purpose-built FreeCAD-
native PDM, OdooPLM is a **36-module extension of an existing, general-
purpose ERP** (Odoo), maintained by a real company (OmniaSolutions)
with a paying/commercial CAD-client product alongside its open-source
server side. `doc/ecosystem/graph.yaml`'s own existing note already
flags something important this read corroborates directly: this
project is older, more active, and better-starred than nearly every
FreeCAD-native PDM attempt in the census, yet is essentially invisible
to the FreeCAD community's own discussions of "the PDM problem."

## 1. Repo & basic facts

- **Scale**: 36 Odoo modules (per `README.md`'s own count) on top of
  one foundation module (`plm`), covering document management/3D
  viewing, BOM variants (engineering/spare/date-effective/kit),
  automated weight/breakage/cut-part tracking, manufacturing
  integration, and — genuinely unusual for this census — an **AI/MCP
  integration** (`plm_mcp`, `plm_mcp_bot`, `plm_mcp_ecr`,
  `plm_mcp_odoo_ai`): a Model Context Protocol server exposing "14
  tools that let an AI assistant ask the engineering database real
  questions — where a part is used, what changed between revisions,
  what is missing a drawing, what a change would affect," reachable
  both via bearer-token API and, with no API key or LLM cost at all,
  by typing `/plm` in a chat with OdooBot. No other project in this
  census has anything comparable — this is the only PDM/PLM system
  found here that treats "let an AI agent query the engineering
  database" as a first-class, shipped product feature rather than a
  hypothetical.
- **Real commercial backing**: developed and maintained by
  OmniaSolutions, with a **proprietary desktop CAD-client connector**
  (SourceForge, `openerpplm` — the project's own naming reveals its
  lineage back to OpenERP, Odoo's predecessor name) sold alongside the
  open-source server modules — a genuine open-core commercial model,
  the only one of its kind in this census (contrast CADBase, which is
  cloud-SaaS but fully open-source across the stack; contrast Taack
  PLM, whose server is simply undownloaded/unexamined rather than
  deliberately closed).
- **Multi-CAD-tool support at commercial-grade depth**: the desktop
  client integrates with SolidWorks, SolidEdge, Autodesk Inventor,
  AutoCAD/DraftSight (2D), ThinkDesign (all listed as "Full
  integration: checkout, upload, BOM sync"), and FreeCAD ("Open-source
  integration") — a broader multi-tool span than CADBase's two clients
  (FreeCAD, Blender), covering the actual major commercial MCAD tools a
  small manufacturer would realistically be using alongside FreeCAD.
- **Licensing is deliberately split and CI-enforced**: per `CLAUDE.md`,
  the foundation `plm` module is **LGPL-3-or-later** while every other
  of the 35 feature modules is **AGPL-3-or-later** — confirmed directly
  (`plm/__manifest__.py`: `"license": "LGPL-3"`) — with a dedicated
  `check-licensing` pre-commit hook that fails any commit where a
  module's manifest license disagrees with `LICENSING.md`'s
  authoritative table. `doc/ecosystem/graph.yaml` currently records
  `license: unknown`; corrected in this pass to reflect the actual
  split (see §6).
- **Candid, unusually direct engineering documentation**: `CLAUDE.md`
  states plainly that black/isort/prettier/eslint/pylint-odoo "were
  removed" from the pre-commit config because "the pinned 2020 versions
  no longer build on Python 3.12, and reinstating them would mean a
  repository-wide reformat (128 files for black, 151 for isort, 167 XML
  files for prettier)," and that flake8 runs "report only... there is a
  backlog of ~1300 findings." This kind of frank, quantified technical-
  debt disclosure is rare and worth noting as a positive transparency
  signal, distinct from the actual architecture question.

## 2. Identity/versioning model

**The most complete revision model found anywhere in this census,
including genuine branch support — something no other project here
attempts.**

- Every versioned object (products, documents/`ir.attachment`, BOMs)
  inherits a shared abstract base, `revision.plm.mixin`
  (`plm/models/plm_mixin.py`'s `RevisionBaseMixin`), giving identity as
  an `engineering_code` + `engineering_revision` pair, uniqueness
  enforced by an `@api.constrains` check across the pair
  (`_check_engineering_constraints`) — the same "human-facing code +
  incrementing revision number" shape as EasyPDM's own model, but
  applied uniformly across *every* versioned entity type in the system
  via a shared mixin, rather than being reimplemented per-object-type.
- **Revision letters are generated the same way EasyPDM's are** — a
  recursive base-26 `convert_to_letter()` helper turning a numeric
  revision index into A, B, C… Z, AA, AB… — independently confirming
  (a third time now, after EasyPDM and Ondsel-Server/CADBase's
  numeric-revision-plus-display-letter pattern) that "store a number,
  display a letter" is a settled, convergent design choice across this
  whole ecosystem regardless of language or backend.
- **Genuine branch/sub-revision support**: alongside the main
  `engineering_revision`/`engineering_revision_letter` pair, the mixin
  also carries `engineering_branch_revision`,
  `engineering_branch_revision_letter`, `engineering_branch_parent_id`,
  and `engineering_sub_revision_letter` — a real secondary revision
  axis for tracking a branch off a parent revision. No other project
  in this entire census has anything resembling this; every other
  system found here (EasyPDM, Ondsel-Server, CADBase, FreePDM's design)
  is strictly linear. Not read in enough depth in this pass to
  characterize *how* branches are created or merged (that logic likely
  lives in `ir_attachment.py`'s 4,802 lines, not read in full) — flagged
  as a strong candidate for a deeper follow-up read if branch/merge
  semantics for CAD data become a focus of this plan's synthesis.
- **A five-state lifecycle machine** — `draft → confirmed → released ↔
  undermodify → obsoleted` (`START_STATUS`/`CONFIRMED_STATUS`/
  `RELEASED_STATUS`/`UNDER_MODIFY_STATUS`/`OBSOLATED_STATUS`) — with an
  explicit `PLM_NO_WRITE_STATE` list (`confirmed`, `released`,
  `undermodify`, `obsoleted`) gating write access. This is
  structurally the same state machine as EasyPDM's
  `w_pracy → sprawdzany → wydany → anulowana`, right down to the
  underlying rule ("outside draft/`w_pracy`, editing is locked") —
  independently reinvented in an entirely different language, backend,
  and business context (French/Italian Odoo-ecosystem PLM vs. a Polish
  solo-developer FreeCAD workbench). This is now a well-confirmed,
  cross-project convergent pattern worth stating plainly in any future
  synthesis: **"a small, closed set of lifecycle states, with editing
  locked outside the initial/draft state" is closer to a settled
  industry norm than a design choice any one project invented.**

## 3. Conflict/concurrency strategy

**Real, database-enforced pessimistic locking — the strongest hard
guarantee against double-checkout found anywhere in this census,
stronger even than EasyPDM's application-level compare-and-swap.**

- `plm.checkout` (`plm_checkout.py`, read in full) is a dedicated model
  representing "Document that are locked from someone" — a row per
  checked-out document, linking `documentid` (the `ir.attachment`) to
  `userid` (who holds it) plus `hostname`/`hostpws` (which machine and
  working-copy path, letting the system know where the checked-out
  file physically lives for the desktop CAD client).
- **The uniqueness guarantee is enforced at the database level, not
  just in application logic**: `_documentid = models.Constraint("unique
  (documentid)", "The documentid must be unique !")` — a real SQL
  `UNIQUE` constraint on the checkout table's `documentid` column. Two
  simultaneous checkout attempts on the same document cannot both
  succeed even under a race, because the second `INSERT` will violate
  the constraint at the database engine level — a materially stronger
  guarantee than EasyPDM's `UPDATE ... WHERE (owner_locked = false OR
  ...)` conditional-write pattern (itself already a reasonable
  compare-and-swap, but enforced by application-issued SQL against a
  boolean flag column rather than by a schema-level uniqueness
  constraint on a dedicated lock table).
- **Checkout and check-in are the model's own `create()`/`unlink()`
  overrides**, not separate action methods — creating a `plm.checkout`
  row *is* the checkout operation (it flips `engineering_writable =
  True` on the target document, raising `UserError` if that write
  fails) and deleting it *is* the check-in (`unlink()` flips
  `engineering_writable` back to `False`, and explicitly **refuses to
  check in if the document `has_error`** — "Unable to check-in due to
  an error on saving document... with error {getLastError()}" — a
  real, concrete safety rail against silently checking in a document
  the system knows failed to save correctly).
- **A related, cascading relation-adjustment mechanism**:
  `_adjustRelations()` stamps every child document relation
  (`ir.attachment.relation`) with the checking-out user's id on
  checkout, and clears it back to `False` on check-in — i.e. checking
  out a top-level assembly document also marks its component relations
  as "owned" by that session, presumably so downstream logic (BOM
  comparison, sync) knows which relations are mid-edit. Not traced
  further into `ir_attachment_relations.py` in this pass.
- **Full audit trail via Odoo's own `mail.thread` chatter**: every
  checkout and check-in posts a message directly onto the document's
  own discussion thread (`docBrws.message_post(body=_(f'Checked-Out ID
  {newCheckoutBrws.id}'))`, and `doc_id.message_post(body=_("Checked-In"))`
  on unlink) — reusing Odoo's own built-in social/activity-log feature
  rather than building a separate audit table, a clean example of
  getting a real audit trail "for free" by building on a host
  platform's existing primitives, the same general instinct (reuse
  what the platform already gives you) already praised in `taack-plm`'s
  and `pr-26306`'s architecture notes for FreeCAD's own native `Uid`
  property.
- **Net assessment**: this is the most complete concurrency story in
  the whole census — a real pessimistic lock, database-enforced
  uniqueness, an explicit safety rail against checking in a broken
  save, cascading relation-ownership tracking, and a built-in audit
  trail, all in ~135 lines because it leans heavily on Odoo's own ORM
  constraint system and chatter infrastructure rather than building
  any of that from scratch.

## 4. File format / serialization touchpoints

- **`ir.attachment` (Odoo's own generic file-attachment model) is
  extended, not replaced**, to become the engineering-document model
  (4,802 lines in `ir_attachment.py`, not read in full in this pass) —
  another instance of building PDM behavior directly onto a host
  platform's existing primitive rather than introducing a parallel
  document-storage concept.
- **Real, substantial format-conversion infrastructure**: the
  `plm_automated_convertion` module does batch STEP→3MF, STEP→STL, and
  STEP→PNG-preview conversion "preserving assembly structure and
  component names," and the published Docker images explicitly ship a
  "full" variant (~4.4 GB) bundling this conversion stack versus a
  "slim" variant (~2.5 GB) without it — a deployment-time size/
  capability tradeoff not seen articulated this explicitly in any other
  project's own documentation in this census.
- **A genuinely capable in-browser 3D/2D viewer** (`plm_web_3d`,
  Three.js-based): 3MF/STEP/GLTF/STL/OBJ/DXF/SVG support, a section
  plane with stencil cap, a snap-enabled measurement tool, per-part
  color/transparency with persistence, and a markup system integrated
  with Odoo's chatter — comparable in ambition to CADBase's
  `three-libs` web viewer, but with more interaction features
  documented (section planes, measurement) than confirmed for CADBase's
  own viewer in this pass.
- **CAD client transport is REST + XML-RPC**, not GraphQL or Protobuf
  — `plm/controllers/main.py` exposes plain REST endpoints
  (`/plm_document_upload/login`, `/upload`, `/upload_pdf`,
  `/isalive`) per `CLAUDE.md`, alongside Odoo's standard XML-RPC API
  — a fourth distinct wire-format choice across this census (after
  EasyPDM/CADBase/Ondsel-Server's REST+JSON/GraphQL and
  `taack-plm-freecad`'s Protobuf).

## 5. Dependencies & integration points

- **Built entirely on Odoo's own framework** (ORM, `mail.thread`
  chatter, ACL/security groups, XML-RPC) — the PLM-specific code is
  almost entirely business logic layered on Odoo's existing platform
  primitives, which is precisely why a ~135-line `plm_checkout.py` can
  deliver a stronger concurrency guarantee than more code-heavy,
  built-from-scratch implementations elsewhere in this census.
- **A real, weekly-rebuilt Docker distribution pipeline**
  (`OmniaGit/DockerOdooPLM`, referenced from this repo's README):
  images published to both GHCR and Docker Hub, "full"/"slim" variants
  crossed with a "-demo" tag that boots pre-populated with a real
  sample product (LSU-100, 12 parts over three BOM levels, 29 STEP/
  3MF/DXF/PDF documents, an in-progress engineering workflow already
  exercised across the demo data: "released revisions, a superseded
  part, one under modification, an open change order, a checked out
  sheet"). This is the most complete, most realistic demo/onboarding
  environment found in this entire census — a prospective user or
  future contributor can see the whole revision/checkout/BOM lifecycle
  already mid-flight on first login, not just an empty schema.
- **External Python dependencies** (`aaa_requirements.txt`, per
  `CLAUDE.md`): `ezdxf`, `matplotlib`, `cadquery`, `numpy-stl`,
  `base64io`, `to-3mf` — a real, if unglamorous, list of CAD/geometry-
  adjacent libraries backing the conversion and viewer features.
- **Git submodules for frontend viewer code**
  (`plm_web_3d/static/src/js/lib/`: Three.js and a DXF viewer) — a
  practical detail `CLAUDE.md` calls out explicitly ("may appear dirty
  in git status — this is expected") as a known, accepted rough edge.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:odooplm` (line 1097): category
`[plm, pdm, bom]` (all three confirmed to fit cleanly by this read —
arguably the best category fit of any node in this census, no caveat
needed unlike OpenPartsLibrary/BCF-Plugin-FreeCAD/`pr-26306`),
`technical_approach: [sql-database]` (confirmed — Odoo runs on
PostgreSQL), `scope: multi-cad-platform` (confirmed and, per §1, even
broader than the tag's other holders — SolidWorks/SolidEdge/Inventor/
AutoCAD/ThinkDesign/FreeCAD), license `unknown`, `first_seen:
2017-10-07`, and an existing note already correctly flagging the core
finding this read corroborates: "152 stars, highest of any
platform-level PLM in the census... absent from FEP-0011's Motivation
'known initiatives' list and from every FreeCAD-org issue thread
researched."

**Correction applied to `graph.yaml` in this pass** (see below):
`license: unknown` → a mixed-license note, matching the pattern already
used for CADBase's node — no single SPDX value fits, since the `plm`
foundation module is LGPL-3-or-later while all 35 other modules are
AGPL-3-or-later, confirmed directly from `plm/__manifest__.py` and
`CLAUDE.md`'s own description of the CI-enforced licensing split.

No correction needed to `first_seen: 2017-10-07` — consistent with the
project's own SourceForge client name, `openerpplm`, pointing back to
Odoo's OpenERP-era predecessor name, which OpenERP itself was rebranded
away from around 2014-2015 — a 2017 first-seen date for this
particular repository/rebrand is plausible and not contradicted by
anything read in this pass.

## 7. Friction points observed firsthand

- **This is now the single clearest, most concrete case in this entire
  census of `doc/ecosystem/graph.yaml`'s broader "uncoordinated
  ecosystem" thesis** — a bigger, older, more active, better-
  distributed, commercially-backed PLM system than nearly every
  FreeCAD-native attempt studied in this series, invisible to the very
  community discussions (FEP-0011, FreeCAD-org issue threads) that keep
  reinventing pieces of what this project already ships. This is
  stronger, first-hand-confirmed evidence for exactly the kind of
  cross-ecosystem-awareness gap the technical-convergence plan and its
  predecessor documents were built to surface.
- **The independently-convergent revision-letter and lifecycle-state-
  machine patterns (§2)**, now confirmed a third time across three
  unrelated codebases/languages/business contexts (EasyPDM, CADBase/
  Ondsel-Server, and now OdooPLM), should be treated as settled,
  cross-ecosystem best practice in any future synthesis — not merely a
  recurring coincidence worth footnoting, but a genuine signal about
  what any future shared "version record" convention proposal should
  assume as a baseline.
- **The database-enforced unique-checkout constraint (§3)** is the
  strongest concurrency-safety mechanism found anywhere in this
  census, and it achieves that strength specifically *because* it
  leans on the host platform's own database constraint system rather
  than trying to re-implement compare-and-swap logic in application
  code — a lesson directly applicable to any future FreeCAD-native PDM
  tool built on a real SQL backend (EasyPDM, FreePDM, a future
  Ondsel-Server successor): prefer a schema-level uniqueness
  constraint on a dedicated lock/checkout table over an
  application-level conditional `UPDATE`, wherever the backend
  supports it.
- **The MCP/AI-agent integration (§1)** is a genuinely new category of
  "collaboration" this census hadn't encountered before this project:
  not human-to-human file sharing or locking, but structured,
  governed (bearer-token, per-user) machine access to the engineering
  database for question-answering. Worth flagging as a possible future
  direction for this whole ecosystem regardless of what happens with
  file-versioning/locking conventions specifically — a shared "ask
  questions about the engineering history" API surface is a
  differently-shaped, and possibly easier, convergence target than a
  shared file-locking protocol.

## 8. Minimal-patch hypothesis

- **Not really a source of a "minimal patch"** in the plan's own
  sense — OdooPLM isn't a small, portable idea, it's a large, mature,
  already-coherent system whose individual mechanisms (the checkout
  constraint, the revision mixin) are valuable as *reference designs*
  to imitate rather than as code to lift out and reuse (Odoo's ORM/
  constraint system isn't something a FreeCAD-native tool could adopt
  piecemeal).
- **The clearest, most actionable "steal this design" recommendation
  in this whole census**: any future SQL-backed FreeCAD-native PDM
  effort (a revived FreePDM, a hypothetical EasyPDM v2) should model
  its checkout mechanism directly on `plm.checkout`'s shape — a
  dedicated lock table with a database-level `UNIQUE` constraint on the
  locked resource, rather than a boolean flag on the resource's own
  row guarded only by application logic. Cost to adopt this pattern
  elsewhere: **small** — it's a schema and a small model, not a new
  architecture; the value is specifically in *where* the guarantee
  lives (the database engine) rather than in any FreeCAD-specific code.
- **A genuinely different kind of recommendation for this plan's
  synthesis than anything proposed so far**: given how invisible this
  project is to the FreeCAD-native PDM discussions this whole census
  otherwise centers on, the single highest-leverage "patch" here may
  not be a code change at all — it may be **making this project's
  existence and design known** to the FreeCAD-native PDM community
  (FEP-0011's authors, the Ondsel Onward Fund contributors, FreePDM's
  team) before any of them invests further effort re-solving problems
  (revision letters, lifecycle states, checkout locking) this project
  already has working, battle-tested, commercially-deployed answers
  for.
