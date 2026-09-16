# Human Factors, Boundary-Crossing, and What FreeCAD Would Actually Need to Dominate

Purpose: this project's research so far (`possible_freecad_collaboration.md`,
`core_vs_addon_collaboration_features.md`) mapped the *technical* state of
FreeCAD collaboration efforts. A companion critique
(`freecad_for_co_edit_stream_research.md`'s discussion thread) flagged that
this project's own architecture docs sometimes reach for distributed-systems
abstractions (federated graphs, IEC 81346 identity, statement/qualifier
provenance) that real users, per the arXiv CAD-version-control paper, never
actually asked for. This doc goes one level deeper on two fronts the prior
research didn't cover directly:

1. What the human-factors/CSCW/organizational-behavior literature says is
   *actually* needed for large-scale collaborative engineering work — not
   just "what do CAD users complain about" but "what makes multi-party,
   cross-organization technical collaboration succeed or fail, structurally."
2. What it would take for FreeCAD specifically to become the dominant,
   go-to platform for **boundary-crossing** engineering collaboration —
   work that has to cross company, discipline, *and regulatory* lines —
   and whether that goal is realistic as stated.

Methodology: literature/web research (CSCW, boundary-object theory, PLM
adoption-psychology studies, AEC/BIM regulatory-interchange precedent,
export-control compliance literature, open-source market-dynamics
research), read critically against this project's own prior findings and
FreeCAD's actual current position. Not a first-principles architecture
proposal — a reality check against evidence from adjacent industries that
have already tried to solve pieces of this problem.

## 1. What the human-factors research actually says is needed

### Awareness and common ground, not synchronization

CSCW research on multidisciplinary design teams treats **awareness** —
knowing what collaborators are doing, have done, and are about to do — as
the load-bearing concept, not real-time synchronization. "We-awareness"
(shared understanding of group state, not just individual peripheral
awareness of others) and **common ground theory** — the shared knowledge
about tasks, roles, and process that lets one person interpret another's
work correctly — are what CSCW systems are actually built to support.
Distributed engineering teams spend enormous effort re-establishing common
ground after any gap (a new participant, a long pause, a handoff across
time zones) — and losing common ground, not losing byte-level sync, is
what causes rework.

This directly supports this project's own prior conclusion (repeated across
`possible_freecad_collaboration.md` and the arXiv paper review): the
lever that matters is **legible state and history**, not simultaneity.
A tool that makes "what changed, by whom, why, and what does that
invalidate downstream" instantly answerable is solving the actual
awareness problem. A tool that makes two cursors move in the same
document in real time is solving a much narrower, much less
load-bearing problem — CSCW literature calls this out explicitly: awareness
support and synchronous presence are different design targets, and the
former is the one that scales past two co-located people.

### Boundary objects: the actual mechanism of cross-boundary collaboration

Star & Griesemer's boundary-object theory (1989, still the dominant lens in
this literature) is the most directly applicable concept this research
turned up and isn't yet named anywhere in this repo's docs. A boundary
object is an artifact that is **simultaneously flexible enough to be
useful locally** to each community that touches it, and **robust enough to
maintain a shared identity** across communities that never fully agree on
what it means. It doesn't require shared ontology, shared tooling, or
shared authority — that's the point. It requires only:

- **interpretive flexibility** — each party can read/use it in the terms of
  their own discipline (a structural engineer, a regulator, a
  manufacturer, and a project owner can each extract what they need from
  the same object without agreeing on a unified schema for what the
  object "really" is), and
- **a stable, shared identity** across those uses — a specific STEP file
  revision, a specific drawing number, a specific approved BOM line — that
  everyone can point at and mean the same referent.

