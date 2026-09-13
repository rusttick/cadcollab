# doc/ — FreeCAD Collaboration Research

This directory holds this project's research into collaborative/multi-engineer
design in FreeCAD (and, where relevant, its OpenCascade geometry kernel),
spanning several documents that grew organically as scope broadened. This
README is the map — read it first; it tells you which document actually has
the answer to a given question. (Renamed from
`freecad_collaboration_research_index.md` so it's discoverable as the
directory's standard entry point.)

## Reading order / map

1. **[`possible_freecad_collaboration.md`](possible_freecad_collaboration.md)**
   — Start here. The original survey: what's live, what's dead, and where the
   unclaimed high-leverage work is in the FreeCAD collaboration/PDM space, as
   of the first research pass. Names the #25681 issue cluster and three named
   PDM attempts (Lens, FreePDM, nanoPLM) as the entry points that everything
   else below grew out of.

2. **[`freecad_core_and_lens_collaboration_issues.md`](freecad_core_and_lens_collaboration_issues.md)**
   — The deep, verbatim dive into **FreeCAD's own official-channel**
   collaboration design effort: the full FreeCAD/FreeCAD#25681 issue cluster
   (#25681, #25682, #25685, #23248), Ondsel-Server#47/#48, the identified
   history of the "old GSoC" BCF plugin (Patrick Podest, GSoC 2019), and the
   complete FEP-0011 "Generic Collaboration Framework" story (the FEP text,
   its implementation PR #26306, its GitHub Discussion #40 in full, and
   developer-meeting-minutes mentions through 2026-09). Also covers
   pieterhijma's adjacent versioning work (PR #28312, FEP-0013 Visual Diffs).
   **Read this for**: what FreeCAD itself is actually doing, who's driving it,
   and where that effort stands right now.

3. **[`ondsel_server_issue_48.md`](ondsel_server_issue_48.md)** — Just the raw
   text of Ondsel-Server issue #48 ("Extent to which Lens is a PDM") on its
   own, kept as a standalone quick-reference (also embedded in full inside
   document 2 above).

4. **[`ondsel_48_research.md`](ondsel_48_research.md)** — Standards-side deep
   dive: reads issue #48 closely and maps every gap the Lens/Ondsel-Server
   maintainers identified onto already-existing, already-standardized prior
   art (VDI 2770, AAS Handover Documentation, AAS Hierarchical
   Structures/BoM, AAS type/instance split, BCF) that they appear unaware of.
   **Read this for**: which international standard already answers which part
   of Lens's stated PDM gap.

5. **[`ondsel_48_response.md`](ondsel_48_response.md)** — The actual comment
   text posted to Ondsel-Server#48 distilling document 4's findings (as of
   this research, still awaiting a reply from the Lens maintainers).

6. **[`ondsel_project_proposal_research.md`](ondsel_project_proposal_research.md)**
   — Assuming the optimistic outcome of document 5 (Lens adopts the standards
   mapping), this is the staged, branch-by-branch implementation-feasibility
   plan for actually building it into Lens's real Node/Mongo codebase without
   a big-bang rewrite. **Read this for**: a concrete "how would this actually
   get built" roadmap, grounded in Lens's real schema.

7. **[`optimistic_locking_research.md`](optimistic_locking_research.md)** —
   An academic-literature survey (not FreeCAD-specific) on why real-time and
   asynchronous version control for parametric CAD has remained a genuinely
   open research problem for 15+ years: syntactic vs. semantic conflicts,
   feature-level locking, CRDTs applied to feature trees, and the more mature
   adjacent BIM/IFC graph-merge literature. **Read this for**: the theoretical
   grounding behind why "just add git" doesn't trivially work for CAD.

8. **[`freecad_pdm_plm_ecosystem_census.md`](freecad_pdm_plm_ecosystem_census.md)**
   — The big one, and the one that keeps growing. An exhaustive,
   continuously-expanding catalog of **every independent third-party effort,
   anywhere, in any language**, that has shown any interest in collaborative
   design in/with FreeCAD or OpenCascade — dedicated PDM/PLM systems, version
   control addons, BOM tooling, cloud CAD sharing, ERP/PLM platform
   integrations, academic connectors, browser/WASM ports, an international
   (German/French/Spanish/Italian/Dutch/Portuguese/Russian/Polish/Czech/
   Ukrainian/Chinese/Japanese/Korean) language sweep, the OpenCascade-specific
   ecosystem independent of FreeCAD branding, an English terminology sweep
   (concurrent engineering, MBSE, EDM, ECO, etc.), and — its richest single
   source — the complete 156-post/3-year forum design journal behind
   FreePDM (§16). Currently sits at ~27 distinct projects/efforts
   found across at least eleven countries and still counting; the document's
   own later sections track how fragmented and mutually-unaware this
   ecosystem is — right up to space-agency scale (ESA's COMET). **Read this
   for**: "has anyone already built X, anywhere, in any language?" — check
   here before assuming something is unclaimed.

9. **[`ecosystem_relationship_mapping_plan.md`](ecosystem_relationship_mapping_plan.md)**
   — Not a findings document — a methodology and staged plan for the *next*
   phase of this research: mapping the conceptual relationships and overlap
   between everything catalogued in document 8, rather than just listing
   projects. Recommends a node/facet/typed-edge data model (with a worked
   example using real pieterhijma/FreePDM/BCF data), a file format
   (`doc/ecosystem_graph.yaml`, not yet created), and a five-stage plan
   (pilot → taxonomy freeze → full extraction → focused visualizations →
   synthesis doc). **Read this for**: where to start if you're picking up
   the relationship-mapping work, before any of that data actually exists.

## How this index itself should be maintained

Whenever a new document is added to this research area, or an existing one is
renamed, this index should be updated in the same edit — it is the one place
a future session (or a future person) can start cold and find everything
without re-discovering the file layout from scratch.
