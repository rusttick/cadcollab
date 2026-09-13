# Ondsel-Server Issue #48: What They're Actually Trying to Solve, and the Standards That Already Solve It

Purpose: read `Ondsel-Server` issue #48 closely (a local copy is at
[`ondsel-server-issue-48.md`](ondsel-server-issue-48.md); the live thread is
[FreeCAD/Ondsel-Server#48](https://github.com/FreeCAD/Ondsel-Server/issues/48)), understand what Lens is and what its
maintainers are actually reaching for underneath the vocabulary they're inventing in real time, then map that onto
Asset Administration Shell (AAS) and adjacent international-standards work already surveyed in
`../co-edit-stream/doc/possible_agile_manufacturing.md` and `possible_data_architecture.md` — confirming with fresh,
targeted research (not just the earlier survey) exactly which existing standard answers which part of their problem.

---

## 1. What Ondsel-Server / Lens actually is

Confirmed by reading the cloned repo (`Ondsel-Server/`), not just the marketing description:

- **Stack**: Node.js/Feathers.js backend (`backend/src/services/*`), MongoDB for structured data, Vue.js frontend,
  a dedicated FreeCAD-core-based "FC-Worker" microservice (FastAPI + Celery) that does model processing/rendering,
  configurable file storage (local or S3). AGPL-3.0-or-later, self-hostable via docker-compose, OIDC/Keycloak
  support. Community-owned under the `FreeCAD` GitHub org since Ondsel the company shut down in November 2024.
- **Current data model** (from `backend/src/services/workspaces/workspaces.schema.js` and
  `backend/src/services/models/models.schema.js`): a **Workspace** is a permissioned container with a root
  **Directory** tree of **Files**. A **Model** is explicitly documented in the code itself as *"a snapshot in time
  for a specific combination of: 1. File Version, 2. SharedModel (Link), 3. User Parameters for the File Revision's
  attributes."* In other words: Lens today versions **files** (and file-derived render artifacts — OBJ/STL/STEP/FCStd
  exports, thumbnails), and lets a file's *parameters* vary per "Model" snapshot, but there is no first-class
  **item** concept — no persistent identity for "this part" that survives across file versions, no typed link
  between an assembly's identity and its constituent parts' identities, independent of which specific file version
  happens to be checked in right now.
- This confirms, from the code, exactly the gap issue #48 states in prose: *"Lens currently only manages (versions
  of) files but items are critical for a PDM."*

## 2. Reading issue #48 closely — what's really being asked for, underneath the invented vocabulary

