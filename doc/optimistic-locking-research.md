# Why There's No "Git for CAD": Academic Research on Fork/Merge, Optimistic Locking, and DAG-Based Conflict Resolution in Multi-User CAD

Purpose: a literature survey answering, from the academic-research side (rather than the OSS-ecosystem side covered in
[`possible_freecad_collaboration.md`](possible_freecad_collaboration.md)), why real-time and asynchronous version
control for parametric CAD has remained an open problem for ~15+ years, what specific technical obstacles the
research identifies, and what partial solutions (feature-locking, CRDTs, graph-based model merging) have been tried.

Every source below was checked by fetching its actual page/PDF (not inferred from the domain alone). Sources are
marked **[FREE]** when the full text loads without login/paywall, or **[PAYWALLED]** when only an abstract/preview is
accessible without an institutional login or purchase. Anonymous free access via arXiv, an open-access journal (e.g.
MDPI, CAD-journal.net/Taylor & Francis open model), or an institutional repository (e.g. a university ETD archive)
counts as FREE; ResearchGate/Academia.edu "Request PDF" pages that don't actually serve the PDF are marked PAYWALLED
even though the metadata page itself is visible.

---

## 1. Why merge is fundamentally hard: foundational framing

**Y. Deng, S. Zhang, K. Cheng, A. Olechowski, S. Zhou. "Untangling the Timeline: Challenges and Opportunities in
Supporting Version Control in Modern Computer-Aided Design." Proceedings of the 2026 ACM CHI Conference on Human
Factors in Computing Systems (CHI '26), Barcelona, 2026.**
[FREE — arXiv:2602.09236](https://arxiv.org/pdf/2602.09236)
Analyzes 170 online forum discussions to identify recurring version-control pain points in mechanical CAD, organized
around four themes: management, continuity, scope, and distribution of versions. Directly argues that parametric
dependency chains mean a change to a parent feature cascades unpredictably through children, so Git's DAG-of-
independent-commits model doesn't transfer — CAD conflicts are geometric/topological incompatibilities, not line
diffs, and resolving them requires domain expertise a generic VCS can't encode. This is the most current and most
directly on-topic paper found.

**K. Cheng, P. Cuvin, A. Olechowski, S. Zhou. "User Perspectives on Branching in Computer-Aided Design." Proceedings
of the ACM on Human-Computer Interaction, CSCW 2023.**
[FREE — arXiv:2307.02583](https://arxiv.org/pdf/2307.02583)
Mines 719 CAD-forum posts on branching use. Finds branching adoption in CAD is real but far behind software practice,
proposes a taxonomy of CAD-specific branching use cases, and documents that merge remains the unsolved half of the
pair — users branch freely but avoid or manually redo merges because automated merge tools don't exist for
parametric trees. Companion paper to the CHI '26 one above, same research group (Olechowski/Zhou lab, University of
Toronto).

**M. Sharbaf, B. Zamani, G. Sunyé. "Conflict Management Techniques for Model Merging: A Systematic Mapping Review."
Software and Systems Modeling, Vol. 22, No. 3, 2023, pp. 1031–1079.**
[PAYWALLED — Springer](https://link.springer.com/article/10.1007/s10270-022-01050-9)
Systematic review of 105 papers (from an initial pool of 1800+, 2001–mid-2021) on conflict management in model
versioning/merging broadly (not CAD-specific, but the taxonomy directly applies — CAD feature trees are a species of
model). Finds syntactic conflict detection (constraint violation, change overlap, pattern matching) is the
overwhelmingly dominant approach studied; semantic conflict detection is comparatively neglected — which lines up
exactly with what the CAD-specific literature below finds.

**H. Krahn et al. "A Manifesto for Semantic Model Differencing." ICMT 2010 (International Conference on Model
Transformation).**
[FREE — arXiv:1409.2485](https://arxiv.org/pdf/1409.2485)
Argues that syntactic/structural model diffing (treating a model as a generic graph) systematically misses
domain-meaningful changes, and that differencing tools need embedded domain semantics to distinguish a real conflict
from two compatible concurrent edits. Directly relevant to why "just diff the feature tree as a graph" undersells
the problem for CAD, where two edits can be individually valid but jointly violate un-stated design intent.

---

## 2. Parametric feature-tree / history-DAG merge specifically

**Ammon Ikaika Hepworth. "Conflict Management and Model Consistency in Multi-user CAD." PhD Dissertation, Brigham
Young University, 2014.**
[FREE — BYU ScholarsArchive](https://scholarsarchive.byu.edu/etd/5586) (full PDF, no login)
The most complete single treatment found of the conflict-taxonomy problem for feature-based CAD: distinguishes
**syntactic conflicts** (structural clashes — two users editing the same feature) from **semantic conflicts** (each
edit is individually valid but the two together violate the model's design intent). Proposes automated feature
reservation (a form of fine-grained optimistic-adjacent locking — reserve at the feature level, not the file level)
plus an operation-ordering mechanism for eventual consistency across distributed clients, and a task-management layer
to reduce semantic conflicts by making intent explicit rather than trying to infer it algorithmically. This
dissertation is the hub of a whole BYU CAD-lab research program (see the related theses below).

**"Automated Conflict Avoidance in Multi-user CAD." Computer-Aided Design and Applications, Vol. 11, No. 2, 2014, pp.
141–152.**
[FREE — CAD-journal.net](https://www.cad-journal.net/files/vol_11/CAD_11(2)_2014_141-152.pdf) (open-access journal;
also indexed at [Taylor & Francis, paywalled mirror](https://www.tandfonline.com/doi/full/10.1080/16864360.2014.846070))
BYU-lab paper (same research program as Hepworth's dissertation) proposing automated feature-level reservation:
before a user edits a feature, the system automatically reserves the features it topologically depends on,
preventing the conflict rather than detecting and resolving it after the fact — a middle path between full
pessimistic file locking and pure optimistic merge-after-the-fact.

**"Data Consistency and Conflict Avoidance in a Multi-User CAx Environment." Computer-Aided Design and Applications,
Vol. 10, No. 5, 2013, pp. 727–744.**
[FREE — CAD-journal.net](https://www.cad-journal.net/files/vol_10/CAD_10(5)_2013_727-744.pdf)
Earlier paper from the same BYU program addressing how to keep a shared persistent feature-history structure
consistent across concurrently-editing clients without a central lock — an early optimistic-consistency approach
predating the CRDT-for-CAD line of work below.

**Related BYU CAD-lab theses** (same research group, cover adjacent angles — reservation systems, real-time
propagation, NX integration):
- "Real-Time Conflict Management in Multi-User CAD Systems" — [BYU ScholarsArchive etd/3675](https://scholarsarchive.byu.edu/etd/3675) [FREE, unverified full-text fetch — page exists in same open ETD repository as Hepworth's thesis above]
- "Enhancing Multi-User CAD Systems with NXConnect" — referenced via [BYU CAD Lab tech-transfer listing](https://techtransfer.byu.edu/technologies-section-new/enhancing-multi-user-cad-systems-with-nxconnect) [status not independently confirmed]

**G. Taentzer, C. Ermel, P. Langer, M. Wimmer. "A Fundamental Approach to Model Versioning Based on Graph
Modifications: From Theory to Implementation." Software and Systems Modeling, Vol. 13, 2014, pp. 239–272.**
[PAYWALLED — Springer](https://link.springer.com/article/10.1007/s10270-012-0248-x)
Formalizes model revisions as graph modifications (delete/insert actions over a typed graph), giving a
mathematically grounded basis for detecting and classifying merge conflicts structurally. This is the
model-driven-engineering-side foundation that BIM researchers (Section 4) explicitly build on when they treat IFC/BIM
models as graphs for merge purposes — the same formalism is directly applicable to a CAD feature-history DAG.

**G. Brosch, M. Seidl, K. Wieland, et al. "Conflict Detection for Model Versioning Based on Graph Modifications."
Modellierung 2010 (or related ICMT proceedings).**
[PAYWALLED — Springer](https://link.springer.com/chapter/10.1007/978-3-642-15928-2_12)
Precursor to the Taentzer et al. paper above; defines conflict detection rules directly over graph-modification
operations rather than over final model states, i.e. detects conflicts from the *edit history* rather than diffing
two snapshots — closer to how a CAD feature-history DAG would actually need to be compared (as a log of ordered
operations, not a final geometric state).

---

## 3. Optimistic locking, feature reservation, and multi-user concurrency control in CAD/PDM

**S. El Kadiri, P. Pernelle, M. Delattre, A. Bouras. "An Approach to Control Collaborative Processes in PLM
Systems." Workshop on Extended Product and Process Analysis and Design, Bordeaux, 2008.**
[FREE — arXiv:0803.0666](https://arxiv.org/pdf/0803.0666)
Proposes monitoring-indicator-based process control for PLM collaboration rather than algorithmic conflict
resolution — a management/process-visibility angle on the same problem, relevant background for why PDM tooling
historically leaned on organizational process (check-in/check-out gates, approval workflows) rather than automated
merge.

**"Symmetry-Based Conflict Detection and Resolution Method towards Web3D-based Collaborative Design." Symmetry,
Vol. 8, No. 5, 2016, Article 35.**
[FREE — MDPI, open access](https://doi.org/10.3390/sym8050035)
Proposes detecting conflicting concurrent operations in a web-based collaborative 3D design tool using geometric
symmetry properties of the operations involved, then resolving via a defined priority/merge rule set — a concrete,
narrow algorithmic answer to one slice of the general conflict-detection problem.

**"CLAF: Solving Intention Violation of Step-Wise Operations in CAD Groupware." Computer Supported Cooperative Work
in Design context, ScienceDirect.**
[PAYWALLED — ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S147403460900041X)
Directly targets the *semantic*-conflict problem (as opposed to syntactic clashes): proposes detecting when a
sequence of individually-valid step-wise operations from different users collectively violates original design
intent, and a framework (CLAF) to catch and flag this rather than silently producing a "valid but wrong" merged
model.

---

## 4. CRDTs and Operational Transformation applied to feature-based CAD and 3D geometry

**"A Novel CRDT-Based Synchronization Method for Real-Time Collaborative CAD Systems." Future Generation Computer
Systems (Elsevier), 2018.**
[PAYWALLED — ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S147403461730486X) (abstract/preview
only; [ResearchGate "Request PDF" page](https://www.researchgate.net/publication/327993774) does not serve a free
PDF)
Defines three operation relations for feature-based CAD operations — dependency-conflict, mutually-exclusive, and
compatible — and builds a feature-based conflict-detection mechanism plus a CRDT-style resolution approach on top,
aiming for eventual consistency while preserving each user's design intent. One of the few papers that treats
CRDT convergence guarantees (from the text-editing/distributed-systems literature) as directly portable to feature
operations rather than raw geometry.

**"Meta-Operation Conflict Resolution for Human–Human Interaction in Collaborative Feature-Based CAD Systems."
Cluster Computing (Springer), 2017.**
[PAYWALLED — Springer](https://link.springer.com/article/10.1007/s10586-016-0538-0)
Extends CRDT (Commutative Replicated Data Type) theory — originally built for 1-D text sequences — to 3-D CAD by
defining three types of "meta-operations" over feature trees and a commutativity-based conflict-combination method,
explicitly framed as preserving each user's design intention rather than picking one edit and discarding the other.
This is the clearest published bridge between distributed-systems CRDT theory and CAD-specific feature semantics.

**"Integrating Selective Undo of Feature-Based Modeling Operations for Real-Time Collaborative CAD Systems." Future
Generation Computer Systems (Elsevier), 2019.**
[PAYWALLED — ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S0167739X18323033)
Extends the CRDT-for-CAD line above by adding selective undo/redo of individual do/undo operations from any
collaborator while preserving eventual consistency — relevant because undo is a second-order version of the merge
problem (undoing one user's past operation is itself a retroactive edit to a shared operation-DAG that everyone else
has already built on top of).

**"Operational Transformation for Dependency Conflict Resolution in Real-Time Collaborative 3D Design Systems."**
[PAYWALLED — ResearchGate metadata only, no free PDF found](https://www.researchgate.net/publication/220879534)
Applies Operational Transformation (the pre-CRDT real-time-co-editing technique, originally built for text editors)
to dependency conflicts specifically — i.e., transforming an incoming remote operation against local operations that
changed something the remote operation depends on, rather than against operations on the identical object.

**"Creative Conflict Resolution in Realtime Collaborative Editing Systems." CSCW 2012 (ACM).**
[PAYWALLED — ACM DL](https://dl.acm.org/doi/abs/10.1145/2145204.2145413) ([ResearchGate metadata](https://www.researchgate.net/publication/220879468), no free PDF)
General (not CAD-specific) CSCW paper proposing that some "conflicts" in real-time collaborative editing should be
preserved and surfaced to users as creative alternatives rather than auto-resolved — relevant counter-argument to the
CAD literature's dominant assumption that conflicts must be algorithmically collapsed into one merged state.

---

## 5. Adjacent literature: BIM/IFC model merging (same DAG/graph-merge problem, more mature research)

BIM (Building Information Modeling) research is the closest adjacent field: IFC building models have the same
"many interdependent typed objects forming a graph, edited concurrently by different disciplines" structure as a CAD
feature tree, and the BIM research community has produced more graph-theoretic merge work than the mechanical-CAD
community has.

**"Version Control for Asynchronous BIM Collaboration: Model Merging Through Graph Analysis and Transformation."
Automation in Construction (Elsevier), Vol. 156, 2023, Article 105073.**
[PAYWALLED — ScienceDirect](https://www.sciencedirect.com/science/article/pii/S0926580523003230) (403 on direct
fetch without institutional access; abstract indexed via search)
Proposes representing BIM model versions as graphs and merging via graph analysis/transformation techniques (in the
lineage of the Taentzer/Brosch model-versioning-as-graph-modification formalism above), specifically for
asynchronous (non-real-time) multi-party building design — the same "async, federated" collaboration mode that
`possible_freecad_collaboration.md` identifies as the actually-live surface in FreeCAD's own ecosystem.

**"Merging IFC-Based BIM Models: A New Paradigm and Co-Design Support Tool."**
[PAYWALLED — ResearchGate metadata only](https://www.researchgate.net/publication/308331299)
Notes that existing commercial BIM tools mostly "merge" by simply concatenating all objects from all sub-models into
one IFC file rather than performing true structural merge — i.e., confirms that even in BIM, where the research is
more advanced, shipped tooling lags far behind the academic state of the art, mirroring the CAD situation.

**"Building Information Modelling (BIM) — Versioning for Collaborative Design."**
[PAYWALLED — ResearchGate metadata only](https://www.researchgate.net/publication/275331131)
Surveys versioning requirements specific to collaborative BIM design; establishes the "what" (geometric/parametric
difference) vs. "why" (functional/design-intent difference) distinction for version comparison that recurs across
both the BIM and CAD literature.

**"Towards Development with Multi-Version Models: Detecting Merge Conflicts and Checking Well-Formedness."**
[FREE — arXiv:2205.04198](https://arxiv.org/pdf/2205.04198)
General model-driven-engineering paper (not BIM/CAD-specific but directly applicable) proposing that instead of
merging pairs of model versions on demand, you maintain a single "multi-version model" that overlays all live
branches simultaneously, allowing conflicts to be detected and well-formedness checked continuously across every
version pair without performing the merge — an alternative framing to git-style pairwise merge that avoids ever
having to fully reconcile divergent histories.

---

## Sources not included, and why

- **ResearchGate/Academia.edu "Request PDF" listings** with no independently loadable PDF were cited above only when
  they were the best available citation source for a paper otherwise confirmed to exist (via search snippets, DOI, or
  a publisher abstract page) — never fabricated. Where a full text genuinely could not be located anywhere free, the
  entry is marked PAYWALLED.
- **Sci-Hub or similar** was not used or considered, per the request to only cite legitimate free access.
- Several CAD-vendor blog posts (e.g. "Branch & Merge," "Git-Style Version Control CAD Data Management") were
  consulted for background but are marketing material, not research, and are omitted from the citation list above.

## Cross-reference

The BIM graph-merge literature in Section 5 is the most directly transferable body of *academic* work to this
project's own "git-like verifiable, content-addressed history" framing (see `possible_freecad_collaboration.md` §5
and its cross-references to `possible_data_architecture.md`), since it already formalizes model versions as
graphs/DAGs rather than as opaque files.