This is a materially different design target than this project's
prior core-vocabulary thinking (part-of/connected-to/satisfies/verified-by
relationships with a single reconciled ontology). Boundary-object theory
says the ontology reconciliation is *not the prerequisite* for successful
cross-org collaboration — stable reference + local interpretive freedom is
enough, and has been the empirically observed mechanism in every
cross-organizational engineering effort surveyed below (aerospace LOTAR,
AEC/openBIM, museum specimen classification, agile systems engineering).
Chasing a single global ontology is where over-engineered "collaboration
platform" efforts tend to stall; a stable identifier plus an open exchange
format is what has actually shipped, repeatedly, across industries.

### Why engineers actually resist PDM/PLM — and why it's not a UX bug

The PLM-adoption-psychology research is unambiguous and directly
contradicts a common assumption in the CAD-tooling world (including,
implicitly, some of this project's earlier framing): engineers don't
resist PLM/PDM because the software is unpolished. They resist it because
adoption **removes autonomy and flexibility** in exchange for a benefit
they cannot see, and they correctly predict that a system which doesn't
have full participation will accumulate untrustworthy data — so they don't
trust it precisely *because* not everyone is using it, and it doesn't get
everyone using it because it isn't trusted. This is a self-reinforcing
adoption trap, not a rollout-timeline problem: research frames it
explicitly as a *human/organizational* failure mode that precedes and
causes the technical failure, and cautions that fixing UX polish alone
does not break the trap. What actually breaks it is closing the specific
loop — "how does this make my job easier, in terms I already care about" —
before or during rollout, not after.

This matters for a project aiming at "FreeCAD dominance": any tool whose
value proposition to an individual engineer is "the organization benefits
from your compliance" rather than "you personally lose less time," will
face this exact resistance pattern regardless of how good the underlying
architecture is. This is an argument for treating single-user ergonomics
(fast, no admin friction, immediate personal benefit) as the actual gate
collaboration features have to pass, not an argument against building
collaboration features.

## 2. The industries that have actually solved boundary-crossing, regulator-inclusive collaboration

Two precedents exist where "many independent organizations plus a
regulator must all trust and jointly act on the same technical model" was
solved at real scale, not just theorized about. Both are directly
transferable, and neither looks like this project's earlier
distributed-systems-flavored proposals.

### AEC/openBIM: ISO 19650 + the Common Data Environment (CDE)

Construction is the closest real-world analogue to "boundary-crossing
engineering collaboration including regulators," and it already has a
mature, ISO-standardized, widely deployed answer: the **Common Data
Environment**. The mechanism, stripped to its essentials:

- **A staged, gated workflow** (Work In Progress → Shared → Published →
  Archived) where moving a model or document between stages requires
  explicit sign-off, and that approval chain is mapped to **contractual
  responsibility, not org-chart hierarchy** — i.e. the CDE encodes who is
  legally answerable for what, which is exactly the missing piece in a
  naive "everyone edits everything" collaboration model.
- **Open, vendor-neutral exchange formats** (IFC for the model, BCF for
  issue/comment threads) are the CDE's actual technological-neutrality
  guarantee — any tool can produce/consume, so no single vendor is a
  chokepoint and regulators/owners aren't locked into a tool choice.
  This is boundary-object theory in production: IFC files are the stable
  shared referent, and every discipline extracts and edits only the
  IFC entities relevant to it.
- **Full audit trail and versioning as a first-class CDE requirement**,
  not an add-on — every information container's status, revision, and
  approval history is tracked, because it's the only way multiple legally
  distinct parties can trust a shared model enough to build/permit/inspect
  against it.

FreeCAD's ecosystem is independently re-deriving a much smaller version of
this (per this repo's `freecad_core_and_lens_collaboration_issues.md` and
`possible_freecad_collaboration.md`: Ondsel-Server's model/part/item/link
debate, FreeCAD core's annotation/conversation work). The CDE precedent
says the shape that has actually worked at regulator-inclusive scale is:
**staged approval gates keyed to legal responsibility + an open exchange
format + full audit trail**, not a live-editable shared graph. That's a
much more constrained, much more buildable target than this project's
earlier "federation of typed-relationship graphs" framing, and it already
has 15+ years of production validation in an adjacent, harder-regulated
industry.

### Aerospace/defense: STEP AP242 + LOTAR

The other working precedent is narrower but even more directly
boundary-crossing-with-regulators: **STEP AP242** ("Managed Model Based 3D
Engineering," ISO 10303 AP242) is the merger of the aerospace and
automotive STEP application protocols, explicitly built and iterated with
direct involvement from regulatory bodies (EASA, FAA, via the Aerospace &
Defense PLM Action Group) because aircraft manufacturers are legally
required to retain readable design data **for the operational life of the
airframe** — often 50+ years, far past any single CAD vendor's product
lifetime. **LOTAR** (LOng Term Archiving and Retrieval) is the resulting
standard for archiving that data in a vendor-neutral, regulator-acceptable
form.

The transferable lesson: regulatory acceptance of an exchange format
doesn't come from a regulator endorsing a tool — it comes from an **open,
versioned, independently-implementable standard** that multiple
competing vendors (and a neutral archive) can all read/write
losslessly, with the regulator caring about the standard's stability and
completeness, not which application produced or consumes a given file.
FreeCAD's actual leverage point for "regulatory-grade boundary crossing"
is STEP/AP242 conformance and long-term-archival-quality export, not a
bespoke FreeCAD-native collaboration protocol that a regulator would have
no reason to trust or standardize on.

### The hard constraint neither the AEC nor aerospace precedent gets to skip: export control and IP

This is the piece missing from a purely technical framing of "boundary
crossing." Cross-organization CAD sharing in regulated industries is not
just a data-format problem — it's a **legal access-control problem**.
ITAR/EAR "deemed export" rules mean that granting a foreign national
access to a controlled CAD model — even briefly, even in a screen-share,
even to an ally-nation engineer — can itself be an unauthorized export,
independent of whether anything crosses a border. Research on this
explicitly flags general-purpose collaboration platforms (shared drives,
generic cloud storage, even generic Git hosting) as high-risk specifically
*because* they don't do fine-grained, auditable, per-recipient access
control matched to legal authorization, not because they lack real-time
sync.

This has a sharp implication for any "global, freely available,
boundary-crossing CAD platform" ambition: **the collaboration
infrastructure has to support per-party, auditable, revocable access
control and a clean access log as a core primitive**, not a bolt-on
permission system. A federation-of-graphs or open-P2P-sync design (this
project's earlier framing, and Ondsel-Server issue #47's from-scratch
sync proposal) needs to answer "how does party A cryptographically prove
to an export-control auditor exactly who saw exactly which revision of
this model, and when access was revoked" before it can be taken seriously
for regulated cross-border work — that requirement is more load-bearing
than any ontology question this project has previously spent time on.

## 3. What FreeCAD would actually need, ranked by leverage

Given the above, here's a realistic (not aspirational) list of what
"dominating boundary-crossing global engineering collaboration" would
require of FreeCAD specifically, ranked by what the evidence says
actually moves the needle:

1. **Rock-solid, regulator-grade STEP AP242 import/export**, verified
   against real LOTAR/aerospace conformance suites — not just "opens most
   STEP files." This is the single highest-leverage move: it's the one
   piece of infrastructure that lets FreeCAD slot into existing
   regulator-trusted supply chains without needing anyone (a regulator, a
   prime contractor, a hospital procurement office) to adopt FreeCAD
   itself. FreeCAD only has to be a trustworthy *participant* in an
   already-standardized exchange, not the platform of record.
2. **A CDE-shaped collaboration layer, not a live-editing layer** — staged
   WIP → Shared → Published → Archived states, approval gated to
   contractually-real responsibility, full audit trail, built on open
   formats. This is squarely addon/server territory per this repo's own
   `core_vs_addon_collaboration_features.md` finding — buildable today,
   without waiting on FreeCAD core, most plausibly by pointing
   Ondsel-Server's item/model/link redesign (issue #48) at the CDE
   pattern instead of inventing a bespoke one.
3. **Fine-grained, auditable, revocable access control with an
   export-control-usable access log**, designed in from the start rather
   than retrofitted — this is the actual gate on regulated/defense/medical
   boundary-crossing use, and it's currently absent from every project
   surveyed in this repo's ecosystem census.
4. **Individual-engineer-first ergonomics on every collaboration feature**
   — per the PLM-adoption-psychology research, any feature whose payoff is
   "the org benefits" rather than "you personally save time today" will be
   resisted into irrelevance regardless of architecture quality. Every
   collaboration feature needs a concrete answer to "what does this save
   *me*, today, with a team of any size, including size 1."
5. **A funded, professional support/maintenance offering** — the market
   research is blunt that enterprise non-adoption of open-source CAD is
   dominated by fear of *support risk*, not licensing cost or feature
   gaps. FreeCAD's own governance is volunteer-driven with a ~€230k/year
   budget (2026) — roughly two to three engineers' fully-loaded cost.
   That is nowhere near sufficient to offer the kind of support SLA that
   would let a regulated enterprise or government agency choose FreeCAD
   over SolidWorks/NX for anything mission-critical. This is the actual
   ceiling on "market leader" ambitions, and no amount of architecture
   work changes it — it requires a funding/business-model answer (a
   support-vendor ecosystem, akin to Red Hat/Linux, rather than the FPA
   itself scaling to enterprise-support size).

## 4. Critical assessment: is "go-to market leader" a realistic goal?

No, not in the sense of displacing SolidWorks/AutoCAD/Onshape across the
general CAD market. FreeCAD's current market share (~0.01%, against
AutoCAD's ~38% and SolidWorks' ~14%) reflects a genuinely enormous gap in
polish, support infrastructure, and ecosystem maturity that this project's
collaboration-architecture work does not address and cannot close alone.
Chasing "market leader" as a literal target would repeat the same mistake
this project has already been warned away from once (the arXiv-paper
review): building toward a broad, aspirational vision instead of the
specific, evidenced gap.

A **defensible, narrower version of the goal is realistic**, and the
research above supports it directly: FreeCAD becoming the **default
neutral participant in boundary-crossing, multi-party, regulator-touched
engineering work** — not the tool every engineer uses day to day, but the
tool that can sit at the edge of a supply chain, a public-infrastructure
project, an academic/humanitarian collaboration, or a small-supplier
network and speak the open formats (STEP/AP242, IFC-adjacent patterns)
and staged-approval/audit patterns that regulators and primes already
trust — the same role LibreCAD/FreeCAD already occupy in SME and
education contexts, extended upward into the regulated-exchange role by
being the best-conforming, best-audited open implementation of standards
those industries already rely on. That's a real, winnable position: it
plays to open source's actual comparative advantage (no vendor lock-in,
inspectable/auditable code — itself valuable for regulator trust — free
global access for under-resourced participants) rather than competing
head-on where FreeCAD is weakest (polish, support SLAs, ecosystem breadth).

This also reframes what this project should build toward: not a novel
collaboration ontology or a federation protocol, but **standards
conformance, audit/access-control primitives, and a staged-approval
addon layer** — all boring, all evidenced by industries that already
solved adjacent versions of this problem, and all achievable without a
FreeCAD core change per this repo's own prior finding.

## Sources

- [From I-Awareness to We-Awareness in CSCW — Springer](https://link.springer.com/article/10.1007/s10606-014-9215-0)
- [Evaluating Computer-Supported Cooperative Work: Models and Frameworks](https://people.eecs.berkeley.edu/~jfc/cs260/f06/lecs/lec7/p112-neale.pdf)
- [Computer Supported Cooperative Work in Design — Springer book series](https://link.springer.com/book/10.1007/11568421)
- [Boundary object — Wikipedia](https://en.wikipedia.org/wiki/Boundary_object)
- [Boundary Objects in Design: An Ecological View of Design Artifacts — ResearchGate](https://www.researchgate.net/publication/220580511_Boundary_Objects_in_Design_An_Ecological_View_of_Design_Artifacts)
- [Boundary Objects and their Use in Agile Systems Engineering — arXiv](https://arxiv.org/pdf/1904.12131)
- [Charting Coordination Needs in Large-Scale Agile Organisations with Boundary Objects and Methodological Islands — arXiv](https://arxiv.org/pdf/2005.05747)
- [The Reluctant Engineer: a Love-Hate Relationship with PLM — LinkedIn/Lionel Grealou](https://www.linkedin.com/pulse/reluctant-engineer-love-hate-relationship-plm-lionel-grealou)
- [Why Engineers Don't Use Your PDM/PLM System — and How to Fix It — Hagerman](https://blog.hagerman.com/why-engineers-dont-use-your-pdm/plm-system-and-how-to-fix-it)
- [PDM/PLM Satisfaction Survey — Tech-Clarity](https://tech-clarity.com/documents/Tech-Clarity-Perspective-PLM-User-Satisfaction.pdf)
- [FreeCAD — Market Share, Competitor Insights — 6sense](https://6sense.com/tech/cad-software/freecad-market-share)
- [CAD Software Industry: 2026 Verified Stats — WorldMetrics](https://worldmetrics.org/cad-software-industry-statistics/)
- [LOTAR Initiative: Long-Term CAD Archiving — CAD Interop](https://www.cadinterop.com/en/your-needs/long-term-archival/lotar.html)
- [STEP AP242 Managed Model Based 3D Engineering — Datakit](https://www.datakit.com/en/step_ap242.php)
- [LOTAR the Right Approach — PDES Inc.](https://gpdisonline.com/wp-content/uploads/2023/10/PDES-DavidSelliman-LOTAR-the-Right-Approach-MBSE-Open.pdf)
- [Design Software History: Blender — Novedge](https://novedge.com/blogs/design-news/design-software-history-blender-the-open-source-revolution-in-3d-design-and-its-impact-on-the-creative-industry)
- [Impact of user skills and network effects on the competition between open source and proprietary software — ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S1567422307000051)
- [A Practical Guide to ITAR Compliance for Manufacturers and Engineers — Cofactr](https://www.cofactr.com/articles/a-practical-guide-to-itar-compliance-for-manufacturers-and-engineers)
- [Maintaining ITAR compliance when sourcing custom metal components — Jiga](https://jiga.io/articles/itar-compliance-manufacturing/)
- [Export Control Regulations for CAD Files — SolidBoris](https://solidboris.com/our-blog/tpost/export-controls-cad-files)
- [Common Data Environment (CDE) and ISO 19650: essential for BIM project success — BibLus](https://biblus.accasoftware.com/en/common-data-environment-cde-and-iso-19650-essential-for-bim-project-success/)
- [ISO 19650 Implementation: Structuring Common Data Environments for Large Projects — Adyantrix](https://www.adyantrix.com/blogs/iso-19650-common-data-environments-large-projects)
- [Common Data Environment for University Construction Projects: An ISO 19650-Compliant Framework — MDPI Sustainability](https://www.mdpi.com/2071-1050/18/15/7586)
- [2025 annual report and plans for 2026 — FreeCAD News](https://blog.freecad.org/2026/04/24/2025-annual-report-and-plans-for-2026/)
- [The FreeCAD Grant Program — FPA](https://fpa.freecad.org/programs/fpadf-announcement)
- [Trustworthy Cross-Company Collaboration in Industrial Data Spaces Through Decentralized Authentication and Blockchain Traceability — Springer](https://link.springer.com/chapter/10.1007/978-3-032-07675-5_31)