The issue is a transcript of a live design conversation (pierreporte, Creymore) that starts from a narrower question
(should collaboration annotations be versioned — [FreeCAD/FreeCAD#25682](https://github.com/FreeCAD/FreeCAD/issues/25682))
and widens into "what would it take for Lens to be a real PDM." Reading it closely, several things are happening at
once that the surface text doesn't separate out:

1. **They are re-deriving a standard PDM vocabulary from first principles, live, in a GitHub issue.** The terms
   *model*, *link*, *part*, *assembly*, *sub-assembly*, *item*, *document*, *drawing*, *specification*, *file*,
   *version*, *container*, *primary content*, *attachment* are all defined inline because — tellingly — nobody in
   the thread cites an existing standard's vocabulary for any of them. This is the single strongest piece of
   evidence that the participants don't currently have a standards reference in view; they're solving a
   40-years-solved classification problem from scratch because they don't know a solved version exists nearby.
2. **The "item" gap is precisely the type/instance-of-documentation problem, not a novel one.** Their own
   definition — *"item: a representation of a part or assembly in terms of documents"* / *"document: a single piece
   of documentation for an item... consists of the primary content and possibly attachments"* — is, word for word,
   the **Document / DocumentVersion** relationship formalized by **VDI 2770 Blatt 1** and consumed by the AAS
   **Handover Documentation** submodel template (detailed in §4 below). They reinvented VDI 2770's core two-class
   model without knowing VDI 2770 exists.
3. **The "meta-document" invention is a symptom of not knowing BCF exists.** Their proposed new term —
   *"meta-document: a document that does not contribute to the specification of an item but allows discussion of
   the item"* — plus the throwaway remark *"possibly in the FreeCAD file itself or as a separate file similar to
   BCF"* is the tell: they've already half-noticed that a solved format (BIM Collaboration Format) covers exactly
   this, floated it, and then dropped it rather than pursuing it, defaulting instead to inventing a bespoke concept.
   This is worth surfacing as a direct answer rather than a new abstraction.
4. **The parameterization/identity tension is a real, harder problem, and is where they're closest to genuinely
   "grasping."** Their own analysis is honest and mostly correct: *"Parameterization of parts is something that
   PDMs typically don't provide since one item is only for one object and the unique ID that identifies an object
   cannot be parameterized... PDMs... take the pragmatic route to store each variant as a separate item. However,
   this does not necessarily have to be the case if the ID can encode a specific set of parameters."* This is a
   real tension in traditional file-centric PDM (SolidWorks PDM, Windchill, Teamcenter) — but it is **not** actually
   unsolved in the broader standards world: it is precisely the **type-asset / instance-asset** relationship AAS
   defines explicitly, with a normative `derivedFrom` relationship (§5 below). The Lens team is rediscovering, from
   the PDM side, a problem the Industry 4.0 side already named and standardized a solution for.
5. **What's actually novel about Lens (and not covered by any of the above) is the FreeCAD-to-anonymous-user
   distribution goal** — hosting parameterized designs that non-FreeCAD users can configure and download as
   STL/STEP. This really is outside classic PDM scope (their own conclusion — *"Lens has other goals beyond the
   goals of a PDM"*) and outside AAS's scope too; nothing below claims to solve this part. It's flagged here so a
   response to #48 doesn't overclaim.

**Read between the lines, the honest summary is:** Lens's maintainers have correctly diagnosed that they're missing
an item/document layer and a parameterized-identity mechanism, they are reasoning about both soundly and pragmatically,
but they are doing all of this without contact with the (already mature, already-standardized) international work
that answers most of it — because nobody in the thread is coming from the Industry 4.0 / AAS / buildingSMART
standards world. This is not a criticism of the thread — pierreporte and Creymore's own definitions are careful and
mostly compatible with the standards below — it's an observation about where a well-aimed comment can save real
re-invention work.

## 3. AAS and related standards: what's already been accomplished, and where it's adopted

This section is a fresh, targeted pass specifically answering "which parts of issue #48 does which standard cover,"
building on (and citing, not repeating) the broader survey already done in
[`possible_agile_manufacturing.md`](../co-edit-stream/doc/possible_agile_manufacturing.md) and
[`possible_data_architecture.md`](../co-edit-stream/doc/possible_data_architecture.md).

### 3.1 The Asset Administration Shell (AAS) itself — governance and maturity

- **IEC 63278** — the AAS metamodel is a published IEC International Standard, not merely an industry consortium
  spec. Maintained and specified in detail by the **Industrial Digital Twin Association (IDTA)**, a body with
  100+ member companies (Bosch, Siemens, SAP, Festo, Schneider Electric, ZVEI/Plattform Industrie 4.0 members,
  etc.).
  [IDTA 01001-3-0 — Specification of the AAS, Part 1: Metamodel](https://industrialdigitaltwin.org/wp-content/uploads/2023/06/IDTA-01001-3-0_SpecificationAssetAdministrationShell_Part1_Metamodel.pdf)
  (free PDF, no login).
- **Eclipse BaSyx** (github.com/eclipse-basyx) is a real, MIT/Apache-licensed, production-used open-source
  implementation (registries, submodel servers, SDKs) — not vaporware, an actual codebase (already flagged in
  `possible_agile_manufacturing.md`).
- **IDTA Submodel Template Repository** ([admin-shell-io/submodel-templates](https://github.com/admin-shell-io/submodel-templates))
  is an open, actively growing library of standardized, reusable submodel *types* — the mechanism by which AAS
  avoids one team owning the whole schema. Two of these templates map directly onto issue #48's gaps (below).

### 3.2 The item/document gap → **VDI 2770 Blatt 1** + AAS **Handover Documentation** submodel (IDTA 02004)

Verified directly from the specification PDF
([IDTA 02004-2-0, June 2025](https://industrialdigitaltwin.org/wp-content/uploads/2025/07/IDTA-02004-2-0_Submodel_Handover-Documentation.pdf),
free, no login):

- **§1.2 Scope of the Submodel** (p.3): *"The Submodel Handover Documentation defines a standardized exchange
  format for information or documentation for a specific asset. This can be both type and instance information...
  In case a machine manufacturer sells a machine to a customer (operator), the manufacturer hands over the machine
  and its documentation in form of an AAS with the Submodel 'Handover Documentation'."* — this is a standards body's
  answer to precisely the workflow Lens is trying to support (a design, handed over with its documents, to a
  downstream party).
- **§1.3 Relevant standards** (p.3) states the submodel's meta data and classification are based directly on
  **VDI 2770 Blatt 1** ("Operation of process engineering plants — Minimum requirements for digital manufacturer
  information"), whose central concepts are two entities: **Document** (*"the understanding of a document in total
  as a specific concept of product-related information"*) and **DocumentVersion** (*"a specific instance of the
  'Document' within its lifecycle, e.g. a released version of the Document"*) — this is, structurally, exactly
  Ondsel's own **item → document → version** definitions, already formalized with a mandatory metadata schema
  (§2.4–2.8, pp.7–17): DocumentId, DocumentClassification (a controlled vocabulary — see Table 1, p.7:
  Identification, Technical specification, Drawings/plans, Assemblies, Certificates, Commissioning, Operation,
  Safety, Inspection/maintenance, Repair, Spare parts, Contract documents), DocumentVersion with StatusValue,
  Language, and one-or-more DigitalFiles.
- **§2.2 Association of documents to Assets and Entities** (pp.5–6) is the direct answer to Ondsel's
  "assembly of parts, each with their own documents" structure: a complex piece of equipment's Handover
  Documentation submodel can mark constituent supplier parts as **Entities**, each either **"co-managed"** (part of
  the parent AAS) or **"self-managed"** (its own independent AAS, referenced by `globalAssetId`), with
  `RefersTo`/`DocumentedEntity` reference elements linking documents to the correct entity — Figure 4 (p.6) diagrams
  exactly the "AAS for equipment, referencing supplier-part AAS via AssetId" pattern that maps onto Lens's
  "assembly → part, each described by its own documents" model, already worked out and standardized rather than
  needing to be designed from scratch.
- **Adoption**: VDI 2770 Blatt 1 is a published VDI guideline (German engineering-standards body, comparable
  standing to ISO/IEC national-committee-adjacent standards) already required/referenced in real supplier-handover
  practice in the process industry; the AAS submodel built on it is IDTA-published and implemented in tooling like
  the **AASX Package Explorer** (screenshotted directly in the spec, p.5).

### 3.3 The assembly/link structure → AAS **Hierarchical Structures enabling Bills of Material** (IDTA 02011)

[IDTA 02011-1-1, June 2024](https://industrialdigitaltwin.org/en/wp-content/uploads/sites/2/2024/06/IDTA-02011-1-1_Submodel_HierarchicalStructuresEnablingBoM.pdf)
(free PDF) — a published, IDTA-approved submodel template whose stated purpose is to serve as *"the authoritative
source for hierarchical structures within an AAS during all lifecycle phases,"* explicitly covering the "asset
composed of sub-assemblies composed of parts" recursive-composition pattern from `possible_agile_manufacturing.md`
§"Solving depth." This is the standardized version of Ondsel's own **model has links to other models (assembly →
part/sub-assembly)** definition — already a public template rather than something Lens would need to design.

### 3.4 The parameterization/identity tension → AAS **type asset / instance asset**, with `derivedFrom`

Verified directly from the Metamodel spec, **§4.2 "Types and Instances"** (pp.23–26):

- **§4.2.1** (p.23): *"The RAMI4.0 model defines a generalized life cycle concept... The basic idea is to
  distinguish between possible types and instances for all assets within Industry 4.0... to distinguish asset
  'type' and asset 'instance', the term 'asset kind' is used in this document."* Table 1 (p.24) spells out the
  life-cycle roles explicitly: a **type asset** is created during Development and carries the shared design (CAD
  data, schematics, value ranges, description); an **instance asset** is created during Production from a type
  asset and carries instance-specific data (serials, measured values, usage/maintenance history).
- **§4.2.2** (pp.24–25), worked example (Figure 3, p.25): a sensor **type** AAS (`assetKind = Type`, ID
  `0215551AA`, carrying shared attributes: manufacturer, value range, product class, description) has two
  **instance** AASes (`assetKind = Instance`, IDs `0215551AAA_T1` / `_T2`, each with its own `measuredTemperature`)
  that reference the type AAS via the explicit relationship attribute **`derivedFrom`**.
- **This is a direct, already-standardized answer to Ondsel's exact stated dilemma**: *"the unique ID that
  identifies an object cannot be parameterized as that item should have a different ID... [but] this does not
  necessarily have to be the case if the ID can encode a specific set of parameters."* AAS's answer: don't try to
  encode the parameters into one ID at all — keep a stable **type** identity for the parameterized general design
  (the shared, unconfigurable properties) and give each configured **instance** its own stable identity plus an
  explicit `derivedFrom` pointer back to the type. Item identity and configuration/parameterization stop being in
  tension because they're modeled as two different, linked assets rather than one asset trying to be both.

### 3.5 The "meta-document" / discussion-without-versioning gap → **BCF (BIM Collaboration Format)**

[buildingSMART Technical — BCF](https://technical.buildingsmart.org/standards/bcf/) and
[buildingSMART/BCF-XML — GitHub](https://github.com/buildingsmart/bcf-xml) (both free, no login). BCF is a mature,
governed, open **buildingSMART International** standard (originally developed by Tekla and Solibri, later formally
adopted by buildingSMART, alongside IFC and bSDD as one of its core openBIM standards) built for exactly the case
issue #48 names in passing: *"transferring... contextualized information about an issue or problem, directly
referencing a view... and elements of a BIM, as referenced via their IFC GUIDs, from one application to another"* —
i.e. a **comment/topic thread pinned to specific model elements, that does not itself version the model**. It ships
in two forms: a file-exchange XML format, and a RESTful web-service API (**BCF-API**,
[buildingSMART/BCF-API](https://github.com/buildingSMART/BCF-API)) with implementer agreements already worked out —
directly relevant to Lens's *"attach version information to it on the server"* framing, since BCF's web-service mode
is precisely a server-hosted, non-file, comment-thread model. Ondsel's own thread already gestured at this
("similar to BCF") without following through — this is the single most directly-reusable piece of prior art in the
whole issue.

### 3.6 Stable cross-discipline identity → IEC 81346 (already surveyed)

Not re-derived here — already covered in depth in `possible_agile_manufacturing.md` §"Solving depth." Relevant to
#48 specifically because Lens's items, models, and documents all eventually need identifiers stable across file
revisions, exports (STEP/STL/FCStd), and — per §3.4 above — across type/instance splits; IEC 81346 is the existing
standard for exactly that stable identity layer, already adopted by commercial EDA tooling (EPLAN) and proposed
as the identity scheme in the earlier co-edit-stream research.

## 4. Proposed responses to issue #48, mapping standards onto their stated goals

Ranked by how directly each maps onto something already said in the thread:

1. **Point at VDI 2770 / AAS Handover Documentation (IDTA 02004) as the existing formalization of their own
   item/document/version definitions.** This is close to a direct restatement of what they already wrote, with
   citations — very low cost, high credibility, because it shows their own reasoning independently converged with
   a published international standard rather than contradicting it.
2. **Point at BCF as the direct, ready-made answer to "meta-document."** They already half-suggested this
   themselves; following through with the actual spec/repo links turns an abandoned aside into a concrete adoption
   candidate, and it's a mature, governed, two-decade-proven format from an adjacent AEC/BIM standards body, not a
   novel design.
3. **Point at AAS's type/instance split (`assetKind`, `derivedFrom`) as the resolution to the parameterization vs.
   item-identity tension** they identified as a genuinely hard, semi-open problem. This is the highest-value single
   pointer in the whole response, because it's the one place their own analysis explicitly stalled ("this does not
   necessarily have to be the case if...") without a concrete mechanism — AAS has a normative, already-worked-out
   mechanism.
4. **Mention the Hierarchical Structures BoM submodel (IDTA 02011) as the standardized version of their
   model/link/assembly/sub-assembly definitions**, so a from-scratch schema for assembly composition isn't
   necessary.
5. **Be honest that Lens's anonymous-configuration/distribution goal is genuinely outside all of the above** — AAS,
   VDI 2770, and BCF all assume an identified asset/document context, not an anonymous public configurator; this
   part of Lens's ambition remains real, unclaimed design work, not something a standards pointer resolves.

## Sources

- [FreeCAD/Ondsel-Server issue #48 — "Extent to which Lens is a PDM"](https://github.com/FreeCAD/Ondsel-Server/issues/48)
- [FreeCAD/FreeCAD issue #25682 — Core: Improve annotations as a collaboration feature](https://github.com/FreeCAD/FreeCAD/issues/25682)
- [FreeCAD/Ondsel-Server — GitHub repository](https://github.com/FreeCAD/Ondsel-Server)
- [IDTA 01001-3-0 — Specification of the Asset Administration Shell, Part 1: Metamodel (PDF)](https://industrialdigitaltwin.org/wp-content/uploads/2023/06/IDTA-01001-3-0_SpecificationAssetAdministrationShell_Part1_Metamodel.pdf)
- [IDTA 02004-2-0 — Submodel Template: Handover Documentation, v2.0 (PDF)](https://industrialdigitaltwin.org/wp-content/uploads/2025/07/IDTA-02004-2-0_Submodel_Handover-Documentation.pdf)
- [IDTA 02011-1-1 — Submodel Template: Hierarchical Structures enabling Bills of Material (PDF)](https://industrialdigitaltwin.org/en/wp-content/uploads/sites/2/2024/06/IDTA-02011-1-1_Submodel_HierarchicalStructuresEnablingBoM.pdf)
- [admin-shell-io/submodel-templates — GitHub](https://github.com/admin-shell-io/submodel-templates)
- [Eclipse BaSyx](https://eclipse.dev/basyx/)
- [buildingSMART Technical — BIM Collaboration Format (BCF)](https://technical.buildingsmart.org/standards/bcf/)
- [buildingSMART/BCF-XML — GitHub](https://github.com/buildingsmart/bcf-xml)
- [buildingSMART/BCF-API — GitHub](https://github.com/buildingSMART/BCF-API)
- [BIM Collaboration Format — Wikipedia](https://en.wikipedia.org/wiki/BIM_Collaboration_Format)
- [`possible_agile_manufacturing.md`](../co-edit-stream/doc/possible_agile_manufacturing.md) (this project's earlier AAS/IEC 81346/ISO 15926 survey)
- [`possible_data_architecture.md`](../co-edit-stream/doc/possible_data_architecture.md) (this project's earlier content-addressed-history/CRDT research)
- [`ondsel-server-issue-48.md`](ondsel-server-issue-48.md) (local copy of the issue text used for this research)

## Cross-reference

This document sits alongside [`possible_freecad_collaboration.md`](possible_freecad_collaboration.md) §3 item 1,
which already flagged issue #48 as the single highest-leverage contribution target in the FreeCAD ecosystem survey.
This document is the deeper, standards-side follow-through on that recommendation — confirming, with primary-source
citations to specific sections and page numbers, exactly which existing standard answers which part of the thread's
own stated gaps, rather than only asserting that a mapping exists.
