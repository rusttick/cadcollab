# FreeCAD as a Substrate for co-edit-stream: Code-Level Analysis and Proposed Architecture

Purpose: analyze FreeCAD's *actual current source code* (not roadmap claims or wiki descriptions) against the five co-edit-stream goals — (1) composable federated data model, (2) real-time multi-party CRDT collaboration, (3) git-like content-addressed history, (4) continuous PR-like workflow with cascading invalidation, (5) backward-compatible adapters — and propose at least one concrete architecture, assuming a new server/SaaS component is built alongside FreeCAD rather than expecting FreeCAD alone to satisfy these goals. Scope is FreeCAD as it stands today; no other project in this repository's `doc/` tree is considered here.

All claims below are grounded in a direct read of `src/App`, `src/Base`, `src/Gui`, and `src/Mod/Part`/`src/Mod/Assembly` in the checked-out `./FreeCAD/` tree.

## 1. What FreeCAD already has, mapped to each goal

### 1.1 A typed, extensible property/object model (partial answer to goal #1)

`App::PropertyContainer` (base of `Document` and `DocumentObject`) holds a static per-class property table built by macros, plus a `DynamicProperty` layer (`PropertyContainer::addDynamicProperty()`) that lets any object gain new, typed properties at runtime — from Python, with no C++ recompile. Every `Property` subclass implements its own `Save()`/`Restore()`. This is not a generic triple-store, but it is genuinely modular: every workbench (Part, PartDesign, Sketcher, Assembly, …) already adds its own static and dynamic properties onto shared base classes (`DocumentObject`, `GeoFeature`) without touching each other's code. A thin core vocabulary (`part-of`, `connected-to`, `satisfies`, `verified-by`, `supersedes`) is directly implementable as one or two new `Property` subclasses (e.g. a typed `PropertyRelation`) attachable via `addDynamicProperty` to *any* existing object, in any existing document, without modifying a single Mod. This is the single strongest structural fit found in the codebase.

### 1.2 A live, fine-grained dependency graph (strong answer to goal #4, partial answer to goal #1)

Every `DocumentObject` maintains bidirectional `_outList`/`_inList` adjacency, kept incrementally consistent as `PropertyLink`-family properties are set (`_addBackLink`/`_removeBackLink`). `App::DepEdge` models edges at **property** granularity (`{fromObj, fromProp, toObj, toProp}`), not just object granularity, gated behind "fine-grained recompute." `Document::recompute()` topologically sorts this graph (`topologicalSort()`, with an explicit cycle-isolating fallback `partialTopologicalSort()`) and propagates `touch` only to genuinely dependent downstream properties, isolating failures via an in-list-derived skip filter.

This is functionally identical in shape to what goal #4 asks for — "acceptance cascades to invalidate exactly the downstream sign-offs/certifications that depended on what changed" — except FreeCAD does it today for *geometric recompute*, not for compliance/sign-off state. Because the graph is edge-typed and already supports adding new participant types (any new `PropertyLink` subclass joins the same `InList`/`OutList` machinery automatically), a `PropertyVerifiedBy` or `PropertySupersedes` edge type can ride the same recompute/touch propagation that PartDesign features use today, giving cascading invalidation of certifications "for free" at the mechanism level — the missing piece is entirely at the domain layer (what a sign-off object *is*), not the graph engine.

### 1.3 Cross-document links, not yet federation (partial, weak answer to goal #1)

`PropertyXLink` (and `PropertyXLinkSub`/`PropertyXLinkSubList`/`PropertyXLinkContainer`) already let one document's object reference an object living in a *separate* `.FCStd` file, lazily loaded via a `DocInfo` handle, with a parallel document-level in/out-list (`PropertyXLink::getDocumentOutList/InList`) distinct from the intra-document object DAG. The modern Assembly workbench (`src/Mod/Assembly`) is built directly on this — assemblies of independently-saved part files are already FreeCAD's normal working mode, not a hypothetical.

The gap is exactly what separates "cross-referencing files" from "federation across independently-owned graphs": `PropertyXLink` resolves by **file path**, assumes local filesystem access, and has no notion of "this reference is pinned to a specific immutable historical version of the other party's document" — it always resolves to whatever is currently on disk. Every part of the mechanism (the reference type, the lazy-resolution `DocInfo`, the document-level dependency tracking) is reusable; what's missing is a resolver that answers to a content-addressed identifier and a remote fetch instead of a mutable path.

### 1.4 A well-formed but non-cryptographic notion of "commit" (partial answer to goal #3)

