# From Lens to a Standards-Based FreeCAD PDM: A Staged Decision Tree

Purpose: this document assumes the optimistic outcome of commenting on
[Ondsel-Server issue #48](https://github.com/FreeCAD/Ondsel-Server/issues/48) — the maintainers say "build it, use
the standards you're pointing at." It answers the follow-up question that raises immediately: **can this actually be
built, or does it require locking in too many undecided things too early?** The answer worked out below is: it can be
built, but only as a sequence of small, separately-shippable decisions, each one gated on real usage feedback from
the one before it — not as a single "adopt AAS" rewrite. This document maps that sequence as an explicit decision
tree, stage by stage, so the branch count is visible rather than hidden inside a single vague roadmap item.

This builds directly on [`ondsel_48_research.md`](ondsel_48_research.md) (what Lens is, what issue #48 is actually
asking for, which standards answer which gap) and the earlier cross-project research in
[`possible_agile_manufacturing.md`](../co-edit-stream/doc/possible_agile_manufacturing.md) and
[`possible_data_architecture.md`](../co-edit-stream/doc/possible_data_architecture.md). It does not repeat the
standards explanations from that document — it assumes them and asks the implementation question instead.

---

## How to read this document

Each **Stage** is one iteration of the cycle: *implement a change → derive/observe a benefit → make a decision
informed by that benefit → implement the next change.* A stage doesn't conclude with one answer — it concludes with
a **branch set**: the small number of genuinely different directions the next stage could take, each with different
cost, benefit, and risk. The tree is deliberately not collapsed to "the one true path" up front, because the
user's own framing is correct: Stage 1 creates a handful of branches, and each of Stage 2's options interacts
differently with each of Stage 1's, so the honest shape of "how do we get from here to a full PDM" is a tree, not a
line. A recommended path is given at the end — but it's a *pruning* of this tree based on today's evidence, not a
claim that the other branches are wrong.

Two disciplines make this tractable instead of combinatorially unmanageable:

1. **Every stage must ship something Lens's current users can use, on its own**, even if the project stopped
   there. This is what keeps the tree from becoming an all-or-nothing bet — a wrong branch choice at Stage 2 is a
   wasted few weeks, not a wasted rewrite, because Stage 1's shipped value doesn't depend on Stage 2 having
   happened.
2. **Prefer branches that keep later options open over branches that look more "complete" now.** Concretely: an
   adapter/shim that stores data AAS-*shaped* inside the existing MongoDB collections is preferred at nearly every
   decision point over immediately standing up standards-native infrastructure (a real AAS server, a full BCF
   service), because it's far cheaper to later export that data into a conformant AAS/BCF store than to have
   over-committed to one early and discover it doesn't fit real usage.

---

## Baseline: what Lens actually has today (grounding for every stage below)

Confirmed directly from the cloned `Ondsel-Server` repo, not assumed:

- **Workspace** (`backend/src/services/workspaces/workspaces.schema.js`) — a permissioned container
  (`groupsOrUsers` with read/write, license, `refName`/`refNameHash` for case-insensitive URL slugs) holding one
  root **Directory**.
- **Directory** (`directories/directories.schema.js`) — a tree of directories/files, no identity beyond
  `parentDirectory` nesting.
- **File** and **Model** (`models/models.schema.js`) — a `Model` is explicitly commented in the code as *"a
  snapshot in time for a specific combination of: 1. File Version, 2. SharedModel (Link), 3. User Parameters."*
  This is the load-bearing fact for everything below: **Lens already has a notion of "a file with parameters
  applied," it just isn't given a persistent identity independent of the file.**
- **SharedModel** (`shared-models/shared-models.schema.js`) — explicitly commented as *"a 'link', a reference...
  not a model by itself."* Carries granular permission flags (`canViewModel`, `canExportSTEP`, etc.),
  `versionFollowing` (Active vs. pinned-to-a-specific-version — this is a real, already-shipped versioning
  policy concept), and — notably — **`messages` and `messagesParticipants`**: Lens already has a flat, unanchored
  comment thread attached to a share link. This matters a lot for Stage 5 below: the "meta-document" gap in issue
  #48 is not a blank slate, it's an existing primitive that needs extending, not inventing.
- **Publisher** (`publisher/publisher.schema.js`) — a separate mechanism for release-cadence-based public file
  publishing (`PublishedReleaseCadenceType`, `releaseDate`), already a first cut at "not every save is a release."
- **Curation** (`curation.schema.js`) — a cross-cutting, shared sub-schema used by users/organizations/workspaces
  for search indexing and nav references. The pattern (a small shared schema, imported into several services'
  own schemas, kept eventually-consistent via `*.distrib.js` "summary" documents) is the house style for adding a
  cross-cutting concern without a big-bang refactor — every stage below should reuse this pattern rather than
  inventing a new one.
- **Service structure** (documented in `docs/technical.md`): every new concept costs a predictable, bounded unit of
  work in this codebase — a `*.class.js`, `*.schema.js`, `*.shared.js`, optionally `*.subdocs.schema.js`,
  `*.distrib.js`, `*.curation.js`. This is genuinely good news for feasibility: the codebase already has a
  well-worn groove for "add a new first-class collection," which every stage below uses.
- **What's confirmed absent**: no `items` service, no cross-file/cross-version persistent part identity, no
  assembly/BoM link structure between items, no type/instance (configurable-part) concept, no geometry-anchored
  annotation (only the flat `messages` list above).

---

## Stage 0 — The gating decision: how does an "item" get an identity that outlives a file version?

**The question.** Everything downstream — documents attached to an item, assemblies linking items, an item having a
type and instances — needs items to have IDs that are stable across file re-uploads, exports, and (per issue #48's
own parameterization discussion) across configuration variants. Today, nothing in Lens has such an ID; the closest
thing is a File's or Model's MongoDB `ObjectId`, which is tied to one upload event, not a persistent "this part"
concept. This is Stage 0, not Stage 1, because every later stage's schema depends on the answer.

**Branch A — Internal-only ID (new `items` collection, plain Mongo ObjectId).**
Cheapest possible answer: an `Item` is just a new collection; its ID is whatever MongoDB assigns. No external
identifier scheme adopted at all.
- *Cost:* near-zero — a day or two of scaffolding following the existing service pattern.
- *Benefit:* unblocks every later stage immediately.
- *Risk:* buys none of the interoperability the standards research promised. An item's ID means nothing outside
  this one Lens instance — no path to referencing a supplier's part hosted on a different Lens/PDM instance, which
  is precisely the federation goal the earlier `possible_agile_manufacturing.md` research flagged as the
  actually-hard, actually-valuable part of this whole space.
- *Social:* invisible to users and to the FreeCAD core team; doesn't require anyone outside the immediate PR to
  agree on anything. Easiest to get merged.

**Branch B — Adopt AAS `globalAssetId` (a URI) as the canonical item identifier.**
Every item gets a real, dereferenceable-in-principle URI as its ID, per the AAS metamodel's identification rules
([IDTA 01001-3-0 §4.3](https://industrialdigitaltwin.org/wp-content/uploads/2023/06/IDTA-01001-3-0_SpecificationAssetAdministrationShell_Part1_Metamodel.pdf#page=27)).
- *Cost:* moderate — needs a URI-minting scheme (likely `https://<lens-instance>/asset/<uuid>` or similar), and a
  decision about whether that URI needs to actually resolve to something (the spec explicitly says it doesn't have
  to — "even though the HTTP scheme is used for the identifier, please be aware that these identifiers are logical
  ones" — but a self-hosted Lens instance dying would silently orphan the identifier space unless this is
  anticipated).
- *Benefit:* real standards conformance from day one; an exported item is legible to any AAS-aware tool without
  translation.
- *Risk:* couples the identifier to a hostname/deployment decision (which Lens instance minted it) at exactly the
  moment federation between independent Lens/FreeCAD-PDM instances is a stated long-term goal (issue #47) — get
  the URI scheme wrong now and it's expensive to change later, because external references would already exist.
- *Social:* requires the Lens/FreeCAD-org maintainers to buy into "we're an AAS-conformant system" as an identity,
  not just "we borrowed some ideas" — a bigger ask of the PR reviewer than Branch A, and arguably premature before
  anyone's used the item concept at all.

**Branch C — Adopt IEC 81346-style hierarchical reference designators.**
Items get identifiers like `=MACHINE.ASSY1.PART3`-style reference designations instead of opaque IDs or URIs, per
the earlier `possible_agile_manufacturing.md` recommendation.
- *Cost:* higher than A or B — reference designators are inherently hierarchical/relative (a part's designator
  depends on its assembly context), which is a real modeling task, not just a field-type choice; also less natural
  fit for Lens's existing flat/workspace-based organization than for the factory/plant context IEC 81346 was
  designed for.
- *Benefit:* strong cross-discipline interoperability (electrical, mechanical, documentation all referring to "the
  same" designator) — but that benefit is mostly relevant once Lens is exchanging data with non-CAD tools, which is
  far downstream of anything in scope here.
- *Verdict:* premature for Lens specifically; worth revisiting only if/when Lens starts talking to EDA or PLM
  tooling that already uses 81346 designators (e.g., EPLAN exports) — noted here so it isn't lost, not recommended
  as a near-term branch.

**Branch D — Internal ID now, AAS-shaped `globalAssetId` as a derived/exported field later (the adapter approach).**
Keep Branch A's cheap internal ID as the actual database key; add a computed, exposed `globalAssetId`-shaped URI
field whenever/if an item is exported to an AAS package (`.aasx`) or exposed via a future AAS-compatible API,
without it being the primary key anywhere internally.
- *Cost:* barely more than Branch A.
- *Benefit:* defers the hostname/URI-scheme commitment from Branch B until there's an actual external consumer that
  needs it (e.g., the first real AASX export, or the first real federation partner), while still making that export
  possible without a schema migration — the internal ID and the exported ID are different concerns from day one, so
  neither blocks the other later.
- *Risk:* minimal — the only real risk is forgetting to design the export mapping carefully when it's finally
  needed, but that's a much smaller, later, better-informed decision than committing to a URI scheme before any
  item exists.

**Recommendation for Stage 0: Branch D.** It's the only option that doesn't force a standards-conformance
commitment before there's a single real item in the system to learn from, while still leaving both later paths
(deeper AAS conformance, or staying lightweight forever) genuinely open.

---

## Stage 1 — Add the Item layer itself (branches multiply against Stage 0's choice)

**The question.** Given an identity scheme from Stage 0, what does an `Item` actually look like as a Lens data
model, and how does it relate to the existing `File`/`Model`/`Directory` hierarchy?

**Branch 1 — New top-level `items` collection, VDI 2770-shaped, referencing existing `File`/`Model` docs.**
An `Item` document holds `{ id, name, documents: [{ documentId, classification, versions: [{ modelId or fileId,
status, releasedAt }] }] }`, following the [Document/DocumentVersion split confirmed in IDTA 02004
§2.1](https://industrialdigitaltwin.org/wp-content/uploads/2025/07/IDTA-02004-2-0_Submodel_Handover-Documentation.pdf#page=5).
Each `DocumentVersion` points at an existing `Model` (which already points at a `File`) rather than duplicating file
storage.
- *Cost:* one new service, following the house pattern (`items.class.js`/`.schema.js`/`.distrib.js`/`.curation.js`)
  — a bounded, estimable unit of work, same shape as any other service in this codebase.
- *Benefit:* directly closes the exact gap issue #48 names; every existing `Model`/`File` keeps working unchanged,
  since `Item` only *references* them.
- *Migration:* additive only — no existing collection needs a schema change, so this can ship without touching
  live workspaces at all (an `Item` is opt-in, created going forward; a backfill script to retroactively "item-ify"
  old files can be a separate, later, non-blocking task).

**Branch 2 — Item as a special kind of `Directory` (a "part folder"), per pierreporte's own suggestion in the
issue** (*"storing an item as a sub-directory in a workspace"*).
- *Cost:* lower schema surface (reuse `Directory`) but higher conceptual risk: `Directory` today is a generic
  filesystem-like container with no notion of "this folder *is* a part," so this branch means either overloading
  `Directory` with new optional fields (messy) or introducing a discriminator, which starts to look like Branch 1's
  new collection anyway, just awkwardly nested inside the existing one.
- *Benefit:* matches users' existing mental model (a part's stuff lives in "its folder") with no new top-level
  concept in the UI's navigation.
- *Risk:* directories are already reused for organizational structure unrelated to parts (arbitrary folders of
  arbitrary files); conflating "a folder" with "a part's identity" risks exactly the kind of schema regret the
  earlier research (Krahn et al., "A Manifesto for Semantic Model Differencing" — cited in
  `optimistic_locking_research.md`) warns about: a generic structural concept (a folder) standing in for a
  domain-meaningful one (a part) breaks the moment someone's folder-per-part convention isn't followed.

**Branch 3 — Item as an AAS submodel served by a companion Eclipse BaSyx instance; Mongo only caches a summary.**
- *Cost:* substantially higher — a new service to run and operate (BaSyx), a sync/cache-invalidation layer between
  it and Mongo, new deployment surface in `docker-compose.yml`.
- *Benefit:* genuine, immediate AAS-server conformance — any AAS-aware tool could query Lens's items directly.
- *Risk:* this is exactly the "stand up standards-native infrastructure before anyone's validated the item model
  against real usage" trap flagged in the reading-guidance above. Nothing yet has confirmed that Lens's users even
  want item-level granularity in their daily workflow — building a second production service to serve a concept
  that might get reshaped after the first month of real use is the highest-cost, most premature branch available at
  this stage.

**Recommendation for Stage 1: Branch 1.** It's the direct, standards-informed answer to issue #48's literal request,
fits the codebase's existing patterns exactly, requires no migration of anything already in production, and — via
Stage 0's Branch D — still keeps a path to Branch 3's AAS-server conformance open for later without having
foreclosed it.

*(Branch count so far: Stage 0 had 4 options, effectively pruned to 1 (D) as the load-bearing recommendation, but the
document keeps A/B/C alive as documented alternates; Stage 1 has 3 real options against that choice. From here on,
"branches so far" is tracked per-stage rather than multiplied out in full, since most of Stage 0's non-recommended
branches don't need independent Stage 1 analysis — they'd each shift Stage 1's cost estimates without changing its
qualitative options.)*

---

## Stage 2 — Assembly / Bill-of-Material links between items

**The question.** Issue #48's own definition — *"model: a design that defines geometry... link: a reference to (the
geometry of) a model... assembly: a collection of parts or assemblies"* — needs a concrete mechanism once Items
exist (Stage 1). But FreeCAD documents **already** encode assembly structure internally (via `App::Link` and
FreeCAD's own Assembly workbench object graph) — so this stage's real question is not "how do we model a BoM," it's
**"do we duplicate FreeCAD's internal assembly graph in Lens's database, or derive Lens's BoM view from it?"** This
is a genuine fork with a real technical hazard on one side.

**Branch A — Lightweight, hand-declared links: an `Item` gets an array of references to child `Item`s** (mirroring
the existing `groupsOrUsers`/summary-array pattern already used throughout the codebase, e.g. in
`workspaces.schema.js`), independent of what any specific FCStd file's internal link graph says.
- *Cost:* low — another array field plus a distrib-style summary sync, same pattern as everything else in this
  codebase.
- *Benefit:* fast to ship, works even for items whose "parts" aren't literally sub-objects in one FreeCAD file (e.g.
  a purchased fastener referenced by a drawing, with no FreeCAD geometry of its own).
- *Risk — the real one:* this creates **two sources of truth for assembly structure**: whatever a user declares in
  Lens's `Item` links, versus whatever the actual FreeCAD file's `App::Link` graph says. These *will* drift —
  someone edits the FreeCAD assembly, forgets (or has no obvious way) to update the Lens-side link declaration, and
  now Lens's PDM view of the assembly lies about what the file actually contains. This is precisely the "syntactic
  vs. semantic" conflict-taxonomy problem surveyed in `optimistic_locking_research.md` §2 (Hepworth's dissertation) —
  except here it's not even a *merge* conflict, it's a standing, silent divergence between two independently-edited
  representations of the same fact.

**Branch B — Derive the BoM automatically from FreeCAD's own internal link graph, via FC-Worker, on every
committed version.**
FC-Worker (which already parses `.FCStd` files to generate previews/exports) is extended to also walk the
document's `App::Link`/Assembly object graph and emit a structured BoM, which Lens stores as a **read-only, derived**
property of the `Model`/`Item`, refreshed each time a new version is processed.
- *Cost:* meaningfully higher than Branch A — requires real FreeCAD-API work inside FC-Worker (this is exactly the
  kind of schema-mapping problem `possible_freecad_collaboration.md` flagged as the weak link in the *JupyterCAD*
  bridge, issues #22/#26/#34/#35 — reading FreeCAD's own object model reliably is a known-hard, known-worth-doing
  problem, not a trivial extension).
- *Benefit:* single source of truth — the BoM Lens shows is always exactly what the file contains, by construction,
  eliminating Branch A's drift risk entirely.
- *Risk:* doesn't handle non-FreeCAD-native BoM entries (a purchased part with no FreeCAD geometry) without a
  fallback — likely needs a hybrid anyway (see Branch C).
- *Note:* this branch is a direct, concrete instance of the FreeCAD-adapter pattern already recommended in
  `possible_freecad_collaboration.md` §4: FreeCAD becomes "a submodel/module producer," and the adapter (here,
  FC-Worker) is the actual integration point, not a from-scratch design.

**Branch C — Hybrid: FC-Worker-derived links are authoritative wherever the geometry exists; Item-level manual
links are the explicit fallback/extension for non-geometric BoM entries** (purchased parts, referenced
specifications, etc.), with the schema clearly tagging which kind each link is.
- *Cost:* sum of A and B's costs, but not their risks compounding — the "two sources of truth" problem from Branch A
  disappears specifically because the schema makes explicit which links are derived-and-authoritative versus
  manually-declared-and-supplementary, rather than letting both silently claim the same kind of fact.
- *Benefit:* this is the actually-correct long-term shape — it matches how real BoMs work (some line items are "this
  sub-assembly, defined right here in the CAD file," others are "buy this fastener from McMaster-Carr, no CAD
  needed").

**Recommendation for Stage 2: start with Branch A (ship the lightweight version fast, on top of Stage 1's Item
layer), but treat it explicitly as a placeholder for Branch C, not as Branch B's competitor.** Branch A alone
validates that users want assembly-level item links at all before FC-Worker engineering effort is spent; if usage
confirms it, migrate the FreeCAD-native portion of it to Branch B/C's derived mechanism. This is the stage where the
"ship something useful at every step" discipline matters most, because Branch B/C's FC-Worker work is genuinely
substantial and shouldn't be front-loaded before Stage 1's Item concept has even been validated with users.

*(Branches so far: Stage 1's recommended path × Stage 2's 3 options = 3 live branches at this depth, before
counting Stage 0's non-recommended alternates.)*

---

## Stage 3 — Decoupling "document version" from "every file save"

**The question.** Issue #48 itself states the PDM requirement precisely: *"It is not required to track intermediate
changes, so you have control over when to release a new version."* Lens today has two partial answers already —
`SharedModel.versionFollowing` (Active vs. pinned) and the separate `Publisher` service's `releaseCadence` — but
neither is "an Item's Document has an explicit, user-triggered Release action that promotes one specific committed
`Model` snapshot to a new `DocumentVersion`, distinct from every other intermediate save."

**Branch A — Manual release action**: a user explicitly marks a `Model` snapshot as "Release as vX.Y" for a given
Item/Document; only released versions appear in the Item's `DocumentVersion` history; unreleased saves remain
ordinary `Model` snapshots exactly as today.
- *Cost:* low-to-moderate — mostly a new mutation/endpoint plus schema field (`releasedVersionOf`, `status`), and
  UI to trigger it. Directly parallels `Publisher`'s existing release concept, so it's extending a pattern that
  already exists rather than inventing one.
- *Benefit:* directly matches the stated PDM requirement; low-risk because "everything not explicitly released is
  invisible to Item history" is a safe default that can't accidentally expose noisy intermediate saves.
- *Social:* very legible to users — "release" is already a concept people bring from other PDM/version-control
  tools; low explanation cost.

**Branch B — Automatic/implicit releases** (e.g., every Nth save, or every save that changes a "significant"
attribute) — rejected outright at this stage: it directly contradicts the explicit requirement in issue #48's own
text ("you have control over when to release"), and there's no evidence anyone asked for automatic behavior here.
Included only to record that it was considered and rejected, not as a live option.

**Recommendation for Stage 3: Branch A**, wired to extend the existing `Publisher`/`versionFollowing` concepts
rather than replace them — this stage is comparatively low-risk and low-branching precisely because Lens already
has adjacent, half-built versions of this idea; it mostly needs connecting to the new Item/Document layer from
Stage 1.

---

## Stage 4 — Parameterization vs. item identity (the genuinely open one)

**The question.** This is the stage issue #48's own authors flagged as unresolved, and it's the one where AAS's
type/instance mechanism ([Metamodel §4.2, pp.23–26](https://industrialdigitaltwin.org/wp-content/uploads/2023/06/IDTA-01001-3-0_SpecificationAssetAdministrationShell_Part1_Metamodel.pdf#page=23))
is the most directly relevant — but "relevant" doesn't mean "trivially applicable"; this stage genuinely branches
three ways with real, unresolved tradeoffs on every branch, and should be treated as the riskiest stage in the
whole tree.

**Branch A — Status quo, extended: every parameter combination is its own full `Item`.**
This is what Lens already effectively does via `SharedModel.cloneModelId` (a share link can be a full independent
clone). Formalizing it just means: a configured/parameterized design, once generated, becomes an ordinary Item with
its own Stage-0 identity — no special "type" or "instance" concept at all.
- *Cost:* ~zero beyond Stage 1 — this isn't a new mechanism, just accepting that configuration produces new items,
  same as today.
- *Benefit:* simplest possible model; no new concepts for users to learn.
- *Risk:* exactly the problem issue #48's authors already diagnosed — no way to know that two items are "the same
  design, different parameters" rather than unrelated items; loses the ability to push a type-level fix (a
  corrected hole pattern) out to every configured instance automatically.

**Branch B — Full AAS type/instance split**: introduce `assetKind: Type | Instance` on Item, plus a `derivedFrom`
reference, exactly per the standard.
- *Cost:* substantial — this is a real new relationship type threaded through Items, Documents, and the UI (users
  need to understand "this is a type, that is an instance of it," a genuinely new mental model for Lens users who
  currently think in terms of files and shares).
- *Benefit:* the complete, standards-conformant answer; enables the "push a fix from type to instances" workflow
  that traditional file-centric PDM structurally can't do.
- *Risk:* this is new *design* work, not just new *schema* work — AAS's spec describes the relationship but
  deliberately doesn't prescribe exactly how a type's parameters propagate to instances, or what happens when an
  instance has been locally modified and then the type changes underneath it (a real conflict-resolution question,
  squarely in the territory `optimistic_locking_research.md` surveys — this is a place where that literature's
  syntactic/semantic conflict taxonomy becomes directly actionable, not just background reading). Attempting this
  before Stage 1–3 have been used in production risks building the most speculative part of the whole tree first.

**Branch C — Narrow hybrid: keep single-item identity (Branch A's simplicity), but let a `Model`'s existing
`attributes` field (already present in the schema today!) carry a structured, queryable parameter set**, without
introducing a second identity class. Two items can then be queried/compared as "these share the same underlying
FreeCAD file/template but different attribute values" without a formal type/instance relationship existing in the
schema.
- *Cost:* low — `Model.attributes` already exists (`Type.Optional(Type.Object({}))` in `models.schema.js`); this
  branch is mostly about surfacing and querying it better, not building new relationship machinery.
- *Benefit:* captures most of the practical value (find/compare configured variants) without committing to AAS's
  full type/instance semantics or its unresolved propagation-conflict question.
- *Risk:* doesn't get the "push a type-level fix to all instances" capability — a real, deliberate deferral, not a
  hidden gap.

**Recommendation for Stage 4: defer.** Ship Branch C as a small, low-risk step that captures observable value, and
explicitly hold Branch B in reserve pending real usage data from Stages 1–3. This is the one stage in the tree where
recommending a full commitment now would be premature by the standard this whole document is trying to apply
consistently — everywhere else, a standard maps cleanly onto an already-stated need; here, the standard's mechanism
is real but its *operational* semantics (conflict handling between type and instance edits) are still open even in
the standards literature itself, not just in Lens's own thinking.

---

## Stage 5 — "Meta-documents": from a flat message list toward BCF-shaped discussion

**The question.** `SharedModels.messages` already exists as a flat, unanchored comment list. Issue #48's own aside —
*"possibly... similar to BCF"* — suggests the destination; the question is how much of BCF's actual structure to
adopt, and when.

**Branch A — Minimal: add `status` (open/resolved) and an optional geometry anchor (a saved viewpoint reference) to
the existing `messages` sub-schema**, without adopting BCF's file format or API at all — just borrowing its two most
valuable structural ideas.
- *Cost:* low — extends an existing sub-schema field by field, same pattern as everywhere else in this codebase.
- *Benefit:* closes most of the practical gap (discussion attached to *a specific thing*, with a *resolved* state)
  with minimal new surface area.

**Branch B — Adopt BCF-XML for file-based export/import** (so a Lens discussion thread can round-trip with any
other BCF-compatible tool — Solibri, Tekla, and the broader openBIM ecosystem), per
[buildingSMART/BCF-XML](https://github.com/buildingsmart/bcf-xml).
- *Cost:* moderate — a real serialization format to implement correctly, plus IFC-GUID-style element referencing,
  which needs a FreeCAD-side equivalent (FreeCAD objects don't natively carry IFC GUIDs) — another FC-Worker-shaped
  mapping problem, structurally similar to Stage 2 Branch B's.
- *Benefit:* genuine interoperability with a mature, decade-plus-old ecosystem, if Lens ever needs to exchange
  annotations with BIM-side tools (plausible if FreeCAD parts end up referenced from building/facility models —
  a real, if not immediate, use case).

**Branch C — Adopt BCF-API for live, server-to-server discussion sync**, per
[buildingSMART/BCF-API](https://github.com/buildingSMART/BCF-API) — the heaviest branch, standing up a whole
protocol surface before there's a second server to sync with.
- *Verdict:* premature — this is the Stage-5 analogue of Stage 1's Branch 3 (BaSyx): real standards infrastructure
  built before there's a second party actually needing to interoperate with it.

**Recommendation for Stage 5: Branch A now; keep Branch B on the table specifically if/when a BIM-adjacent
integration becomes real; treat Branch C the same way Stage-1's premature-infrastructure branches were treated.**

---

## Stage 6 — Compliance, audit trail, and the durable-history question

**The question.** Issue #48 names a real compliance driver (*"sometimes a legal requirement, for example in
aerospace through EN 9100 compliance"*). Once Stage 3's "release" concept exists, the question becomes: what
guarantees does a released `DocumentVersion` actually carry? This is where this document's scope brushes against
`possible_data_architecture.md`'s content-addressed-history research, without needing to adopt all of it.

**Branch A — Do nothing extra beyond Stage 3**: a released version is just a Mongo document with a timestamp, same
durability/audit guarantees as everything else in the database (i.e., whatever the ops team's backup/audit practices
already provide, nothing cryptographically stronger).
- *Cost:* zero.
- *Risk:* doesn't actually satisfy "legal requirement" framing in any strong sense — a released version's history
  could in principle be edited in the database with no detectable trace.

**Branch B — Content-hash releases**: when a `DocumentVersion` is released (Stage 3), compute and store a content
hash of its canonicalized document set (the file content plus its Stage 1/2/3 metadata), so tampering or accidental
mutation after release is at least *detectable*, without adopting a full Merkle-DAG store, Automerge, or any of the
heavier machinery `possible_data_architecture.md` surveys for the CRDT/live-collaboration case (which doesn't apply
here — Lens's releases aren't a live-multi-editor scenario, they're a discrete checkpoint, a much simpler problem).
- *Cost:* low — hashing is cheap; the real cost is agreeing on a canonicalization scheme (same discipline flagged
  as "a permanent operational requirement" in `possible_data_architecture.md`) so the same content always hashes
  identically.
- *Benefit:* a real, cheap, verifiable step toward the compliance framing, decoupled from any larger architectural
  bet.

**Branch C — Full content-addressed durable-history layer** (Fluree/Dolt-style, per `possible_data_architecture.md`
Layer 3): genuinely the "correct" long-term answer if Lens ever needs cryptographically verifiable, branchable
history at scale — but a multi-month infrastructure project, not a next step from anything in Stages 0–5.

**Recommendation for Stage 6: Branch B**, explicitly framed as "the cheap 80% of the compliance value," with Branch
C named and deliberately deferred rather than silently forgotten.

---

## Cross-cutting concerns that apply across every stage

These aren't stage-specific branches — they're constraints that should shape *how* any branch above gets executed,
regardless of which one is chosen.

- **This is a `FreeCAD` GitHub-org repo, not a private project — every stage has a governance dimension, not just a
  technical one.** Decisions here ripple into the parallel FreeCAD-core annotation discussion
  ([#25682](https://github.com/FreeCAD/FreeCAD/issues/25682)/[#25685](https://github.com/FreeCAD/FreeCAD/issues/25685))
  and touch pierreporte/Creymore's own already-stated vocabulary. Landing Stage 1's `Item` schema, for instance,
  effectively settles a vocabulary question ("what is an item, really") for the whole ecosystem, not just for Lens's
  codebase — worth explicitly seeking sign-off from the issue #48 participants at the *design* stage of Stage 1, not
  just at PR-review time, since retrofitting a different vocabulary after other people start building against it is
  much more expensive than aligning early.
- **Backward compatibility is non-negotiable at every stage**, because FC-Worker, the Vue frontend, and any existing
  external API consumers all depend on `File`/`Model`/`SharedModel` continuing to behave as they do today. Every
  branch recommended above was deliberately chosen to be *additive* (new collections, new optional fields) rather
  than requiring changes to existing schemas' meaning — this is a real constraint on the tree, not an afterthought:
  several branches that looked technically cleaner (e.g., Stage 1 Branch 2, overloading `Directory`) were downgraded
  specifically because they risked changing existing behavior.
- **Team capacity is a real, separate risk from technical feasibility.** Lens is community-maintained after Ondsel
  (the company) shut down in November 2024 — the realistic velocity for a volunteer/community-driven team is far
  lower than a funded team's. This is the strongest argument in this whole document for the stage-by-stage,
  ship-value-at-each-step structure over any "let's design and build the full PDM" framing: a stalled multi-stage
  effort that shipped Stages 1–3 still leaves Lens meaningfully better off; a stalled big-bang rewrite leaves
  nothing shippable at all.
- **Standards-version drift is a real, ongoing maintenance cost, not a one-time integration cost.** The AAS
  metamodel has already gone through multiple breaking revisions (V2.0 → V2.0.1 → V3.0RC01 → V3.0RC02 → V3.0,
  visible in the spec's own changelog annex). Every stage that stores AAS-*shaped* data (which is most of them, per
  the "adapter over adoption" discipline) should pin to a specific IDTA template version explicitly and treat
  upgrading it as a deliberate, reviewed decision — not something that happens silently because a field name matched
  a newer spec revision by coincidence.

---

## The recommended path, stated as one line per stage

For reference when discussing this with the Lens/FreeCAD maintainers — this is the pruned path through the tree
above, not a claim that the other branches are unreasonable:

0. **Identity:** internal ID now; AAS-shaped `globalAssetId` exported later, only when a real external consumer
   needs it (Stage 0, Branch D).
1. **Items:** new `items` collection, VDI 2770 Document/DocumentVersion-shaped, referencing existing
   `File`/`Model` docs, no new infrastructure (Stage 1, Branch 1).
2. **Assemblies:** ship lightweight manual item-links first to validate demand; migrate the FreeCAD-native portion
   to FC-Worker-derived, single-source-of-truth links once validated (Stage 2, Branch A → C).
3. **Releases:** explicit, user-triggered release action promoting a `Model` snapshot to a `DocumentVersion`,
   extending the existing `Publisher`/`versionFollowing` concepts (Stage 3, Branch A).
4. **Parameterization:** defer the full type/instance relationship; ship queryable structured attributes on
   existing `Model.attributes` as the low-risk interim step (Stage 4, Branch C).
5. **Discussion:** extend the existing `messages` sub-schema with status and geometry-anchor fields; hold BCF-XML
   export and BCF-API sync in reserve for a real cross-tool integration need (Stage 5, Branch A).
6. **Compliance:** hash released document sets for tamper-evidence now; hold a full content-addressed history
   layer in reserve for if/when Lens needs branchable, queryable historical state at real scale (Stage 6, Branch B).

Every stage above is independently shippable, independently valuable even if nothing after it ever happens, and
none of them requires the Lens team to adopt AAS-server infrastructure, BCF infrastructure, or a distributed
content-addressed database before there's concrete evidence such infrastructure is earning its cost. That is the
direct answer to the original question: this is buildable, and the way to keep it from requiring premature
irreversible decisions is to insist, at every single stage, on the cheapest branch that doesn't foreclose the more
standards-complete branch later — not to pick the "best" architecture up front and build toward it in one motion.

## Sources

Standards and prior-art sources are the same as [`ondsel_48_research.md`](ondsel_48_research.md)'s Sources section
and are not repeated here; this document adds no new external sources beyond those already cited there, and instead
cites specific files inside the cloned `Ondsel-Server` repository, listed inline above at each point they inform a
decision:
- `backend/src/services/workspaces/workspaces.schema.js`
- `backend/src/services/directories/directories.schema.js`
- `backend/src/services/models/models.schema.js`
- `backend/src/services/shared-models/shared-models.schema.js`
- `backend/src/services/shared-models/message.schema.js`
- `backend/src/services/publisher/publisher.schema.js`
- `backend/src/curation.schema.js`
- `docs/technical.md`

## Cross-reference

This document is the implementation-feasibility follow-through on [`ondsel_48_research.md`](ondsel_48_research.md)'s
§4 proposed responses and on [`possible_freecad_collaboration.md`](possible_freecad_collaboration.md)'s ranked
contribution targets. It intentionally does not repeat the standards descriptions or the conflict-taxonomy research
from [`optimistic_locking_research.md`](optimistic_locking_research.md) — it cites them at the specific points
(Stage 2, Stage 4) where they become directly actionable rather than background context.
