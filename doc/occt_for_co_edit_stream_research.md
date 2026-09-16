# OCCT as FreeCAD's Geometry Kernel: Code-Level Analysis and Proposed Changes for co-edit-stream

Purpose: the companion FreeCAD analysis (`freecad_for_co_edit_stream_research.md`) identified three suspected obstacles to co-edit-stream's goals that trace to OpenCASCADE Technology (OCCT), FreeCAD's geometry kernel, rather than to FreeCAD's own App layer — but that analysis was written without OCCT's source available, from how FreeCAD's App layer *calls* OCCT. With a full OCCT clone now available at `./OCCT/`, this document verifies, corrects, and deepens those claims against OCCT's actual code, and proposes concrete, scoped changes to OCCT itself that would make it easier for a FreeCAD-based co-edit-stream implementation to achieve content-addressed, cryptographically verifiable history (goal #3) and stable federated identity for geometry (goal #1). Scope is OCCT as it stands today; the CRDT/collaboration layer (goal #2) and the durable store (goal #3's other half) are covered in the FreeCAD document and the broader `co-edit-stream` research, not repeated here.

All claims below are grounded in a direct read of `src/ModelingAlgorithms/TKBO` (BOPAlgo), `src/ModelingData/TKBRep` (TopoDS/BRep/BinTools/BRepTools), `src/FoundationClasses/TKernel` (OSD/Standard), and `src/ApplicationFramework` (TNaming/TDF) in the checked-out `./OCCT/` tree.

## 1. Correcting the prior analysis: BOPAlgo determinism under `RunParallel=true`

The prior document (correctly) flagged FreeCAD's blanket `SetRunParallel(true)` on every Boolean operation as a risk to canonicalization.
Reading BOPAlgo's actual implementation **narrows, rather than confirms, that risk**:

- `BOPAlgo_Options::SetRunParallel()` (`BOPAlgo_Options.hxx:114-117`) only ever feeds `BOPTools_Parallel::Perform()` (`BOPTools_Parallel.hxx:156-184`),
which parallelizes over a **pre-built, index-addressed vector** (`NCollection_Vector<BOPAlgo_FaceFace>` etc.)
that is always constructed by a strictly sequential loop first (e.g. `BOPAlgo_PaveFiller_6.cxx` ~line 490-529).
Each parallel worker writes into its own fixed-index slot;
nothing is written into a shared unordered container by parallel workers.
The subsequent merge back into `BOPDS_InterfFF`/etc. is a sequential `for` loop over that same fixed index order.
This pattern — sequential fan-out, parallel per-slot compute, sequential fan-in —
recurs identically across all the pairwise-intersection phases (vertex-vertex, vertex-edge, edge-edge, edge-face, vertex-face) and the later builder/splitter phases.
- Final result assembly, `BOPAlgo_Builder::BuildResult()` (`BOPAlgo_Builder_1.cxx:130-168`),
walks `myArguments` (the caller's own input order) and appends each argument's already-sequentially-computed image list into the output compound,
deduplicated through a fence map.
So: **output topology is a deterministic function of input order, given fixed inputs — `RunParallel` changes nothing about it.**

So goal #3's canonicalization work does *not* need to fight arbitrary nondeterminism from BOPAlgo's own parallel execution —
that part of the original concern was **overstated**.
Two genuine risks remain, both narrower and more specific than "parallel Booleans are nondeterministic":

- **A real, documented data race, not just an ordering question.**
`Adaptor3d_Surface.hxx:54-56` states outright:
BSpline surface evaluation caches polynomial coefficients,
and "these evaluations are not thread-safe and parallel evaluations need to be prevented."
If two concurrently-running intersection tasks evaluate adaptors over the *same underlying* `Geom_Surface`
(plausible whenever a solid is placed/copied multiple times and Boolean'd against itself or against another instance sharing geometry)
the shared coefficient cache is mutated without synchronization.
This is not a canonicalization problem —
it is a potential source of genuinely wrong or corrupted numeric results under `RunParallel=true`,
which is strictly worse for a co-edit-stream deployment than reordering,
since it can silently produce different (not just differently-serialized) geometry across runs.

- **Iteration-order leakage in anything built on `TopTools_ShapeMapHasher`.**
`std::hash<TopoDS_Shape>` (`TopoDS_Shape.hxx:329-342`) hashes the `TShape` **pointer address**, not shape content.
This doesn't affect BOPAlgo's own output order
(confirmed deterministic above, since BOPAlgo never emits by iterating such a map),
but it does affect anything downstream that iterates an `NCollection_Map`/`DataMap` keyed this way for its own output —
most notably `BRepTools_History` (see §2), which is exactly what FreeCAD's `ElementMap` and any future co-edit-stream identity layer would consume.

## 2. The Topological Naming Problem: OCCT provides bookkeeping, not identity

`TopoDS_Shape` identity is a plain 3-tuple — `myTShape` (a handle), `myLocation`, `myOrient` (`TopoDS_Shape.hxx:324`) — and `IsSame()`/`IsPartner()` are pointer/handle equality, nothing more (`TopoDS_Shape.hxx:263-271`). There is no content-derived identity anywhere in `TopoDS_Shape` or `TopoDS_TShape`. A rebuild that reconstructs a geometrically identical shape from scratch always produces new, unrelated identity. **This confirms the prior document's claim precisely**, and reading the code sharpens why: OCCT was never designed to answer "is this face the same face as before," only "is this the same C++ object."

Two mechanisms exist that *look* like they might solve this, and reading them closely shows neither does:

- **`BRepTools_History`** (`BRepTools_History.hxx:249-259`) — used internally by `BOPAlgo_BuilderShape::Modified()/Generated()/IsDeleted()` (`BOPAlgo_BuilderShape.hxx:49-77`) — is just two pointer-hashed old-shape→new-shape-list maps that the algorithm itself must explicitly populate as it runs. It is bookkeeping *for one recompute's before/after pairs*, not identity that survives independently of having run that specific algorithm invocation, and not content-derived at all.
- **`TNaming` (OCAF's "Topological Naming" module)**, `src/ApplicationFramework/TKCAF/TNaming/` — despite the name, is the *same idea* relocated into OCAF's document/label tree: `TNaming_NamedShape` (`TNaming_NamedShape.hxx:44-148`) stores an evolution type (`PRIMITIVE/GENERATED/MODIFY/DELETE/REPLACE/SELECTED`) plus old/new shape pairs, populated by explicit `TNaming_Builder` calls from application code. It is exactly as reliable as what the calling algorithm chooses to report — no automatic, content-verified identity. **FreeCAD does not use OCAF or TNaming at all** — its own `App::Document`/`Property` system is entirely independent of it, which confirms (rather than merely assumes, as the prior document had to) that FreeCAD's `ElementMap` was necessarily bolted on externally precisely because nothing in OCCT — including OCCT's own purpose-built naming module — solves this problem at the content level.

**Implication for the proposed architecture**: there is no "just turn on OCAF/TNaming" shortcut available. A stable, federation-worthy sub-shape identity has to be either (a) FreeCAD's existing `ElementMap` approach, kept as-is, or (b) a new content-derived scheme built fresh (geometric-signature hashing per sub-shape: type + parametrization + adjacency, independent of any pointer or algorithm-reported history) — and (b) is new engineering with no OCCT scaffolding to lean on, not an integration task.

## 3. Thread-safety: safe only when shapes don't alias underlying data

`OSD_Parallel`/`OSD_ThreadPool` (`src/FoundationClasses/TKernel/OSD/`) already use a **context-per-thread** pattern (`BOPTools_Parallel.hxx:53-89`, a mutex-guarded map from `Standard_ThreadId` to a per-thread algorithm-context handle) specifically because OCCT's own maintainers know mutable per-call scratch state can't be shared across threads — this is a real, working precedent for how a co-edit-stream integration should structure any future OCCT-touching parallelism, not something to invent from scratch.

`Standard_Transient` reference counting is atomic (`std::atomic_int`, `Standard_Transient.hxx:107-128`), so handle copy/destroy across threads on a shared object is lifetime-safe — but that says nothing about safety of *mutating* that object's other fields. Two concrete shared-mutable-state hazards were found, beyond the BSpline cache in §1:

- `BRep_TFace` (`src/ModelingData/TKBRep/BRep/BRep_TFace.hxx:89-140`) stores the **tessellation cache** (`myTriangulations`, `myActiveTriangulation`) directly on the shared `TShape`-derived object, unguarded by any lock. Any two `TopoDS_Face` values that trace back to the same underlying `TFace` (e.g. two placements of one imported/shared solid — a routine pattern in assemblies) are unsafe to mesh (`BRepMesh_IncrementalMesh`) concurrently.
- More generally: **"safe" in OCCT means "no shared `TShape`/geometry handles across the threads involved," not "safe by shape-value equality."** Two independently-copied (deep-copied via `BRepBuilderAPI_Copy`) shapes are safe to operate on concurrently; two `TopoDS_Shape` values pointing at the same `TShape` (the normal, cheap, shallow-copy case) are not. OCCT provides no detection or enforcement of this — it is caller-responsibility, documented only in scattered header comments, not centrally or defensively checked.

This directly confirms and sharpens the earlier claim in the FreeCAD document that concurrent OCCT execution across a shared process is unsafe in general — with the important refinement that the unsafety is about **shape aliasing**, not blanket "OCCT is single-threaded." A co-edit-stream server that, say, wants to recompute several *unrelated* documents' geometry in parallel worker processes is fine; a single process trying to run two concurrent recomputes against objects that might share underlying geometry (common in any assembly-heavy federation scenario, exactly the case co-edit-stream cares about) is not, without deep-copying first — an added cost/complexity a purely App-layer read of FreeCAD's code could not have surfaced.

## 4. Serialization: structurally deterministic, but not designed to be canonical, and unhashed

No hashing/digest of any kind exists in OCCT's geometry code (`grep -rniE "SHA|MD5|digest|checksum" src` returns nothing relevant — the only hits are unrelated variable names in the deprecated `TopOpeBRep*` module).

The genuinely useful finding: `TopTools_ShapeSet`/`BinTools_ShapeSet` (`TopTools_ShapeSet.hxx`) build their serialized sub-shape index via a **depth-first traversal from the root shape**, deduplicating through a pointer-hashed map but **emitting in DFS insertion order, not hash-bucket order** (`TopTools_ShapeSet.hxx:192`, `NCollection_IndexedMap`). Combined with §1's finding that BOPAlgo's output structure is itself deterministic given fixed inputs, this means: **`BinTools`/`BRepTools` binary/ASCII output is already a deterministic function of a shape's structure for a fixed in-memory object graph.** What it is *not* — and this is the real gap — is canonical across two independently-constructed but semantically-equivalent shapes (e.g., the same solid Boolean'd from arguments supplied in a different order, or reloaded and re-saved), because DFS order still follows whatever order the constructing algorithm happened to build children in, and floating-point values (`BinTools::PutReal`/`PutShortReal`, `BinTools.hxx:35-38`) are written as raw native doubles with no normalization of `-0.0`, no explicit endianness handling, and no NaN canonicalization — a real portability risk for hashing across heterogeneous server/client architectures.

**Net correction to the FreeCAD-layer document**: canonicalization for content-addressing is *not* "fighting kernel-level nondeterminism" (§1 shows there mostly isn't any, modulo the BSpline race) — it is "normalizing a format that's deterministic-per-object-graph into one that's canonical-across-equivalent-object-graphs," a materially easier and more scoped problem than originally framed, but one OCCT's existing serialization does nothing to help with on its own.

## 5. OCAF's `TDF_Delta`/`TDF_Transaction`: not usable scaffolding

`TDF_Delta`/`TDF_Transaction` (`src/ApplicationFramework/TKLCAF/TDF/`) implement a linear, integer-clock-keyed undo/redo log per OCAF document — begin/commit/abort producing a delta of per-label attribute changes. There is no branching, no content hashing, no merge operation, and no cross-process/distributed delta format. Functionally it overlaps with FreeCAD's own `App::Transaction` in *purpose* but is strictly less capable for co-edit-stream's needs (no causal/concurrent-edit model at all) — and since FreeCAD doesn't use OCAF, it isn't even integration-adjacent scaffolding. **This is a dead end for the proposal**: any versioning/CRDT layer has to be built fresh at the FreeCAD App layer or the new server, as the FreeCAD document already proposed; nothing in OCCT should be expected to contribute here.

## 6. License: patching is SaaS-safe, distributing a patched client is not

`LICENSE_LGPL_21.txt` is unmodified LGPL v2.1. `OCCT_LGPL_EXCEPTION.txt` grants one narrow, specific carve-out: *object code* incorporating material from OCCT **header files** (inline/template/handle/collection code compiled directly into a consumer's binary — unavoidable given how header-heavy OCCT's API is) may be distributed under terms of the consumer's choosing, provided attribution is given. This is what lets FreeCAD (and any proprietary application) use OCCT's handle/collection templates without becoming LGPL itself, and without triggering LGPL §6 relinking obligations for that header-derived code specifically.

It does **not** extend to modifications of OCCT's own `.cxx` library source. Ordinary LGPL v2.1 governs the library itself: distributing a *modified* OCCT (source or compiled) to third parties requires making that modified library's source available under LGPL, with §6(a)'s relinking guarantee for anything statically linked. Critically, **LGPL's obligations trigger on distribution, not on network use** (it predates AGPL's network clause) — so a co-edit-stream SaaS backend can fork and patch OCCT (e.g., implementing §7's proposals below) and run it privately on its own servers indefinitely with zero disclosure obligation. The obligation only arises if a **patched OCCT is shipped inside a distributed FreeCAD desktop client** — at which point the patch itself (not the rest of the proprietary stack) must be made available under LGPL to recipients. This cleanly supports an architecture where OCCT patches for content-hashing/canonicalization live primarily in a server-side fork (no disclosure needed) with only the minimal client-side pieces (if any) that must ship in the desktop app kept small and clearly LGPL-labeled.

## 7. Proposed OCCT changes, in priority order

These are scoped patch points, not a full patch — chosen because each is directly load-bearing for the FreeCAD/co-edit-stream architecture's canonicalization and identity requirements (goals #1 and #3), each has a concrete, localized insertion point in real code, and each is ordered here by impact-to-invasiveness ratio.

### 7.1 Canonical-order sort before final compound assembly (highest priority, lowest invasiveness)

**Where**: `BOPAlgo_Builder::BuildResult()`, `src/ModelingAlgorithms/TKBO/BOPAlgo/BOPAlgo_Builder_1.cxx:130-168` (already `virtual`, already overridden by `BOPAlgo_BOP`/`BOPAlgo_Splitter`, so the change point is shared across all Boolean operation types with one edit).

**What**: insert a canonical sort of each argument's image-shape list (by a geometry-derived key — e.g. shape type, then bounding-box-lexicographic, then centroid/area/volume — not insertion order) immediately before the existing `BRep_Builder().Add()` loop.

**Why this, first**: §1 confirmed output is already deterministic *given fixed input order*, but not canonical across equivalent-but-differently-ordered inputs (Boolean'ing A∪B vs B∪A, or re-running a Boolean after a reload where argument order was reconstructed differently). This is exactly what a content hash needs to be order-independent for the same logical result, and the insertion point is a single, already-isolated loop — low risk of breaking anything else in BOPAlgo.

### 7.2 Native content-hash entry point on `TopoDS_Shape`, built on existing serialization traversal

**Where**: `TopoDS_Shape.hxx` (new method, e.g. `ContentHash()`, alongside the existing `std::hash<TopoDS_Shape>` at `TopoDS_Shape.hxx:329-342`), implemented by re-driving `TopTools_ShapeSet`/`BinTools_ShapeSet`'s existing DFS traversal and geometry-writing methods (`WriteGeometry`/`AddGeometry`, `TopTools_ShapeSet.hxx:140-161`) into a streaming cryptographic hash instead of / alongside a byte buffer.

**What**: hash shape type, flags, and geometric parameters recursively through child structure — explicitly excluding the `TShape` pointer address (the existing format already isolates content from pointers per §4, so this is additive, not a rework) — and normalize floating-point representation (fixed endianness, canonical `-0.0`/NaN handling) as part of the same patch, since §4 confirmed the current binary writer does neither.

**Why**: this is the most direct way to get "OCCT can tell you the content hash of a shape natively" rather than relying on an external canonicalization pass in the co-edit-stream server reimplementing shape traversal from scratch. Medium invasiveness — the DFS/geometry-writing extension points already exist and are already used polymorphically for ASCII-vs-binary format selection, so threading a hash accumulator through the same dispatch is a natural, if non-trivial, extension.

### 7.3 Ordered (not pointer-hash-bucket-ordered) `BRepTools_History` containers

**Where**: `src/ModelingData/TKBRep/BRepTools/BRepTools_History.hxx:249-259` — change `myShapeToModified`/`myShapeToGenerated` from `NCollection_DataMap<..., TopTools_ShapeMapHasher>` to `NCollection_IndexedDataMap` (and `myRemoved` similarly to `NCollection_IndexedMap`) — both container types already exist and are already used elsewhere in OCCT (e.g. `TopTools_ShapeSet.hxx:192`), making this a mechanical container-type swap rather than new data-structure design.

**Why**: any full iteration of shape-history (as opposed to point lookups by a known shape) currently leaks pointer-address-dependent order (§1's "iteration-order trap"). Any future code — FreeCAD's `ElementMap`, or a new co-edit-stream identity/diff layer — that dumps or diffs this history for serialization needs reproducible order; this patch removes a source of nondeterminism at negligible risk, since it's a pure container substitution, not a logic change.

### 7.4 Guard the documented BSpline-adaptor cache race

**Where**: `src/ModelingData/TKG3d/Adaptor3d/Adaptor3d_Surface.hxx` and its `GeomAdaptor_Surface` implementation — wherever the polynomial-coefficient cache flagged at `Adaptor3d_Surface.hxx:54-56` is populated/read.

**What**: replace the unsynchronized cache with an immutable-after-first-computation pattern (atomic pointer publish / `std::call_once`), not a plain mutex, since this is an evaluation hot path and lock contention would be a real performance regression.

**Why last, despite being the most serious correctness issue found**: it is the highest-invasiveness item (a hot-path change requiring careful near-zero-cost synchronization design, and it's the one place where getting it wrong risks a performance regression FreeCAD's existing single-threaded users would notice) and it only matters when Boolean/intersection operations run concurrently over shapes that alias geometry — a condition a co-edit-stream deployment could initially avoid operationally (deep-copy before any concurrent OCCT call, per §3) while this patch is developed, rather than blocking on it.

## 8. Summary: what changes from the pre-source (FreeCAD-only) analysis

- **Overstated risk, now narrowed**: BOPAlgo's parallel execution does not itself introduce nondeterminism in output structure (§1) — the canonicalization problem for goal #3 is real but smaller than "fight kernel nondeterminism"; it is "impose canonical ordering on an already-deterministic-per-input-order format" (§4, §7.1).
- **Confirmed and sharpened**: OCCT provides no content-derived sub-shape identity anywhere, including in its own purpose-built OCAF `TNaming` module (§2) — FreeCAD's `ElementMap` is not a stopgap around an oversight, it's a necessary bolt-on FreeCAD had to build because OCCT genuinely has nothing better, even for OCAF-based applications.
- **New, more precise finding**: thread-unsafety is about shape-aliasing (shared `TShape`/geometry handles), not blanket single-threadedness (§3) — a materially more usable constraint for designing a concurrent server architecture than "don't run OCCT concurrently."
- **New, previously unknown finding**: a real, documented, unguarded data race exists in BSpline surface evaluation caching under concurrent use of shared geometry (§1, §7.4) — a correctness risk, not just a determinism/ordering risk, that the FreeCAD-only analysis had no way to discover.
- **New, actionable finding**: OCCT's serialization traversal (`TopTools_ShapeSet`) is structurally reusable as the basis for a native content-hash method (§4, §7.2), meaning the canonicalization work doesn't have to be built as an external re-implementation of OCCT's own shape-walking logic.
- **Confirmed dead end**: OCAF's `TDF_Delta`/`TDF_Transaction` versioning primitives are not useful scaffolding for co-edit-stream's Merkle-DAG/CRDT layer (§5) — nothing lost by ignoring OCAF entirely, consistent with FreeCAD's own choice not to use it.
- **Licensing clarified, not just assumed**: patching OCCT for a server-side co-edit-stream backend carries no disclosure obligation as long as the patched library isn't distributed to third parties (§6) — meaning §7's proposed changes can be developed and run privately with no LGPL compliance burden unless/until a patched OCCT ships inside a distributed FreeCAD desktop build.