`App::Transaction` batches a set of per-object `TransactionObject` records (new/delete/property-change) under one id, is itself serializable, and is explicitly designed (documented in `Transactions.h`) so that transaction ids can span *multiple documents*, letting a cross-document edit set be undone/redone as one atomic unit. This is a real "atomic multi-object commit" primitive already in the code.

It falls short of goal #3 in three concrete ways, confirmed by reading the save path: transaction ids are a locally incrementing counter, not a content hash; transactions live only in a bounded in-memory undo stack (`setUndoLimit`), not a durable append-only log; and — critically — **no cryptographic hashing of document content exists anywhere in `src/App` or `src/Base`** (the only `QCryptographicHash` use is SHA-1 of a *file path* for a temp-directory name, unrelated to content integrity). Serialization itself (`Document.xml` inside the `.FCStd` zip) is not deterministic: self-generated UUIDs, `LastModifiedDate` timestamps, floating-point formatting, and zip entry ordering all vary run to run, so today's save format could not be hashed for content-addressing without a canonicalization pass first.

### 1.5 Geometry stays opaque (informs goal #5 and the boundary of goal #1)

`Part::TopoShape` wraps OpenCASCADE's `TopoDS_Shape` directly; at rest, `PropertyPartShape::Save()` writes the shape as a native OCCT BRep blob referenced by filename from `Document.xml`. FreeCAD's own document format does not expose B-Rep topology as structured, individually addressable graph nodes — the only exposed structure is the `ElementMap` (stable `Face1`/`Edge3`-style names surviving recompute, FreeCAD's answer to the "topological naming problem"). This is the right shape for a federated ontology: geometry is a versioned, content-addressable **leaf blob** reachable via `part-of`/`satisfies` edges, not something to decompose into the core vocabulary itself. Sketcher's 2D constraint geometry is comparatively more structured (typed `Constraint` objects) and is a plausible second, richer example domain submodel if one is wanted beyond solids.

### 1.6 No multi-user support exists, but the observer seams for building it do (central to goal #2)

A direct search of `src/App` and `src/Gui` for CRDT/collaboration/networking/multi-user code found nothing — FreeCAD's `Document` is unambiguously single-process, single-writer, in-memory. There is no concept of two writers touching the same live document.

What *does* already exist, and matters a great deal: `App::Document` exposes roughly 25 `MainThreadSignal`-typed signals covering the entire object lifecycle (`signalNewObject`, `signalChangedObject`, `signalBeforeChangeObject`, `signalTransactionAppend/Remove`, `signalOpenTransaction/CommitTransaction/AbortTransaction`, `signalUndo/Redo`, `signalRecomputedObject`, `signalStartSave/FinishSave`, …), explicitly engineered to marshal onto the main thread "so GUI observers can rely on main-thread delivery even when recompute runs on a worker" — language in the header that indicates FreeCAD's own maintainers have already anticipated async/off-main-thread mutation. Python gets first-class access to exactly this stream via `App.addDocumentObserver()`, which every FreeCAD addon already uses as its standard extension point. Combined with the fact that dynamic properties, transactions, and the full object DAG are all Python-bound (generated `FooPy` bindings throughout `src/App`), this means: **a change-capture and sync layer can be built as an ordinary Python workbench/addon, observing every property write and transaction boundary, with no FreeCAD C++ fork required** for the client side of collaboration. The genuinely new work is everything downstream of capture — translating FreeCAD's last-writer-wins property assignments into CRDT operations, merging concurrent edits, and reconciling merge results back into a single in-process `Document`, none of which the codebase has any notion of today.

### 1.7 Licensing (bears on the SaaS framing)

FreeCAD is LGPL-2.1-or-later (confirmed via `LICENSE` and consistent `SPDX-License-Identifier` headers; third-party code under `src/3rdParty` carries its own separate licenses, not audited here). LGPL permits embedding/linking FreeCAD into a proprietary SaaS backend without requiring the backend to be open-sourced, as long as modifications to FreeCAD's own LGPL source are themselves released and relinkability is preserved. A Python addon/workbench is a separate, dynamically loaded script rather than statically linked object code, which sidesteps LGPL linking questions for the sync layer entirely — another point in favor of the addon-first approach below.

## 2. Summary: fit by goal

| Goal | FreeCAD today | Gap |
|---|---|---|
| #1 Composable federated model | Dynamic properties + PropertyXLink give real building blocks | No content-addressed cross-doc resolution; no provenance/statement model; links are path-based |
| #2 Real-time multi-party CRDT | Rich observer/signal seams, Python-bound everything | Zero concurrency model; single-writer in-memory Document; no CRDT anywhere |
| #3 Content-addressed history | Transaction = well-formed atomic commit already | No hashing, no canonical serialization, no durable/immutable log |
| #4 Continuous cascading invalidation | Fine-grained live DAG + touch propagation is functionally this, today, for recompute | Not generalized past geometric recompute to sign-off/certification semantics |
| #5 Backward-compatible adapters | Opaque geometry blobs + per-property Save/Restore already isolate format concerns | No canonical target representation to adapt *into* yet (that's the new server's job) |

The pattern across all five rows is the same: **FreeCAD's core engine already contains the right-shaped mechanism for four of the five goals** (typed extensible properties, a live dependency DAG with cascading touch, an atomic multi-object transaction primitive, cross-document link resolution) **but each mechanism stops one property short of what co-edit-stream needs** — path-based instead of content-addressed, in-memory instead of durable, single-writer instead of CRDT-converged, geometric-only instead of generalized to compliance/provenance semantics. This argues strongly against a from-scratch reimplementation and for an architecture that keeps FreeCAD's document/DAG/transaction engine as the local, single-user editing core, and adds the missing properties in a new layer around it — the approach the AGENT's own research in this repo's `co-edit-stream` docs already independently converged on (Automerge for CRDT convergence, a content-addressed durable store, canonicalization before hashing) once fitted to FreeCAD-specific integration points instead of a hypothetical blank-slate app.

## 3. Proposed Architecture: "Sidecar Sync Workbench + Content-Addressed Federation Server"

This is one concrete architecture satisfying all five goals; alternatives (a from-scratch web-native re-implementation of FreeCAD's document model; a C++-level fork adding native concurrency) are discussed briefly in §4 as rejected/deferred paths.

### 3.1 Component overview

```
┌─────────────────────────────┐      ┌─────────────────────────────┐
│   FreeCAD Desktop Client A   │      │   FreeCAD Desktop Client B   │
│                              │      │                              │
│  App::Document (unmodified) │      │  App::Document (unmodified) │
│         ▲   │ observer sigs │      │         ▲   │ observer sigs │
│         │   ▼                │      │         │   ▼                │
│  co-edit-stream Workbench    │      │  co-edit-stream Workbench    │
│  (Python addon, new)         │      │  (Python addon, new)         │
│  - DocumentObserver capture  │      │  - DocumentObserver capture  │
│  - property-diff → CRDT op   │      │  - property-diff → CRDT op   │
│  - CRDT op → Document mutate │      │  - CRDT op → Document mutate │
│  - local Automerge replica   │      │  - local Automerge replica   │
└──────────────┬───────────────┘      └──────────────┬───────────────┘
               │  WebSocket (Automerge sync protocol) │
               └───────────────────┬───────────────────┘
                                    ▼
                    ┌───────────────────────────────┐
                    │   co-edit-stream Sync Server    │
                    │   (new; Rust/Node/JVM)          │
                    │  - Automerge repo / relay        │
                    │  - role-based view filtering     │
                    │  - canonicalization on checkpoint│
                    └───────────────┬───────────────────┘
                                    ▼
                    ┌───────────────────────────────┐
                    │  Durable Content-Addressed Store │
                    │  (new; Fluree or Dolt, per        │
                    │   co-edit-stream data-arch doc)   │
                    │  - hashed, immutable commits      │
                    │  - core vocabulary graph           │
                    │    (part-of/connected-to/          │
                    │     satisfies/verified-by/         │
                    │     supersedes)                    │
                    │  - opaque geometry blobs as leaves │
                    │  - format adapters (STEP, AML, …)  │
                    └───────────────────────────────┘
```

### 3.2 Layer-by-layer design, tied to specific FreeCAD mechanisms

**A. Client capture/apply layer — a Python workbench, no C++ fork.**
Register via `App.addDocumentObserver()`; subscribe to `slotChangedObject`, `slotCreatedObject`, `slotDeletedObject`, and transaction boundary callbacks (`slotOpenTransaction`, `slotCommitTransaction`, `slotUndoDocument`/`slotRedoDocument`). Each observed property write is translated into an Automerge CRDT operation keyed by `(objectID or a stable UUID minted at object creation, propertyName)`. `DocumentObject::getID()` is in-document only and not globally stable across federated copies, so the workbench must mint and store a globally-unique id (a new dynamic property, e.g. `App::PropertyUUID GlobalID`, added via `addDynamicProperty` at object-creation time) — this is exactly the "thin core vocabulary property" pattern from §1.1, and doubles as the join key between FreeCAD's local object and the federated graph node. Remote CRDT ops received from the sync server are applied back by calling the same property setters FreeCAD's own UI would call, inside an `openTransaction`/`commitTransaction` pair so they participate in undo/redo like any local edit and trigger the existing recompute/touch cascade (§1.2) automatically — remote edits get correct downstream invalidation for free.

**B. Local CRDT replica — Automerge, per the co-edit-stream data-architecture research.**
Each client's workbench keeps a local Automerge document mirroring the subset of the federated graph relevant to objects open in that session, and uses `automerge-repo`'s sync protocol over WebSocket to converge with peers via the sync server. This directly reuses the reasoning already established for co-edit-stream generally (Automerge chosen over Yjs specifically for content-addressed `ChangeGraph` hashing) — nothing FreeCAD-specific changes that choice; FreeCAD only supplies what gets wrapped (typed property values) and where converged state lands (property setters under a transaction, as above).

**C. Server — sync relay + role-based view + canonicalization boundary.**
The new server is a thin Automerge sync relay/broker for the live layer (this is genuinely new code; FreeCAD has nothing resembling it), plus the point where role-appropriate views are enforced (§ co-edit-stream goal #2's "role-appropriate view over the same underlying data" — e.g. a regulator's session only receives CRDT ops for objects/properties their role is scoped to, filtered before relay, not trusted to the client). At a checkpoint (the moment a user marks a sheet "ready for review," analogous to the WIP→Shared transition already used in this repo's other collaboration research), the server takes the converged Automerge state, runs it through a canonicalization pass, and writes it as a new content-addressed commit into the durable store.

**D. Canonicalization — new work, required regardless of which durable store is chosen.**
Because FreeCAD's own serialization is non-deterministic (§1.4), canonicalization cannot simply reuse `Document::Save()`. It must instead: strip volatile fields (`LastModifiedDate`, self-generated UUIDs not part of the semantic model) from the hash domain, impose a stable sort order over properties and objects (independent of C++ struct layout / insertion order), fix floating-point formatting to a canonical form, and hash the opaque geometry blob (BRep bytes) as an atomic leaf rather than trying to canonicalize OCCT's internal representation. This becomes the concrete implementation of "canonicalization before hashing" that the co-edit-stream data-architecture research flags as a permanent, universally-binding discipline — here scoped precisely to which FreeCAD fields are volatile versus semantic.

**E. Durable store — content-addressed commits form the core vocabulary graph.**
Each canonicalized checkpoint is stored as an immutable, hash-identified commit (Fluree or Dolt, per the existing co-edit-stream durable-layer research — that comparison is not re-litigated here, since it isn't FreeCAD-specific). The core vocabulary relations (`part-of`, `connected-to`, `satisfies`, `verified-by`, `supersedes`) are modeled as edges in this store, populated from two sources: (a) FreeCAD's *existing* `PropertyLink`/`PropertyXLink`-derived object graph, harvested directly at checkpoint time (an `App::Link` becomes a `part-of` edge; a `PropertyXLink` between documents becomes a `connected-to` edge crossing a federation boundary), and (b) new relation types (`verified-by`, `supersedes`) written by the small new `PropertyRelation` type from §1.1, which any workbench — including third-party ones — can attach to any object without further core changes. A regulator's attestation or a vendor's datasheet claim is stored as a Wikidata-style statement-with-provenance attached to a node, per the existing co-edit-stream architecture research, rather than overwriting a shared field — directly resolving FreeCAD's current single-writer-property limitation at the federation layer instead of trying to solve it inside `Document` itself.

**F. Cascading invalidation of sign-offs — reusing FreeCAD's live DAG mechanism, generalized.**
A certification/sign-off object is modeled as an ordinary lightweight `DocumentObject` (or a server-side graph node with no FreeCAD-local representation at all, for sign-offs that don't need desktop editing) linked to what it certifies via `verified-by`/`satisfies` edges. Because these edges ride the same `PropertyLink` family described in §1.2, any change to the certified object automatically marks the sign-off object as touched through FreeCAD's existing recompute/touch propagation — no new dependency-tracking engine needs to be built for the *desktop-open* case. For sign-offs tracked only server-side (not open in any live FreeCAD session), the server replicates the same touch-propagation logic against the durable graph's edge set at commit time — same semantics, executed server-side because no client has that document open.

**G. Backward compatibility — adapters target the canonical representation, not FreeCAD's native format.**
FreeCAD's per-property `Save()`/`Restore()` polymorphism (§1.4) already isolates "how does this one property type serialize" from the rest of the document — the same seam an AutomationML or STEP import/export adapter uses today. A co-edit-stream adapter for a legacy format runs entirely server-side (or as an offline batch job): parse the legacy file, populate the canonical core-vocabulary graph plus opaque-blob leaves exactly as a checkpoint from a live FreeCAD session would, and commit it into the same content-addressed store — indistinguishable in kind from natively-authored history once committed, matching the "no hard cutover" requirement in the co-edit-stream README.

### 3.3 What is genuinely new work versus reused

**Reused as-is from FreeCAD:** the property/object model and its dynamic-property extensibility; the live property-level dependency DAG and touch/recompute cascade; the transaction abstraction's atomicity and multi-document scope; `PropertyXLink`'s cross-document reference *shape* (not its path-based resolution); the Python observer/binding surface as the entire client integration point; per-property `Save`/`Restore` as the adapter seam.

**New, and squarely outside FreeCAD's current codebase:** the CRDT layer and its translation to/from FreeCAD property writes; the sync relay server and role-based view filtering; canonicalization of FreeCAD's serialized state; all content hashing and the durable content-addressed store itself; resolving `PropertyXLink`-style references against content-addressed identifiers instead of file paths (a client-side resolver change, but still addon-level, not core); the statement/qualifier/provenance model for multi-party facts (server-side graph feature, not a FreeCAD concept at all today); generalizing touch-cascade semantics to server-only (not-desktop-open) sign-off nodes.

### 3.4 Concurrency caveat carried over honestly

FreeCAD's `Document` remains single-writer in-process even in this architecture — the workbench applies remote CRDT-converged changes back into the local `Document` serially, on the main thread, the same way a human's own edits are applied. This means two users editing *the same object open in two different desktop sessions* converge correctly (Automerge guarantees strong eventual consistency across replicas), but a single desktop session never has two writers touching its in-memory `Document` concurrently — convergence happens between sessions, not within one. This matches how every other CRDT-backed desktop-app integration works (e.g., Automerge-backed native apps generally), and requires no invasive change to `Document`'s threading model, but should be stated as a real constraint rather than implied away: it is not "true" multi-writer access to one process's document, it is convergent multi-writer access to the *federated document*, mediated through each participant's own local, still-single-writer FreeCAD instance.

## 4. Alternatives briefly considered and why they're not the primary recommendation

- **Native C++ concurrency inside `Document`** (making `Document` itself accept concurrent writers, replacing the undo `Transaction` stack with a hash-chained log at the C++ level): technically the most "native" fit for goal #3's content-addressing and would remove the canonicalization/translation layer's overhead, but requires a genuine FreeCAD fork with all the maintenance-burden and upstreaming risk that implies, versus an addon that tracks upstream releases loosely. Worth revisiting only if the Python-addon sync layer proves to be a measured performance bottleneck on large documents (the same caveat the co-edit-stream data-architecture research already raises about Automerge itself at scale) — not something to assume up front.
- **Ignore FreeCAD's native format entirely and build a web-native document model from scratch**, treating FreeCAD only as an import/export target: discards §1.1–§1.3's genuinely reusable mechanisms (extensible properties, live DAG, transaction atomicity, cross-doc links) for no clear benefit, and abandons FreeCAD's existing desktop-user base and OCCT geometry investment. Rejected as strictly worse than the sidecar approach unless the goal shifts to "a new CAD tool," which is outside this analysis's brief (FreeCAD as it stands today).

## 5. Open questions carried forward

- Minting a stable `GlobalID` per object (§3.2.A) needs a policy for objects that already exist in years of legacy `.FCStd` files with no such property — a one-time migration/backfill adapter, not a blocker, but real work.
- `PropertyXLink`'s resolution-by-path (§1.3) needs a concrete replacement resolver design (content-addressed id → fetch from durable store → materialize a local cached copy for OCCT to load) before cross-document federation can work offline/disconnected — not designed here, flagged as the first substantive follow-on design task.
- Fine-grained recompute (`DepEdge`, §1.2) is described in the code as an optional/gated mode (`Application::isFineGrainedRecomputeEnabled()`) — confirming its production maturity and performance characteristics at the property-edge granularity this architecture leans on is a prerequisite validation step, not an assumption to carry forward untested.
