# FreeCAD/OpenCascade PDM, PLM & Collaboration Ecosystem Census

Purpose: an exhaustive, continuously-growing census of **every independent
project, anywhere, in any language, that has ever shown any interest in
groups of engineers collaborating on design in or with FreeCAD or its
OpenCascade geometry kernel** — PDM, PLM, version control, BOM/parts-library
management, cloud CAD sharing, real-time multi-user editing, academic
connectors, and anything else that doesn't fit a narrower label. This is
**not** about FreeCAD's own official collaboration effort (the #25681 issue
cluster and FEP-0011) — that's covered in full, verbatim, in
[`freecad_core_and_lens_collaboration_issues.md`](freecad_core_and_lens_collaboration_issues.md).
This file is the outside-in view: everyone else who's ever tried.

Section numbering starts at 10 rather than 1 because this file was split out
of a larger combined document (`freecad_collaboration_ecosystem_research.md`,
now retired) once it grew too large and topically distinct to stay merged
with the official-channel issue research — the original section numbers were
kept as-is rather than renumbered, to avoid breaking the many internal
cross-references (`§10.9`, `§11.3`, etc.) already written throughout this
document and referenced from others. New sections continue the same numbering
sequence (12, 13, ...) as the census keeps growing. An index tying together
all of this project's FreeCAD-collaboration research is at
[`README.md`](README.md).

Fetched/compiled across multiple research passes, first on 2026-09-13 and
continuing.

## 10.0 Original scope note (superseded by later sections — kept for history)

The original survey (`possible_freecad_collaboration.md`) named three independent PDM
attempts (Lens, FreePDM, nanoPLM) plus FEP-0011's motivation section named three more
(CADBaseLibrary, Taack FreeCAD PLM, Ondsel-Lens-Addon). Framing the search around "PDM"
specifically was too narrow — a broader sweep (PLM, version control, git integration,
BOM management, cloud CAD sharing, real-time collaboration, teamwork) surfaced roughly
**25+ independent projects**, most never cross-referencing each other, spanning 2013 to
literally days before this fetch (2026-09-13). This section is the exhaustive record.
Search method: GitHub REST repository-search API (`api.github.com/search/repositories`)
across keyword combinations (`freecad+pdm`, `freecad+plm`, `freecad+versioning`,
`freecad+collaboration`, `freecad+teamwork`), the official Addon Manager catalog
(`FreeCAD/FreeCAD-addons`), and targeted web search for named projects and their prior
art. All stats (stars, dates, license) pulled live from the GitHub API on the fetch date
unless noted otherwise.

## 10.1 Dedicated PDM/PLM systems/addons targeting FreeCAD (standalone or in-app)

| Project | Description (from repo) | Stars | Created | Last push | License | Status note |
|---|---|---|---|---|---|---|
| [FreeCAD/Ondsel-Server](https://github.com/FreeCAD/Ondsel-Server) ("Lens") | Community-owned PDM/collaboration server, ex-Ondsel-the-company | — | — | active | AGPL-3.0 | Covered in full in sections 1–9 above |
| [grd/FreePDM](https://github.com/grd/FreePDM) | "A PDM for FreeCAD" (Go) | 81 | — | 2025-10-24 | MIT | Already known from original survey. **Full founding history documented in §16**: a 156-post, ~3-year (2022-04 to 2025-02) forum design journal, author "grd"/Gerard, presented at an actual "FreeCAD Day" event in 2025 |
| [alekssadowski95/nanoPLM](https://github.com/alekssadowski95/nanoPLM) | "Open-Source PLM for small machine manufacturers - natively supports FreeCAD" | 72 | — | 2025-07-21 | — | Already known; hub of a larger personal cluster, see §10.9 |
| [nerd-sniped/GitPDM](https://github.com/nerd-sniped/GitPDM) | "An Open Source Git Based PDM Addon For FreeCAD" — commit/push/pull `.FCStd` from inside FreeCAD, multi-host (GitHub/GitLab/Bitbucket/Gitea-Forgejo/SourceHut) | 37 | 2025-12-26 | 2026-07-26 | MIT | **New find.** Real, documented, packaged as a proper FreeCAD addon with releases; actively developed within the last ~2 months of the fetch date |
| [pawelcel/EasyPDM](https://github.com/pawelcel/EasyPDM) | "Self-hosted, open-source PDM system for CAD files — numbering, revisions, BOM, and SolidWorks/FreeCAD integration for small engineering teams" | 5 | 2026-08-04 | 2026-09-09 (4 days before this fetch) | MIT | **New find, extremely current.** Notably: the README states the author (a mechanical design engineer, not a programmer) had the entire application "written for me by Claude (an AI model from Anthropic)" based on their requirements. Features: unique numbering, revisions, item locking/check-out, BOM export, shared material/manufacturer catalogs, notifications |
| [alekssadowski95/we-have-PDM-at-home](https://github.com/alekssadowski95/we-have-PDM-at-home) | "An open-source workflow for product data management, comprised of FreeCAD, LibreOffice and subversion (VisualSVN Server and Tortoise SVN)" | 3 | 2025-07-26 | 2025-07-26 | MIT | **New find.** A deliberately low-tech, SVN-based workflow rather than a purpose-built app — the "PDM at home" framing is explicit about being a pragmatic assembly of existing tools, not new software |
| [pistock-org/PiStock](https://github.com/pistock-org/PiStock) | "A lightweight, self-hosted PLM and inventory management system running on Raspberry Pi, featuring FreeCAD integration and web-based 3D visualization" | 12 | 2026-05-25 | 2026-07-04 | AGPL-3.0 | **New find.** Targets hobbyist/small-shop hardware (Raspberry Pi) rather than enterprise deployment |
| [Cascadia-PLM/FreeCAD-Connector](https://github.com/Cascadia-PLM/FreeCAD-Connector) | "Cascadia PLM workbench for FreeCAD 1.x: link documents to PLM items, vault check-out/check-in, property sync, guided ECO flow" | 0 | 2026-08-21 | 2026-08-22 | unassigned | **Resolved (§13.1 below):** the parent product, [Cascadia PLM](https://cascadiaplm.com/) / [Cascadia-PLM/Cascadia-App](https://github.com/Cascadia-PLM/Cascadia-App), is real, open-source, and actively maintained (27 stars, pushed 2026-09-11) — a general-purpose "Digital Thread" PLM, not FreeCAD/OCCT-specific; this connector is its FreeCAD-side bridge |
| [Taack/taack-plm-freecad](https://github.com/Taack/taack-plm-freecad) | "Simple PLM For FreeCAD" | 15 | — | 2026-09-13 (fetch date itself) | — | Already known from FEP-0011's Motivation list; confirmed still actively pushed on the very day of this research |
| [mnnxp/cadbaselibrary-freecad](https://github.com/mnnxp/cadbaselibrary-freecad) | Addon for [CADBaseLibrary](https://cadbase.rs/en/) | — | — | — | — | Already known from FEP-0011's Motivation list |
| [ZhurbaV/PDMWorkbench](https://github.com/ZhurbaV/PDMWorkbench) | "FreeCAD Workbench to integrate with a local PDM system" | 0 | 2025-05-18 | 2025-05-18 | none declared | **New find.** Appears to be a single-commit scaffold; not independently verified beyond the repo description |
| [DanMiel/PDMforFreeCAD](https://github.com/DanMiel/PDMforFreeCAD) | "Trial for PAD for FreeCAD" [sic, likely means "PDM"] | 1 | — | 2022-11-28 | — | **New find.** Small, apparently abandoned trial |
| [Schumbi/freecad-plm](https://github.com/Schumbi/freecad-plm) + [Schumbi/freecad-plm-addon](https://github.com/Schumbi/freecad-plm-addon) | "A small plm for Freecad FCStd files with 3D-model data" / its FreeCAD addon wrapper | 0 / 0 | 2026-07-17 / — | 2026-09-11 / 2026-09-12 (1–2 days before this fetch) | none declared | **New find, extremely current.** A brand-new, two-repo (core + addon) split PLM project, started and pushed within the same week as this research |
| [ojo42/fc-plm](https://github.com/ojo42/fc-plm) | "A product lifecycle management system to be used with FreeCAD" | 0 | — | 2021-11-01 | — | **New find.** Dormant since 2021 |
| [cadracks-project/freecad-workbench-plm](https://github.com/cadracks-project/freecad-workbench-plm) | "PLM workbench for FreeCAD" | 1 | 2018 | 2018-08-23 | — | **New find** — see §10.7, an empty scaffold, never built beyond a license file and an empty `specs/` directory |

## 10.2 FreeCAD workbenches/addons specifically for version control, git integration, or visual diffing

| Project | Description | Stars | Created | Last push | License | Notes |
|---|---|---|---|---|---|---|
| [eblanshey/HistoryWorkbench](https://github.com/eblanshey/HistoryWorkbench) | "A FreeCAD workbench to track CAD model history and review changes using 3D and tree comparisons" | **144** | 2026-05-05 | 2026-09-01 | LGPL-2.1 | **This is the "History Workbench" already referenced without attribution in `possible_freecad_collaboration.md` §5 ("shipped Sept 2026, wraps git with CAD-friendly terms and 3D/tree diffing") — now identified by name and author.** By far the highest-star project in this entire census, suggesting real community pull for exactly the "git-friendly diffing" angle over the "server-based PDM" angle |
| [reox/FreeCAD_gitproject](https://github.com/reox/FreeCAD_gitproject) | "Make Versioning FreeCAD Projects the easy way" | 11 | 2017-11-08 | 2018-01-06 | LGPL-2.1 | **New find.** Dormant ~8 years; an early (2017–2018) attempt at the same problem HistoryWorkbench solves in 2026 |
| [p-friedrich/versioncontrol-workbench](https://github.com/p-friedrich/versioncontrol-workbench) | "FreeCAD Workbench for using version control systems with CAD" | 8 | 2018-07-20 | 2018-09-26 | none declared | **New find.** Dormant since 2018 |
| [levity0815/freecad_git_tryout](https://github.com/levity0815/freecad_git_tryout) | "trying to manage freecad documents with git" | 3 | 2018-07-08 | 2018-08-09 | — | **New find.** An explicit experiment/prototype, one month of activity, dormant since 2018 |
| [cadracks-project/freecad-workbench-git](https://github.com/cadracks-project/freecad-workbench-git) | "Git version management of CAD projects" | 0 | 2018 | 2018-08-23 | — | **New find** — see §10.7, another empty scaffold from the same abandoned cadracks initiative |
| pieterhijma's PR #28312 + FEP-0013 (Visual Diffs) | Core-side, file-format-level approach to the same problem | — | — | active (2026-09) | — | Already covered in full in §9 above — cross-referenced here because it's the same problem (git-friendly `.FCStd`) approached from inside FreeCAD core rather than as an external addon |

**Pattern:** at least **five** independent "make FreeCAD play nice with git" attempts exist (2017, 2018×3, and the 2026 HistoryWorkbench), plus a sixth core-side attempt (FEP-0013/PR #28312) — the single most duplicated sub-problem found in this entire census. HistoryWorkbench's star count (144, dwarfing every other project here) suggests this specific problem — not full server-based PDM — is where the actual FreeCAD user community's demand concentrates.

## 10.3 Cloud CAD-sharing / hosted collaboration platforms

| Project | Description | Stars | Created | Last push | Notes |
|---|---|---|---|---|---|
| [FreeCAD/Ondsel-Lens-Addon](https://github.com/FreeCAD/Ondsel-Lens-Addon) | FreeCAD-side addon for Lens | — | — | — | Already covered in sections 1–9 |
| [opencomputeproject/CADCloud](https://github.com/opencomputeproject/CADCloud) | "CADCloud is a cloud based sharing infrastructure dedicated to Open Hardware communities... allows users to share design files, browse them interactively through a simple web interface" | 108 | 2020-07-16 | 2022-01-12 | **New find, confirmed FreeCAD-specific.** README states it *"'support[s]' FreeCAD 0.19 (current developer version) with the CLOUD workbench pre-compiled"* — ships a **custom FreeCAD 0.19 build with a dedicated `Cloud` workbench** (`Cloud.URL`, `Cloud.TokenAuth`, `Cloud.Save`/`Cloud.cloudrestore` Python API), backed by Amazon S3 storage, distributed as a Docker container bundling a snap-packaged FreeCAD. Second-highest star count in this census after HistoryWorkbench. Dormant since Jan 2022. **Enrichment (§14):** the project's own Open Compute Project blog post describes an explicit git-fork-style federation feature — *"project forks to allow users to retrieve existing projects from one user to the other, avoiding the need to redesign parts"* — i.e. CADCloud was designed around shared, forkable mechanical parts, not just private cloud backup |
| GrabCAD Workbench (Stratasys, commercial, not open source) | Cloud PDM product, launched 2013/2014 as "design collaboration in minutes" | n/a | n/a | **Discontinued 1 June 2023** | **New find, included as prior art/context.** Not FreeCAD-specific or open-source, but frequently came up in FreeCAD-user PDM discussions (GrabCAD's own Q&A has a `pdm`-tagged FreeCAD thread). Its shutdown is cited in industry commentary (Beyond PLM blog) as a cautionary tale for "PLM for SMBs" — relevant context for why open, self-hostable alternatives keep getting re-attempted rather than trusting a vendor's cloud PDM long-term |

## 10.4 BOM / parts-library tooling (PDM-adjacent, not full PDM)

| Project | Description | Stars | Notes |
|---|---|---|---|
| [alekssadowski95/FreeBOM](https://github.com/alekssadowski95/FreeBOM) | "Open-source multi-level bill of materials (BOM) to manage assemblies and parts in manufacturing software like PDM or PLM" | 1 | **New find.** Explicitly designed as a component other PDM/PLM tools can plug into, not a full system itself; topics include `cyclonedx` (a software supply-chain SBOM standard, notably reused here for hardware BOMs) and `nanoplm` (cross-links to the author's own nanoPLM) |
| [APEbbers/BillOfMaterials-WB](https://github.com/APEbbers/BillOfMaterials-WB) | "FreeCAD Workbench to create a Bill of Materials for various assembly workbenches" | — | **New find**, via the official Addon Manager catalog (`AddonCatalog.json`) — a listed, installable addon, actively pushed (2026-07-25) |
| [alekssadowski95/OpenPartsLibrary](https://github.com/alekssadowski95/OpenPartsLibrary) (+ `-Flask`, `-packages`, `-website`) | "Build machines faster with cheap standard components and ready-to-use CAD files"; the Flask variant's description explicitly says *"Build your own inventory system, bill-of-materials (BOM), product data management (PDM) system or product lifecycle system (PLM)"* | 27 (main repo) | **New find.** A parts-library-as-a-service intended as infrastructure for other PDM/PLM tools to build on, not a PDM system itself |
| [InvenTree](https://inventree.org/) | General-purpose open-source inventory/BOM management system with a plugin architecture | — (large, unrelated-to-FreeCAD-specifically project) | Searched specifically for a FreeCAD integration; **none found** as of this research — InvenTree has an official KiCad plugin and Fusion 360 BOM-exporter addin, but no dedicated FreeCAD plugin exists yet (would need custom development against InvenTree's plugin API) |

## 10.5 ERP/PLM platform integrations (FreeCAD as one of several supported CAD tools, not FreeCAD-native)

| Project | Description | Notes |
|---|---|---|
| **OpenPLM** (multiple, unrelated/semi-related lineages) | Django-based open-source PLM, one of the earliest (pre-2015) open PLM systems with an actual FreeCAD plugin | See detailed breakdown below — this is a small family of repos, not one project |
| [OmniaGit/odooplm](https://github.com/OmniaGit/odooplm) | "Open source PLM/PDM for Odoo — CAD integration (SolidWorks, Inventor, Solidedge, Autocad, ...), BOM revisioning, engineering workflows, browser 3D viewer" | **New find, real and current.** 152 stars (highest star count of any *platform-level* PLM found in this census, though it's an Odoo module, not FreeCAD-native), created 2017-10-07, **pushed 2026-09-12 — one day before this fetch.** Maintained by OmniaSolutions, who published a dedicated blog post ("FreeCAD" on omniasolutions.website) and presented "From Design to Business: The Power of CAD/PDM/PLM and Odoo Integration" at Odoo Experience 2025, and have a "FreeCAD OdooPLM Integration" demo video on YouTube. This is the most commercially mature FreeCAD-adjacent PLM integration found in this entire census, and it was **not previously known to this research** despite being older and more active than most FreeCAD-native attempts |

### OpenPLM lineage detail

- [amarh/openPLM](https://github.com/amarh/openPLM) — the original Django-based openPLM, GPL-3.0-or-later, 47 stars, created 2014-11-07, last pushed 2024-01-30. Its documentation (`wiki.openplm.org` / `openplm.org/docs`) describes a dedicated **FreeCAD plugin adding an "OpenPLM" workbench** to FreeCAD, supporting check-in (save to server), creating new document revisions, and creating new PLM documents from previously un-checked-out files — a full classic-PDM check-in/check-out workflow. **Confirmed by primary-source forum history to date to 2010** — openPLM's own developer ("Philippe") posted to the FreeCAD forum on 2010-09-14 announcing the plugin and asking for a sample part for a demo video, drawing replies from all three FreeCAD co-creators (wmayer supplied a sample file from FreeCAD's own SVN repository; yorik and jriegel both replied enthusiastically) — making this the **earliest FreeCAD/PDM integration found anywhere in this research to actually ship working code**, predating every other project in this census by at least four years. It did not age well: by a 2017 forum discussion, a contributor confirmed openPLM's FreeCAD plugin had gone **four years without a release and only ever supported FreeCAD 0.10/0.11** — obsolete relative to FreeCAD's own evolution within a few years of shipping.
- [cadracks-project/openplm](https://github.com/cadracks-project/openplm) — "OpenPLM revival," 11 stars, created and abandoned within one day (2018-08-16 to 2018-08-17) — part of the broader cadracks initiative, see §10.7.
- [openPLM/openplm](https://github.com/openPLM/openplm) — a separate, apparently unrelated project that has simply reused the "openPLM" name for "An Open Source PLM for CPG/Retail" (consumer packaged goods/retail, not engineering/CAD) — created 2026-01-16, 0 stars, single commit. **Name collision, not a continuation** — flagged here only to prevent future confusion between it and the actual CAD-focused openPLM lineage above.
- [docdoku/docdoku-plm](https://github.com/docdoku/docdoku-plm) — a comprehensive open-source PLM (Java EE/SQL-based) that came up independently in two different forum discussions (2017 and later) as known prior art alongside openPLM; no FreeCAD-specific plugin or integration was ever mentioned for it in either discussion, so it appears to be a PLM the FreeCAD community was aware of and considered, but never actually integrated with.

### A related, independent 2014 and 2017 thread lineage

Two more forum threads, not focused on openPLM specifically, ran the same "let's build FreeCAD PDM/PLM" idea from scratch, years before FreePDM (§16):

- **2014** — a community member (eukreign) proposed a web-based collaboration platform with a simple check-out/check-in model and camera-position screen-sharing for remote review sessions. jriegel (FreeCAD co-creator) replied with pointers to wiki pages on the same subject as his own 2009 proto-PDM ticket (`freecad_core_and_lens_collaboration_issues.md` §0). No shipped code resulted.
- **2017** — a community member (Malkov) proposed "a free enterprise level PLM/PDM system integrated with FreeCAD," explicitly citing prior Siemens Teamcenter experience and proposing a NoSQL-based architecture — the exact database-technology debate FreePDM's own thread would re-run from scratch five years later. wandererfan (FreeCAD core contributor) pointed at the same historical wiki page jriegel's 2014 responders had cited. No shipped code resulted from this thread either.

Full detail on both, including verbatim quotes, is in
`freecad_core_and_lens_collaboration_issues.md` §0 (since both threads
featured a FreeCAD co-creator or core contributor directly participating).

## 10.6 Real-time / concurrent multi-user editing (cross-reference only — already fully covered elsewhere)

Already documented in depth in `possible_freecad_collaboration.md` §1 and the academic
literature review in `optimistic_locking_research.md`: **CollaborativeFC/OCP** (Stefan
Tröger, archived May 2026) and **JupyterCAD** (the only project with genuine real-time
CRDT sync, via a parallel `.jcad` format rather than editing `.FCStd` directly). Not
repeated here. Notably, Stefan Tröger (GitHub handle `ickby`) personally showed up in
FEP-0011's PR #26306 comments (§8.5 above) raising the identity-verification question —
the CollaborativeFC/OCP author is directly engaged with the *current* collaboration
framework effort, a continuity between the "dead" 2020s real-time-editing era and the
2026 asynchronous-collaboration era that isn't visible unless you cross-reference commit
authorship across projects.

## 10.7 Design-only or never-implemented concepts (real design substance, no shipped code)

- **[cadracks-project](https://github.com/cadracks-project)** (GitHub org, self-described "CadRacks — Assembly of heterogeneous CAD files") ran an entire coordinated initiative around **2018** that attempted several of the exact pieces this whole research area keeps re-attempting, all abandoned within about one month of creation (mid-to-late August 2018):
  - [freecad-workbench-git](https://github.com/cadracks-project/freecad-workbench-git) — "Git version management of CAD projects" (empty scaffold: `.gitignore`, `LICENSE`, a one-line `README.md`, an empty `specs/` directory — never implemented)
  - [freecad-workbench-plm](https://github.com/cadracks-project/freecad-workbench-plm) — "PLM workbench for FreeCAD" (same empty-scaffold state)
  - [openplm](https://github.com/cadracks-project/openplm) — "OpenPLM revival" (created and abandoned within a single day)
  - [opm](https://github.com/cadracks-project/opm) — **"Open Product Model"**: *"The goal of opm is to propose an open product data model for the PLM of Open Hardware projects."* Unlike the scaffolds above, this repo actually contains real design substance: a cited academic reference (`CIRP_Design_2011.pdf`), a MySQL Workbench data-model file (`open_product_model.mwb`), a design slide deck (`.odp`), and two PDFs (`open_product_model.pdf`, `open_product_model_design.pdf`) — i.e., someone did real modeling work on an open PDM data schema for open-hardware PLM, comparable in *intent* to the AAS/VDI-2770 standards work covered in `ondsel_48_research.md`, but apparently done independently, without reference to those standards, and never implemented in code.
  - The org's other repos (`osvcad` — "Open source CAD system for Open Hardware projects", `cadracks-core`, `cadracks-ide`, `cadracks-party`, `freecad-workbench-anchors`, `standard-cad-parts`) show this was a serious, multi-repo attempt at an entire open-hardware CAD+PLM ecosystem parallel to FreeCAD itself (built on CadQuery/pythonOCC rather than FreeCAD proper for some pieces), not a single throwaway experiment — but the whole cluster's activity is concentrated in 2018–2021 and appears fully dormant now (`cadracks-core`, the most recently touched repo, was last pushed 2021-02-09).
  - **This is the clearest example found in this census of real intellectual effort toward exactly this project's own "open, standards-based PDM schema" goal, done years earlier, independently, and now essentially lost to the ecosystem** — nobody in the current FEP-0011 / Ondsel-Server#48 conversation references cadracks or "Open Product Model" at all.

## 10.8 Updated fragmentation count and synthesis

Counting only projects with any FreeCAD-specific PDM/PLM/version-control/collaboration
intent (excluding pure BOM-library infrastructure and excluding the ERP-platform
integrations that treat FreeCAD as just one of several supported CAD tools):

**FreePDM, nanoPLM, Ondsel/Lens, CADBaseLibrary, Taack FreeCAD PLM, GitPDM, EasyPDM,
we-have-PDM-at-home, PiStock, Cascadia-PLM/FreeCAD-Connector, ZhurbaV/PDMWorkbench,
DanMiel/PDMforFreeCAD, Schumbi/freecad-plm(+addon), ojo42/fc-plm,
cadracks/freecad-workbench-plm, HistoryWorkbench, reox/FreeCAD_gitproject,
p-friedrich/versioncontrol-workbench, levity0815/freecad_git_tryout,
cadracks/freecad-workbench-git, CADCloud, cadracks/openplm, cadracks/opm** — **22
distinct, non-interoperating projects**, plus the two platform-level integrations
(amarh/openPLM, OmniaSolutions' OdooPLM) that predate and out-mature most of the
FreeCAD-native attempts, plus GrabCAD Workbench as commercial prior art that already
failed and was discontinued.

Add the three-repo Open Source Ecology "Village Construction Set Library" system
(§10.10 — schemas, a validating/compiling FreeCAD workbench, and a publishing site,
all pushed 2026-09-07) and the count rises to **25+ distinct, non-interoperating
efforts**. This is roughly **8× the fragmentation the original survey identified** (3
named projects → 25+). Reading the dates across all of them, several things stand out
that weren't visible at the smaller scale:

1. **The fragmentation isn't slowing down — it's accelerating.** At least four of the
   projects above (EasyPDM, Schumbi/freecad-plm + addon, Cascadia-PLM/FreeCAD-Connector,
   Taack-plm-freecad's latest push) were created or actively pushed within **the same
   week as this research** (early-to-mid September 2026). Nobody starting a new one
   appears to be searching for or finding the others first.
2. **The community's actual, revealed-preference demand is for git-friendly diffing/
   history, not server-based PDM.** HistoryWorkbench (144 stars) and CADCloud (108
   stars) are the two highest-star projects in the entire census, both far ahead of
   every dedicated PDM/PLM system (the next-highest is OdooPLM at 152 stars, but that's
   an Odoo-ecosystem project where FreeCAD is one of several supported CAD formats, not
   a FreeCAD-native signal). Every FreeCAD-native, from-scratch PDM/PLM server project
   sits in the single-to-low-double-digit star range.
3. **At least two contributors have independently built entire personal multi-repo
   ecosystems around this problem** rather than a single tool — see §10.9. This mirrors
   the pattern already observed in FreeCAD core itself (pieterhijma driving FEP-0011,
   the annotation issues, PR #28312, and FEP-0013 essentially single-handedly).
4. **OdooPLM is the single most surprising find**: older (2017), more active (pushed
   the day before this fetch), better-starred (152), and backed by an actual company
   with conference talks and demo videos — yet it does not appear anywhere in FEP-0011's
   Motivation section's "known initiatives" list, nor in any of the FreeCAD-org issue
   threads researched in sections 1–9. The FreeCAD-core collaboration conversation and
   the most mature actually-shipping FreeCAD-PLM integration are, as far as this
   research can tell, **completely unaware of each other**.
5. **The Open Product Model (cadracks/opm) is a small tragedy of institutional memory**:
   real, standards-minded design work toward exactly this project's own "open PDM
   schema" thesis, done in 2018, entirely forgotten by 2026's participants.

## 10.9 Pattern: single-maintainer "personal ecosystems"

Two contributors stand out for having built *clusters* of interrelated repos around
this problem space, rather than one tool each — worth naming as a pattern in its own
right, since it changes how "22+ independent projects" should be read (it's not 22
independent *people/teams*, several are the same person iterating across repos):

- **pieterhijma** (FreeCAD core): FEP-0011 (Collaboration Framework), PR #26306 (its
  implementation), the #25681/#25682/#25685 issue cluster, PR #28312 (versioning file
  format / document cache), and FEP-0013 (Visual Diffs) — five interlocking efforts,
  fully documented in sections 1–9 above.
- **alekssadowski95** (FreeCAD ecosystem, external): nanoPLM, FreeBOM,
  we-have-PDM-at-home, OpenPartsLibrary (+ its Flask blueprint, packages repo, and
  website), and PyPDM — at least six interlocking repos toward the same "PDM for small
  manufacturers" goal, built by one person across a very high-output GitHub profile
  (70+ repos total, many FreeCAD-related — STEMFIE construction-toy system,
  FreeCAD-Beginner-Assistant, FreeCAD-AI-Toolbar, OpenRadioss tooling, and more).

A third instance of the same pattern, at institutional rather than individual scale:

- **Open Source Ecology** (org): `vcs-library` (schemas/validators/ontology),
  `ose-library-workbench` (the FreeCAD-side authoring/compile/validate tool),
  `ose-library-site` (publishing), plus the older `ose-3d-printer-workbench` and
  `ose-workbench-platform` — a coordinated, internally-consistent system (§10.10),
  but one that, like the two individual clusters above, shows no sign of awareness of
  anything else in this census.

Neither pieterhijma's, nor alekssadowski95's, nor OSE's work references the others', and
none references cadracks-project's 2018 attempt at the same thing. This is the clearest
evidence in the whole census that the fragmentation problem isn't primarily about lack
of individual effort, ambition, or even coordination *within* a team — several
individuals and at least one well-organized institution have independently produced
substantial, multi-repo, internally coherent bodies of work — it's specifically a
**cross-project discovery and coordination** failure: the same small set of ideas (item
identity, schemas, validation, revisions, BOM, check-out locking, git-backed history)
gets re-invented by isolated, capable people and organizations who apparently never
find each other's prior art before starting.

## 10.10 Open Source Ecology's "Village Construction Set Library" — a live, days-old schema/PDM effort

The Open Source Ecology wiki page "FreeCAD PDM" that prompted this sub-search
(https://wiki.opensourceecology.org/wiki/FreeCAD_PDM) could not be fetched directly —
it's behind a Cloudflare bot-challenge that blocked both `curl` (with a browser
User-Agent) and the MediaWiki API endpoint in this session. Search-engine snippets
describe it only in general terms: OSE (Marcin Jakubowski's long-running open-hardware
project building the 50-machine "Global Village Construction Set") uses FreeCAD for
mechanical design and has been "developing design workbenches for each of the 50 GVCS
machines," with the page apparently discussing PDM challenges for managing those design
files collaboratively.

But the live GitHub org (`OpenSourceEcology`) tells a much more concrete and current
story than the wiki snippet: **on 2026-09-07 — six days before this research — OSE
created a coordinated three-repo system** that is, functionally, exactly the kind of
schema-driven, validated, versioned part-library PDM infrastructure this project's own
research has been recommending in the abstract (`ondsel_project_proposal_research.md`
Stage 1). It appears to be entirely unconnected to any other project in this census —
no reference to Lens, FreePDM, nanoPLM, HistoryWorkbench, or FEP-0011 found anywhere in
its docs.

- **[OpenSourceEcology/vcs-library](https://github.com/OpenSourceEcology/vcs-library)**
  — "Village Construction Set part library: schemas, compilers, validators, and library
  ontology." From its README: a canonical part library organized into four explicit
  layers — `parts` (individual buildable pieces) → `modules` (reusable building
  modules) → `assemblies` (groups of modules/parts) → `structures` (whole structures,
  "declared and currently empty") — i.e., an explicit part-of hierarchy strikingly
  similar to Ondsel-Server issue #48's own part/assembly/sub-assembly vocabulary
  (§5 above), arrived at completely independently. Ships a `libtools` Python package
  with a `validate-code` command, `GOVERNANCE.md`, `LIBRARY_ONTOLOGY.md`, and
  `CONTRIBUTING.md` docs, and cites an OSE wiki page "Schema Canon"
  (https://wiki.opensourceecology.org/wiki/Schema_Canon) as its governing structure.
  A `collections/gvcs` sub-tree holds "sourced Universal Axis, Power Cube, and CEB Press
  geometry with preserved provenance and review status" — i.e., real machine data, not
  a placeholder.
- **[OpenSourceEcology/ose-library-workbench](https://github.com/OpenSourceEcology/ose-library-workbench)**
  — "FreeCAD workbench for authoring and validating OSE part-library entries." A real
  FreeCAD 1.x workbench with a documented, non-trivial workflow: **Open Library** (pick
  a library root) → **Compile Entry** (schema → geometry, creating a FreeCAD document)
  → **Edit Parameters** / **Apply** (re-run the compile; *"A failed compile rolls back,
  retaining the previous geometry and parameters. Other objects you added to the
  document are preserved"* — i.e., it has real transactional/rollback semantics, a
  detail most of the smaller projects in this census don't attempt) → **Validate
  Entry** (checks geometry and source against the schema, writing reports to
  `<library-root>/reports/`) → **Export changed schema…** (writes a `schema.py.new`
  for manual review, deliberately not auto-applying schema changes). Saved `.FCStd`
  documents retain "library location, entry identity, managed object names, and last
  successfully applied schema" — i.e., FreeCAD documents carry back-references to their
  PDM-library entry, a concrete instance of exactly the "item identity that outlives a
  file" problem `ondsel_project_proposal_research.md` Stage 0 discusses in the abstract.
- **[OpenSourceEcology/ose-library-site](https://github.com/OpenSourceEcology/ose-library-site)**
  — "Static site generator for OSE part libraries — entry pages with 3D views, BOMs,
  fab drawings, and validation status" — the public-facing/publishing leg of the same
  system.

Two more OSE repos from the same push window round out the picture: **iconic-cad**
(pushed 2026-09-07) — *"OSE fork of Collin DeSantis's Iconic CAD — AI-assisted
parametric CAD for the Seed Eco-Home"* (upstream: `gitlab.com/collindesantis/iconic-cad`,
not independently researched in this pass) — and **iconic-tutor** — *"Interactive
graphical composition tutor for Open Source Ecology"* — pushed literally on this
research's own fetch date (2026-09-13T00:26:53Z), the single most recent commit found
anywhere in this entire census.

**Why this matters more than its lack of stars (0/0/0, all three repos) suggests:** this
is schema-first, validated, rollback-aware, provenance-preserving part-library
infrastructure for FreeCAD, built by a well-resourced, decades-old open-hardware
organization, shipped days before this research — and it is completely invisible to
everyone in the FEP-0011/Ondsel-Server#48 conversation. It's the single most current
and concrete counter-example to this section's own "22+ projects, mostly small and
stagnant" framing: at least one of these efforts is neither small in ambition nor
stagnant, it's simply happening in total isolation from the rest of the ecosystem this
document maps.

## Sources for section 10

- GitHub REST repository-search API: `api.github.com/search/repositories?q=freecad+pdm`, `...+plm`, `...+versioning`, `...+collaboration`, `...+teamwork`
- `FreeCAD/FreeCAD-addons` repository (`AddonCatalog.json` and root submodule listing) — the official Addon Manager catalog
- [OpenPLM FreeCAD plugin docs (search-indexed, live site unreachable)](http://wiki.openplm.org/docs/dev/en/user/plugin_freecad.html)
- [OmniaSolutions blog — "FreeCAD"](https://www.omniasolutions.website/blog/our-blog-1/post/freecad-20)
- [Odoo Experience 2025 — "From Design to Business: The Power of CAD/PDM/PLM and Odoo Integration"](https://www.odoo.com/event/odoo-experience-2025-6601/track/from-design-to-business-the-power-of-cadpdmplm-and-odoo-integration-7865)
- [Beyond PLM blog — "GrabCAD Workbench Fond Farewell"](https://beyondplm.com/2023/04/10/grabcad-workbench-fond-farewell-lessons-and-what-is-next-in-plm-for-smbs/)
- [Open Source Ecology wiki — "FreeCAD PDM"](https://wiki.opensourceecology.org/wiki/FreeCAD_PDM) — **fetch blocked (HTTP 403) in this session; not independently verified, flagged as an unresolved lead below**
- Individual repository pages cited inline above (GitHub API `created_at`/`pushed_at`/`stargazers_count`/`license`/`topics` fields, and raw READMEs where quoted)

## Not yet fetched / unresolved from section 10

- **Open Source Ecology's "FreeCAD PDM" wiki page** (https://wiki.opensourceecology.org/wiki/FreeCAD_PDM) itself remained unfetchable (Cloudflare challenge blocked both `curl` and the MediaWiki API in this session) — but its likely current, operational descendant was found and fully documented instead: see §10.10, the `OpenSourceEcology/vcs-library` + `ose-library-workbench` + `ose-library-site` triad pushed 2026-09-07. The historical wiki page's exact original content remains unverified; worth a dedicated retry (different network path, or asking the user to fetch it manually) if the *history* of OSE's PDM thinking (as opposed to its current shipped state) becomes relevant. Also unfetched: the `Schema Canon` and `Catarina_Log` wiki pages referenced by `vcs-library`'s own README.
- **docdoku/docdoku-plm** — a comprehensive open-source PLM found in the same search sweep as OpenPLM; FreeCAD-specific support not confirmed.
- **amarh/openPLM's FreeCAD plugin** — described only via search-indexed doc snippets in this pass; the live `wiki.openplm.org`/`openplm.org` sites were unreachable (DNS failure) in this session, so the plugin's actual behavior/code was not independently inspected.
- **Cascadia-PLM/FreeCAD-Connector**'s parent "Cascadia PLM" product — not independently researched; unclear if it's commercial, private, or itself open.
- ~~A live, exhaustive **GitHub topics** search~~ — **done, see §17.**

---

# 11. Gap-check pass: academic connectors, browser ports, and categories missed by keyword search

Prompted by the question "is there any effort anywhere NOT in this doc?" — the keyword
searches in section 10 (`freecad+pdm/plm/versioning/collaboration/teamwork`) were biased
toward projects that self-describe with those exact words. This pass deliberately
targeted categories that wouldn't surface that way: peer-reviewed academic connectors,
BIM/AEC data-platform bridges, browser/WASM ports (a collaboration *enabler*, not a PDM
tool itself), cloud-storage-access workbenches, and a non-English-language spot-check.
Confirmed new, real findings below; ruled-out leads are listed at the end so they aren't
re-searched later.

## 11.1 Academic real-time-collaboration connector: FreeCAD ↔ NVIDIA Omniverse (University of Manchester / UKAEA fusion research)

This is the single most substantial gap found — a **peer-reviewed, funded, actively
maintained** academic project doing exactly what CollaborativeFC/OCP attempted and
failed at (real-time collaborative CAD), from a completely different community
(nuclear fusion engineering) that never appears in any FreeCAD-org or Ondsel-Server
discussion.

- **Paper:** Raska Soemantoro, Lee Margetts. "An Omniverse Connector for FreeCAD."
  *Journal of Open Research Software*, 2025. https://doi.org/10.5334/jors.559
  (open access). Earlier conference poster: "A case study of real-time collaborative
  design in FreeCAD and NVIDIA Omniverse," Computing Insight UK, Manchester, 4–5
  December 2024.
- **Authors/affiliation:** University of Manchester (Soemantoro is a postgraduate
  researcher affiliated with the UK's Fusion CDT — Centre for Doctoral Training).
- **Funding:** EUROfusion (EU/Euratom Research and Training Programme, Grant 101052200),
  UK EPSRC, and the UK Atomic Energy Authority (UKAEA) Fusion Industry Programme — i.e.,
  this is government/EU-funded fusion-energy engineering infrastructure work, not a
  hobbyist project.
- **Code:** https://github.com/Metaverse-Colab-for-Fusion-Energy/FreeCAD-Omniverse — BSD
  3-Clause, 6 stars, created 2023-10-31, **pushed 2026-06-24** (still maintained),
  archived on Zenodo (DOI 10.5281/zenodo.14866087) for citability, current release
  v3.0.3.
- **What it does:** a FreeCAD workbench that connects to an NVIDIA Omniverse Nucleus
  server (a "secure, central storage platform"), maintaining **dual-format storage**
  per asset — an authoritative STEP file plus a lightweight USD (Universal Scene
  Description) proxy for Omniverse — with a strict forward-only rule (geometry is only
  ever pulled from STEP, never reverse-converted from USD, to avoid corrupting the
  authoritative model). Assemblies use USD References (pointers, not duplicated
  geometry) so a change propagates automatically. **Version control**: Omniverse's
  built-in checkpoint system tags each change with a token visible as a FreeCAD object
  property, linking STEP and USD history together — though the paper is explicit that
  only a single linear "default" branch is supported; **branching/merging is not yet
  available**, an honest limitation, not a solved problem. **Collaboration**: an
  on-demand sync mode (push/pull between FreeCAD and Nucleus) plus a **live assembly
  mode** that streams real-time position updates from Omniverse into FreeCAD during a
  connected session (currently one-directional, Omniverse→FreeCAD only).
- **Case study:** engineers collaboratively designing a sensor for a stellarator fusion
  reactor; separately, the connector was stress-tested on the largest single asset in
  any project found in this whole census — a 300MB STEP / 1.93GB USD model of a
  stellarator fusion device's inner vessel.
- **Why this was missed by section 10's searches:** it never uses the words "PDM,"
  "PLM," or even "collaboration" as its primary self-description — it's filed under
  digital-twin/Omniverse/USD tooling in the fusion-energy literature, a completely
  different citation network than the FreeCAD-addon or PDM-vendor worlds section 10
  searched.

## 11.2 Academic BIM-data-platform bridge: Speckle ↔ OpenCascade (EPFL/ETH, explicitly targets FreeCAD)

- **[ENAC-CNPA/speckle-opencascade](https://github.com/ENAC-CNPA/speckle-opencascade)**
  (repo was renamed from `speckle-freecad` at some point — the old name still redirects)
  — "developed as a research project conducted at the LAPIS laboratory at EPFL ENAC IA,
  building on previous research... at the CNPA laboratory. The project was supported by
  the Open Research Data Program of the ETH Board." 5 stars, created 2024-10-07, pushed
  2026-03-23.
- **What it is:** a prototype Python connector between **Speckle** (an open-source data
  platform for exchanging 3D/BIM data across AEC tools — already namechecked, without
  a link, by marcuspollio in the FEP-0011 discussion in section 8.6 above) and
  OpenCascade (FreeCAD's own geometry kernel), via `specklepy` and `python-occ-core`.
  Its own README states its two long-term goals explicitly: *"to serve as a basis for
  developing connectors for any software based on OpenCascade, such as FreeCAD... to
  provide Speckle with geometric computation capabilities."* **FreeCAD is named as the
  explicit target, not yet built** — this is upstream infrastructure for a future
  FreeCAD↔Speckle bridge, funded by a Swiss university consortium (EPFL + ETH Board),
  with zero visibility in any FreeCAD-org or Ondsel-Server conversation found in this
  research.

## 11.3 Browser/WebAssembly ports of FreeCAD (a collaboration *enabler*, not a PDM tool)

Not PDM/PLM themselves, but directly relevant to any future "collaborate on a FreeCAD
model without installing FreeCAD" story — and notable for how they were built:

- **[Virtastic/freecad-web](https://github.com/Virtastic/freecad-web)** — "FreeCAD
  compiled to WebAssembly and running in the browser: the real application, wasm64 +
  JSPI, same workbenches, same solvers." Created 2026-07-02, **pushed on this
  research's own fetch date (2026-09-13)**. Hosted live at
  https://freecad.virtastic.app. Full port — same 20 workbenches, 578 commands, OCCT
  kernel, CPython interpreter, same solvers — not a stripped-down demo.
- **magik.net's FreeCAD port** (https://magik.net/freecad/) — a separate, independent
  WASM port with the same goal (also LibreCAD, per `magik.net/librecad/`). Notable
  detail: per the port's own public writeup, the engineering work was performed *"almost
  entirely by an AI agent (Fable/Claude Code) across 4 days, 3 sessions, 15 multi-agent
  workflows, and 159 sub-agent invocations."* First load ~96MB compressed, requires a
  recent Chromium-based browser (WebAssembly JSPI isn't shipped by Firefox/Safari yet).
  Supports 17+ workbenches including FEM with real mesh geometry.
- **Relevance to this research:** neither project claims to solve multi-user editing,
  but a real, full-fidelity, no-install FreeCAD running in a tab is the precondition
  for the kind of "just send a link" collaboration that Google Docs-style tools take for
  granted and every project in section 10 has to work around with file check-in/out.
  Worth revisiting if either project adds any shared-session capability.

## 11.4 Cloud-storage-access workbenches (yet another duplicated-effort pair)

- **[hitclawagent/freecad-cloud-browser](https://github.com/hitclawagent/freecad-cloud-browser)**
  — "FreeCAD workbench to browse and open files from cloud storage (Google Drive,
  Dropbox, OneDrive, S3, FTP, WebDAV)." 3 stars, created 2026-05-04, pushed 2026-05-07.
- **[sabi137032/freecad-cloud-browser](https://github.com/sabi137032/freecad-cloud-browser)**
  — "Browse and open files from S3, FTP, SFTP, and WebDAV storage directly within the
  FreeCAD interface using this workbench plugin." 1 star, created **three days later**
  (2026-05-07) than the repo above, **still being pushed on this research's fetch date**
  (2026-09-13). Same name, overlapping feature set, no visible relationship between the
  two authors.
- **Pattern match:** this is the same "duplicated effort within days of each other,
  same name, no cross-awareness" signature already documented for HistoryWorkbench-era
  git tools (§10.2) and the Schumbi freecad-plm pair (§10.1) — now confirmed a third
  time, in a completely different sub-category (cloud file access, not PDM/versioning
  per se), suggesting the discovery/coordination failure named in §10.9 is a structural
  feature of the FreeCAD addon ecosystem generally, not specific to PDM.

## 11.5 Checked and ruled out (searched specifically, no FreeCAD-specific project found)

- **OpenBOM** — a real, mature multi-CAD BOM/PLM platform (SolidWorks, Fusion 360,
  Inventor, Solid Edge, Altium, Onshape integrations all confirmed) with **no FreeCAD
  integration found**, prebuilt or community. Its own docs point to a public
  REST API/CAD-integration-toolkit as the path if anyone wanted to build one — nobody
  appears to have.
- **Wikifactory** — a well-funded ($4.5M raised), real "Cloud PDM + CAD Viewer +
  Manufacturing" platform for open hardware, 20,000+ projects, viewer support for 30+
  CAD formats. No FreeCAD-specific integration, plugin, or case study was found in this
  pass — plausible that FreeCAD files are viewable generically (e.g. via STEP export)
  but nothing confirms a native FreeCAD connection.
- **CERN** — searched specifically for CERN's use of FreeCAD for accelerator-component
  collaborative design. Found nothing — CERN's major open-hardware/collaboration
  investment is in **KiCad** (a 17,000-component open-source PCB library, released
  2026), not FreeCAD. Worth recording as a deliberately-checked negative, not an
  oversight.
- **Chinese-language FreeCAD PDM/协同设计 search** — a single spot-check search (not a
  systematic non-English sweep) surfaced only generic Chinese-language PDM/PLM industry
  content and a Chinese-language FreeCAD community site (`free-cad.cn`), no
  FreeCAD-specific PDM/collaboration project distinct from what's already in section 10.
  **This was one query in one non-English language; German, French, Russian, Japanese,
  Korean, and Portuguese-language FreeCAD communities were not searched at all in this
  pass** — flagged as the clearest remaining systematic gap in this entire document.

## 11.6 Remaining gaps acknowledged (not searched, or searched only shallowly)

- **Non-English-language communities**, as above — only one Chinese-language query was
  run; this is very likely where additional independent, undiscovered efforts exist,
  given the pattern (documented throughout section 10) of isolated individuals/small
  groups building PDM-adjacent tools without cross-referencing the English-language
  GitHub/forum ecosystem this research is anchored in.
- **University thesis repositories / institutional repositories** beyond the one
  Manchester paper found — no systematic search of ProQuest, university ETD archives
  (BYU's ScholarsArchive was searched in `optimistic_locking_research.md` for the
  *general* CAD-merge literature, but not re-run with "FreeCAD" as a specific keyword).
- **Corporate/commercial FreeCAD-hosting or FreeCAD-support companies** that might offer
  proprietary collaboration layers on top of FreeCAD as a paid service — not
  systematically searched; only OmniaSolutions (OdooPLM, §10.5) surfaced this way.
- **Discord/Slack/forum-native "workgroup" coordination** (as opposed to shipped
  software) — e.g., informal BIM-workbench weekly meetings (mentioned in passing in the
  2026-09-06 developer-meeting minutes, section 8.7) — these are process/community
  patterns, not software projects, and weren't systematically catalogued here since they
  don't produce a discoverable repo.

---

# 12. International-language sweep (all languages, all regions)

Prompted directly by the question "is there any effort anywhere in the world... I may
have artificially limited the scope with my poor choice of terminology" — five parallel
research passes covered Western European languages, Eastern European/Russian, East
Asian languages, the OpenCascade-specific technical ecosystem independent of any
language, and untried English terminology. Results below; sections 13 and 14 cover the
latter two passes.

## 12.1 Western Europe (German, French, Spanish, Italian, Dutch, Portuguese)

**No new standalone FreeCAD-native collaboration/PDM project was found in any of these
six languages.** What did surface:

- **[ALSADO](https://alsado.de/freecad-support)** (Germany) — a real commercial FreeCAD
  consulting/support company that lists "nanoPLM - open source PLM FreeCAD" as one of
  its service offerings. Not a new project — it's a business built partly around
  *supporting* the already-known nanoPLM (§10.1), which is itself worth noting as a
  signal that nanoPLM has enough real-world traction to sustain paid consulting around
  it.
- **"Petit PLM"** (French-language LinuxFr.org journal post, 2023-02-09,
  https://linuxfr.org/users/yboy360/journaux/petit-plm-pour-freecad) — turned out to be
  the Taack PLM developer's own French-language announcement of Taack PLM (already
  fully catalogued in §10.1 as `Taack/taack-plm-freecad`), not a separate project.
- **openPLM's French-language documentation mirror** (`openplm.org/docs/*/fr/`) —
  same already-known tool (§10.5), just multilingual docs.
- Spanish, Italian, Dutch, and Brazilian Portuguese searches returned **only generic
  marketing/comparison content** (CAD-software listicles, commercial PDM vendor ads) —
  explicitly checked, explicitly empty.

(This pass also surfaced the 2009 Jürgen Riegel ticket and the "About PDM for FreeCAD"
forum thread, both English-language artifacts found incidentally — see
`freecad_core_and_lens_collaboration_issues.md` sections 0 and 10, since they belong
with FreeCAD's own official-channel history, not this third-party-project census.)

## 12.2 Eastern Europe and Russian (Russian, Polish, Czech, Ukrainian)

**No new standalone project found.** One existing entry enriched with new facts:

- **CADBase** (already catalogued at §10.1 as `mnnxp/cadbaselibrary-freecad`) is
  **confirmed Russian in origin** — per a 2023 Habr.com article
  (https://habr.com/ru/articles/720120/), development began in 2018, with "Russian
  language localization and FreeCAD integration" added as a more recent update.
  **Correction to prior entry:** its actual development repository is on **GitLab**,
  not GitHub — https://gitlab.com/cadbase (GraphQL API, cloud storage, supplier
  catalogs, multi-company permissions). The `mnnxp/cadbaselibrary-freecad` GitHub repo
  already in this census is only the FreeCAD-side addon, a mirror/client of the real
  GitLab-hosted platform.
- **[cccp3d.ru](https://cccp3d.ru)** — a real, active Russian-language CAD/PLM/PDM
  discussion forum (dominated by SolidWorks/KOMPAS-3D content), confirmed to have
  FreeCAD-specific threads (e.g. `/topic/202246-freecad-10/`) — but FreeCAD and PDM are
  discussed there as **separate** topics; no combined FreeCAD-PDM thread or project
  found on it.
- Polish, Czech, and Ukrainian searches (native terminology: "praca zespołowa,"
  "zarządzanie danymi produktu," "týmová spolupráce," "správa dat o výrobku," "спільне
  проектування," "командна робота," etc.) returned only commercial PDM-vendor pages
  (dps-software.pl, cadworks.pl, SolidVision, TechCAD, CADstudio) and, in the Czech
  case, one hit — already-known FreePDM — but nothing new. A Cyrillic-description
  GitHub repo-search query (`freecad+совместн`) returned zero results.

## 12.3 East Asia (Chinese, Japanese, Korean)

**One genuine but ambiguous new lead; two more incidental English-language finds
surfaced (folded into the core-issues doc, not repeated here); otherwise confirmed
empty.**

- **UESOFT / AutoPDMS → FreeCAD port** — a real, established Chinese engineering
  software company (优易软件, Changsha, Hunan, founded 2000; AutoPDMS is their flagship
  product, self-described as *"the next-generation 3D plant collaborative design
  management system,"* claiming ~50% share of China's piping hanger/support design
  software market and 5,000+ users as of a 2016 figure) maintains a
  **[gitee.com/FreeCAD](https://gitee.com/FreeCAD)** org (Gitee = China's
  GitHub-equivalent) whose own description states, in Chinese: *"FreeCAD是UESOFT公司移植
  AutoPDMS到FreeCAD开源软件的团队"* — "FreeCAD is UESOFT company's team porting AutoPDMS
  to the FreeCAD open-source software." **Caveat:** the actual repos under that org
  (`geom`, `gui`, `kernel`, `shaper`, `smesh`, `configuration`) are labeled
  "clone-salome-platform" — i.e. they currently mirror **SALOME** (a different
  OCCT-based CAD/CAE platform, see §13 below), not FreeCAD-specific code. So: a real
  company with a stated intent to port a real, dominant commercial plant-PDM product to
  FreeCAD/OpenCascade, but the visible repo evidence is SALOME-platform mirroring, not
  a shipped FreeCAD PDM addon yet. Also noted: a separate Chinese company,
  **Jinan Youquan Software** (`gitee.com/jinan-youquan-software/FreeCAD`), maintains its
  own FreeCAD fork on Gitee — relationship to PDM not confirmed, flagged only as another
  Chinese commercial entity directly engaging with FreeCAD source.
- Japanese search (共同設計/チーム設計/製品データ管理 + FreeCAD, including a Qiita-specific
  pass) found only generic tutorial content; one tangential Qiita KiCad-FreeCAD article
  recommends *external* version control because FreeCAD can lose work history, but no
  dedicated Japanese FreeCAD-PDM tool or community effort was found.
- Korean search (협업 설계/PDM/PLM, 팀 설계/도면관리 + FreeCAD) found only FreeCAD
  encyclopedia/wiki pages (Namuwiki, Korean Wikipedia) and download guides — no
  Korean-originated project found.
- **Assessment, stated honestly by the research pass itself:** non-Latin-script
  indexing is a real constraint — Gitee search in particular surfaces mirrors/forks far
  more readily than genuinely original native-language projects, so this pass likely
  **undercounts** real Chinese/Japanese/Korean-originated efforts rather than
  confirming their absence. Treat 12.3 as the weakest-confidence section in this
  document, not as evidence East Asia lacks activity in this space.

## 12.4 Cross-cutting observation from the international sweep

Across four language passes (12.1, 12.2, 12.3 above), the pattern is consistent:
**most non-English "hits" are not new projects but multilingual surfaces of projects
already found in English** (nanoPLM support in German, Taack PLM announced in French,
openPLM docs in French, FreePDM mentioned in Czech) — real signal that a handful of the
English-language-originated tools (nanoPLM, Taack PLM, FreePDM, openPLM) have broader
international reach than their GitHub star counts alone suggest, rather than evidence
of parallel non-English development. The one clear exception is **CADBase**, which is
genuinely Russian in origin and whose real platform (GitLab-hosted) had been
under-documented in the English-language-only pass. The UESOFT lead (12.3) is the one
finding that could be a genuinely independent, non-English-originated project — but its
current unconfirmed/ambiguous state means it's the single highest-value lead for a
future, deeper Chinese-language research pass.

---

# 13. The OpenCascade-specific ecosystem (independent of FreeCAD branding)

OCCT (Open CASCADE Technology) is FreeCAD's geometry kernel but is also used by other,
independent applications and libraries. This section covers collaboration/PDM efforts
built around OCCT itself, not branded "FreeCAD."

## 13.1 Cascadia PLM — a real, general-purpose, actively-maintained open PLM (resolves a prior open lead)

Section 10.1 already listed `Cascadia-PLM/FreeCAD-Connector` as an unresearched lead.
This pass resolved it:

- **[Cascadia PLM](https://cascadiaplm.com/)** — "Open Source PLM for Hardware Teams."
- **Core repo:** [Cascadia-PLM/Cascadia-App](https://github.com/Cascadia-PLM/Cascadia-App)
  — described as *"a modern Digital Thread application designed to replace traditional
  low-code PLM platforms."* **27 stars, created 2026-04-19, pushed 2026-09-11** — two
  days before this research's own fetch window, i.e. genuinely actively maintained, not
  another dormant scaffold. License field shows as `NOASSERTION` on GitHub's detector
  (present but non-standard — a `LICENSING.md` exists in the repo and would need a
  manual read to confirm terms).
- **Relationship to OCCT/FreeCAD:** Cascadia-App itself appears to be a **kernel-agnostic
  PLM/digital-thread layer** — the FreeCAD-specific bridge is the separate
  `Cascadia-PLM/FreeCAD-Connector` addon (§10.1: "link documents to PLM items, vault
  check-out/check-in, property sync, guided ECO flow"). So Cascadia PLM belongs in the
  general PLM-platform category (alongside OdooPLM, §10.5) more than in the
  OCCT-specific category — noted here because that's where the resolution work
  happened, but cross-referenced from §10.1's table entry.

## 13.2 OCCT 8.0's BRepGraph — kernel-level infrastructure to watch, not a project itself

OCCT 8.0 (released 2026) introduced a new **graph-based B-Rep topology representation**
with built-in history tracking, mutation guards, and bidirectional traversal (sourced
from `github.com/Open-Cascade-SAS/OCCT` discussions #1170, #1191, #1316, and
`dev.opencascade.org/content/brepgraph-new-topology-geometry-graph-coming-occt-80`).
This is not a collaboration or PDM tool — it's kernel infrastructure — but it's
flagged here because built-in, kernel-level history tracking is a genuine enabling
precondition for a future native geometry-diff/versioning tool built directly on OCCT
rather than at the FreeCAD-file level (the approach every project in section 10.2 takes
today). Worth revisiting if any project starts building on OCCT 8.0's BRepGraph
specifically for version-control purposes.

## 13.3 Checked and ruled out (real OCCT-based projects, no collaboration/PDM layer found)

- **pythonocc / pythonocc-core** — explicitly positioned by its own community as a
  *building block* for others to build CAD/PDM/PLM apps on top of, not an application
  itself. No dedicated PDM/PLM/collaboration project was found built on it independent
  of what's already catalogued (cadracks-project's `ccad`/`cadracks-core`, §10.7).
- **CAD Assistant** (opencascade.com/products/cad-assistant/) — OPEN CASCADE SAS's own
  free viewer/converter app. Its only "collaboration" feature is exporting to 3D-PDF
  for offline review/annotation — no real-time or multi-user capability, no PDM layer.
- **SALOME platform** (CEA/EDF, built on OCCT) — a CAE/simulation-orchestration
  platform (geometry + meshing + solvers + Python API). No PDM or team-collaboration
  features found; data management is explicitly out of scope for it.
- **IfcOpenShell** — has real BCF support (BCF-XML 2.1/3.0, BCF-API 3.0, via
  `bcfxml.py`/`bcfapi.py`), but this is the same BCF standard already fully covered via
  FEP-0011/the BCF plugin research (`freecad_core_and_lens_collaboration_issues.md`
  sections 3 and 8) — no *additional* PDM/version-control layer found beyond BCF
  itself.
- **CadQuery** — no team-collaboration or version-control-specific project found
  independent of the general "version control for CAD" content already covered
  elsewhere in this document.
- **OCCT's own "collaborative development" portal** (dev.opencascade.org) — a false
  positive: "collaborative" there refers to OCCT's *own* development process (issue
  tracking, git workflow for the OCCT codebase itself), not a CAD-collaboration feature
  for OCCT's downstream users.
- **Open Cascade SAS's commercial "Platform" product** — markets PLM/ERP *integration*
  capability (linking 3D configurators to external PLM/ERP systems) but is not itself a
  PDM/PLM product — no OCCT-native PDM offering found from the company itself.

## 13.4 Unresolved sighting

- **PLMore** (`github.com/PLMore/PLMore`, "your non mega-corporate Open source PLM") —
  surfaced once during the OCCT-ecosystem search; its relationship to OCCT or FreeCAD
  was **not confirmed**. Flagged as an unresolved lead only, not a finding — do not cite
  as FreeCAD/OCCT-relevant without independent verification.

---

# 14. English terminology sweep: concurrent engineering, MBSE, EDM, and other angles not previously tried

Sections 10–11's searches used "PDM," "PLM," "collaboration," "versioning," "teamwork,"
"version control," "git," "BOM," "cloud sharing," "real-time collaboration," and
"multi-user." This pass tried the vocabulary of adjacent engineering-management
disciplines instead.

## 14.1 COMET / CDP4-COMET — a major, real concurrent-engineering platform (no confirmed FreeCAD tie, but too significant to omit)

- **[COMET](https://comet.io.esa.int/)** / **CDP4-COMET** — code:
  [STARIONGROUP/COMET-WebServices-Community-Edition](https://github.com/STARIONGROUP/COMET-WebServices-Community-Edition)
  — an open-source **Model-Based Systems Engineering (MBSE) / concurrent-engineering**
  tool implementing the **ECSS-E-TM-10-25** standard, developed by Starion Group.
  **Officially adopted by ESA in 2022** for its Concurrent Design Facility (CDF) at
  ESTEC — used to design entire space missions in 4–8 weeks via a single centralized
  shared-model repository that every engineering discipline reads from and writes to
  simultaneously.
- **No FreeCAD or OpenCascade integration was found or confirmed** in this pass. Flagged
  here anyway because it's the single most mature, standards-based, institutionally
  battle-tested "many engineers, one shared model, real-time" concurrent-engineering
  platform found anywhere in this entire research project — more mature than anything
  in sections 10–13 — and it comes from precisely the discipline (systems engineering,
  not mechanical-CAD-specific PDM) that this project's own vocabulary search never
  thought to check. Worth a dedicated future pass to see whether COMET has (or could
  have) any CAD-geometry bridge at all.

## 14.2 Anchorpoint — a commercial, explicitly-FreeCAD-targeted "git for CAD" product

- **[Anchorpoint](https://www.anchorpoint.app/blog/git-with-freecad)** — commercial, not
  open source, but its own blog post is a direct, explicit "Git with FreeCAD" workflow
  guide: automatic file-locking on `.FCStd` binaries to prevent concurrent-edit
  conflicts, layered on top of real git. This fills a specific gap that none of the
  open-source projects in section 10.2 (HistoryWorkbench, FreeCAD_gitproject,
  versioncontrol-workbench, etc.) fully solve — those focus on *diffing/history*, not
  *preventing* simultaneous conflicting edits via locking. Worth noting as evidence
  that the "lock, don't merge" approach (also seen in EasyPDM's item-locking feature,
  §10.1, and OCTC's own checkpoint-token model in the Omniverse connector,
  `freecad_core_and_lens_collaboration_issues.md` §11.1) is a recurring, independently
  arrived-at pattern across totally unrelated projects.

## 14.3 Corroboration and enrichment of existing entries

- **DocDokuPLM** (`docdoku/docdoku-plm`) — already flagged in §10.5 as an unresearched
  lead; this pass independently surfaced it again, from a completely different search
  angle ("self-hosted Autodesk Vault alternative for FreeCAD"), which corroborates that
  it's a real, recognized option in this space even though its FreeCAD-specific support
  remains unconfirmed.
- **CADCloud** — enriched in §10.3's table above with the git-fork-style federation
  detail from the project's own Open Compute Project blog post.

## 14.4 Academic lead, unconfirmed

- **rapidDCO** — "RMIT Adaptable Platform for Interactive Distributed Design,
  Customisation and Optimisation" (RMIT University) — described as a CAD/CAE research
  platform for multidisciplinary, collaborative, distributed engineering environments.
  **No FreeCAD/OpenCascade tie confirmed** in this pass; flagged as an unresearched
  academic lead only.

## 14.5 Meta-resource

- **[mlightcad/awesome-cad](https://github.com/mlightcad/awesome-cad)** — a curated,
  actively-organized list of open-source CAD projects by category. Not itself a
  collaboration project, but a resource worth re-checking on any future pass, since
  curated lists surface projects that keyword search misses.

## 14.6 Terminology angles that came up empty

"Engineering data management"/EDM, digital thread (beyond the already-known
3DfindIT/CADENAS integration), configuration management, engineering change
order/ECO, federated CAD (mostly BIM/Navisworks-specific, unrelated to FreeCAD),
design-reuse repositories (only already-known GrabCAD/TraceParts/CADBase), makerspace/
fablab shared-library management (only generic equipment-booking software, not
CAD/design management), and requirements traceability (no CAD-native tooling found at
all — the literature confirms this is generally solved, if at all, by bolting on IBM
DOORS + external version control, not CAD-native). MBSE specifically surfaced two real
open tools — **Capella** (Thales/Eclipse, Arcadia methodology) and **Virtual
Satellite** (DLR) — neither with any confirmed FreeCAD tie.

---

# 15. Updated fragmentation count after the international/OCCT/terminology sweep

Adding this round's confirmed new or newly-resolved entries — Cascadia PLM (§13.1, now
counted as a real general-PLM platform alongside OdooPLM rather than an unresolved
lead), the UESOFT/AutoPDMS lead (§12.3, counted provisionally given its ambiguous
current state), and CADBase's now-confirmed real platform location (GitLab, not a new
project, no count change) — the running total of distinct, non-interoperating
FreeCAD/OpenCascade collaboration-adjacent efforts found across this entire research
project stands at **roughly 27**, spanning at least eleven countries/language
communities (US/UK, Germany, Poland, Russia, China, Belgium/EU institutions via
Ondsel's NLnet funding, Switzerland via EPFL/ETH, and more) and two major adjacent
disciplines this research hadn't previously touched (space-systems concurrent
engineering via COMET, and plant/process-industry PDM via UESOFT's AutoPDMS).

The pattern identified in §10.8–10.9 (isolated, capable people and institutions
re-inventing the same handful of ideas without finding each other) now has
**international** confirmation, not just an English-language-GitHub-ecosystem one:
CADBase's Russian developers, UESOFT's Chinese engineers, and the ESA/Starion COMET
team all appear equally unaware of the FreeCAD-core FEP-0011 conversation, of each
other, and — per the incidental discovery in section 12.1 — even of **FreeCAD's own
16-years-earlier internal attempt** at the same problem
(`freecad_core_and_lens_collaboration_issues.md` §0). The discovery/coordination
failure this document has been tracking is not a property of the FreeCAD community
specifically — it looks structural to how open-source engineering-collaboration
tooling gets built everywhere, in every language, all the way up to space-agency scale.

---

# 16. FreePDM's founding forum thread — the complete design journal

Source: `forum.freecad.org/viewtopic.php?f=8&t=68350`, "About PDM for FreeCAD" —
**156 posts, 2022-04-28 to 2025-02-18 — essentially the entire multi-year design
journal of FreePDM (`github.com/grd/FreePDM`, §10.1), told largely in the founder's
own words,** with substantial input from a recurring cast of FreeCAD community
members, including production PDM administrators sharing real operational
experience. What follows is an organized synthesis with load-bearing quotes kept
verbatim.

## 16.1 Cast of characters

- **grd** (Gerard) — the thread's original poster and FreePDM's author/maintainer
  throughout its entire multi-year run.
- **Jee-Bee** — grd's main early collaborator; wrote much of FreePDM's initial spec
  documents (referenced throughout as `github.com/grd/FreePDM/blob/main/...`) and
  maintained a parallel exploratory fork (`github.com/Jee-Bee/FreePDM/tree/conceptod`).
- **heda** — the thread's most consistent design-philosophy voice, arguing throughout
  for flexible/configurable data structures over fixed schemas, and for keeping the
  system's core "PDM tasks" independent of FreeCAD itself; created the official
  `PDM_Roadmap` wiki placeholder page (`freecad_core_and_lens_collaboration_issues.md`
  §10) as a pointer back into this very thread.
- **user1234, adrianinsaval, Jee-Bee** — recurring technical debaters on VCS choice
  (git vs. SVN vs. database) and metadata/search requirements.
- **Zolko** — brought real production experience (a 5-team Siemens NX/TeamCenter
  project) arguing for a pure-SVN, directory-permission-based approach instead of any
  bespoke PDM software at all.
- **wmayer** — FreeCAD co-creator (Werner Mayer) — appears once, early, pointing at
  prior art (openPLM) with four separate old forum-thread links.
- **chrisb** — another long-time FreeCAD core contributor, contributes an SVN
  pre/post-commit-hook idea for stripping redundant BRep data.
- **dan-miel** — a production PDM administrator (SolidWorks PDM) who joined mid-2022,
  contributed detailed real-world PDM operational knowledge, initially tried to help
  code it, "gave up" (his words) citing database-skill limits, but kept contributing
  design feedback for years afterward. **Confirmed to be the same person as
  `DanMiel/PDMforFreeCAD`**, already independently found in §10.1's GitHub sweep as "a
  small, apparently abandoned trial" — the forum thread reveals it wasn't abandoned so
  much as folded into advising grd directly instead.
- **Nenad, Kartoffelpüre** — brought detailed comparisons to Autodesk Vault, Bentley
  MicroStation/ProjectWise, and SolidWorks/3DExperience 3DDrive's VirtualFS approach in
  later posts (2022 and 2025 respectively).
- **onekk** — flags, in 2024, a since-unlocated historical forum discussion between
  **wmayer and Jürgen Riegel** (the same Riegel from the 2009 ticket in
  `freecad_core_and_lens_collaboration_issues.md` §0) specifically about the FCStd
  zip-format's incompatibility with VCS tools — a second, independent sighting of the
  same two co-creators having thought hard about this problem, still not tracked down
  to an exact post.

## 16.2 The arc, phase by phase

**Phase 1 — Genesis and the git-vs-SVN-vs-database fight (Apr–May 2022, the bulk of
pages 1–9).** grd opens by misnaming his goal PDM vs. PLM, then proposes **Perkeep**
(formerly Camlistore, Brad Fitzpatrick's post-Google personal-data-store project,
`perkeep.org`) as a possible storage engine — a genuinely novel idea never revisited
again in the thread, abandoned after adrianinsaval and grd himself talk each other out
of it. The real fight is git vs. SVN vs. a SQL database as the versioning substrate:
user1234 makes the case (echoed by several others across dozens of posts) that
**"Something like this never work with git or similar... the diffs would be every
save too big, even for FreeCAD... It is just not practical"** and that CAD needs
**discrete states/baselines/revisions**, not line diffs. Zolko counters with a real
production pattern — SVN with per-subdirectory write permissions matching a
structured sub-assembly layout — that he'd used successfully on a 5-person Siemens NX
project specifically *because* TeamCenter was unusable for a small team. heda
contributes the most detailed early architecture sketch: Python, SQLAlchemy as a
database-agnostic middleware layer, an add-on workbench with independent tree-view,
"make all 'pdm' tasks independent of fc." grd settles, mid-fight, on **SVN + SQLite**,
with an explicit KISS "poor man's PDM" framing against SolidWorks PDM as the target
comparison, explicitly rejecting OpenPLM's complexity: **"I want to make a 'poor man's'
PDM... I don't care about fancy stuff, such as what OpenPLM is about."**

**Phase 2 — Real prior art gets surfaced, then dismissed as dead (early-mid May
2022).** wmayer (FreeCAD co-creator) posts four old forum links to **openPLM**
(the same tool independently traced in `freecad_pdm_plm_ecosystem_census.md` §10.5's
"OpenPLM lineage detail" via GitHub search — this forum thread is the primary source
those links were archived from). user1234 investigates and reports back: **"OpenPLM
looks nice, but seems death. The website does not load more and the newest commit i
found was one from 2018"** — independently confirming, in real time in 2022, the
same "unmaintained since ~2018" status this research's GitHub-side investigation later
found by checking commit dates directly. Jee-Bee separately tries to run openPLM's
code and reports dependency rot (`LEPL`, `django-south` no longer available).

**Phase 3 — Specs get written, the project gets a name and a real repo (mid-May–June
2022).** Jee-Bee and grd spend two weeks producing real specification documents
(`AttributesList.md`, `DatabaseSetup.md`, a `ConceptOfDesign` doc) at
`github.com/grd/FreePDm` — the repo the rest of this research already knew about, now
with its actual origin story attached. Real production PDM veterans weigh in with
detailed operational knowledge: **Nenad** describes an Autodesk Vault-based workflow in
full (naming schemes, check-out/check-in creating implicit versions, three-status
work/review/released states, custom searchable attributes, "copy design" as the single
most valuable PDM feature). **dan-miel** brings SolidWorks-PDM-administrator experience,
including a real, specific complication this research hadn't previously connected to
PDM at all: **FreeCAD's Toponaming problem** — updating a part in an assembly-heavy
FreeCAD project (he cites A2plus specifically) breaks constraints in ways that would
make version-to-version PDM tracking a serious headache, independent of whatever
storage/database architecture is chosen.

**Phase 4 — The pivot away from a "PDM built on a VCS" toward a bespoke filesystem
(2023).** By March 2023, grd abandons the SVN/git framing entirely in favor of a
custom **SFTP/SSH-based virtual filesystem** with manually-numbered file-revision
suffixes (`blah.FCStd.000`, `.001`, ...), directory-permission-mode tricks (`0700`
while checked out, `0555`/read-only otherwise) for check-out/check-in semantics, and
Samba/SSHFS for cross-platform file sharing. Zolko explicitly calls this out —
**"Aren't you re-inventing the wheel? This looks very much like any version control
software. Why don't you choose an existing one"** — and grd's answer amounts to "the
backend doesn't matter, I need to control the filesystem semantics myself." By
September 2023 this crystallizes into a five-level directory hierarchy design
(project → user-writable subdirs → file-level directories showing version-count and
checkout-owner → version directories → actual read-only files) that grd credits
directly to dan-miel's earlier operational input.

**Phase 5 — Language pivot and an admitted stall (2023–mid-2024).** grd hits real
tooling friction with PySide6/PyQt6/Qt6 on Linux and pivots the entire project's
implementation language, first floating a rewrite in **Nim** ("NextPDM"), landing on
**Go** instead: **"What language should I use? The answer is Go of course... That
counts too for Python, but I don't like the lack of types."** (This explains the
Python↔Go discontinuity a reader would otherwise notice comparing this thread's early
Python-centric design debate against the shipped `grd/FreePDM` repo's actual Go
implementation.) By June 2024, under direct community questioning ("How is your super
project going?"), grd admits a real stall: **"No. Unfortunately. I still accept any
code from others but I haven't done anything since. I think it is just too alien for
the python guys."** — a rare, candid admission of exactly the single-maintainer
bus-factor risk this document's §10.9 identifies as a general pattern across this
whole ecosystem, from the project's own author's mouth.

**Phase 6 — Revival, a real conference presentation, and the thread's final open
question (2025).** By February 2025, grd reports having presented FreePDM at an
actual **"FreeCAD day"** event (a real, named FreeCAD community conference — not
independently verified/dated in this pass, but a concrete claim, not vague), with a
stated one-year target for BOM/document-generation/part-attribute features. The
thread's final exchange, with Kartoffelpüre, is a detailed comparison of check-out
semantics against SolidWorks PDM, Bentley ProjectWise, and the "VirtualFS" pattern
used by OneDrive/Dropbox/Nextcloud/3DExperience 3DDrive — grd rejects VirtualFS
approaches partly on FOSS-purity grounds (OneDrive/Dropbox aren't open) and partly on
a genuine cross-platform Unix-permissions limitation he'd hit with Nextcloud. **The
thread's very last substantive question, unanswered as of the last post (2025-02-18):
"does you work base on ondsel or another Opensource PDM?"** — grd's reply sidesteps
the question entirely, answering about BOM/versioning priorities instead. This is the
closest this entire research project has found to a direct, explicit link between the
FreePDM community and the Ondsel/Lens community — a question asked, in public, and
never actually answered.

## 16.3 New tools/projects surfaced by this thread, not previously in this census

- **[Perkeep](https://perkeep.org/)** (formerly Camlistore) — Brad Fitzpatrick's
  post-Google personal-data-store project, floated by grd as a possible PDM storage
  engine in the very first post, never pursued further. Not FreeCAD-specific at all,
  but worth recording as the thread's road-not-taken.
- **realthunder's "Save as directory" branch** — mentioned by Kunda1 as an existing,
  little-known FreeCAD fork feature that unzips an `.FCStd` file into its own directory
  for git-friendliness — predates HistoryWorkbench (§10.2) by years and comes from the
  same `realthunder` whose Assembly3 fork is already well-known in the FreeCAD
  ecosystem for unrelated reasons.
- **[SnowFS](https://github.com/Snowtrack/SnowFS)** — an open-source version-control
  system purpose-built for large binary/3D-model files (not CAD-specific), flagged by
  a poster ("jackal") as potential prior art; grd examined it and concluded its
  git-like internals wouldn't suit FCStd's zip-based format. Notable as commercial
  development had reportedly diverged from the open-source line at the time of
  discussion (2023) — another instance of the "commercial takes the good parts,
  open-source original stalls" pattern already seen with GrabCAD Workbench (§10.3).
- **Commercial PDM products named as comparison points, none adopted**: **cad-plan.com's
  "Cronos"** and **pdm-studio.tech** (both mentioned once by Zolko, never followed up
  on by anyone in the thread); **Autodesk Vault**, **Bentley MicroStation/ProjectWise**,
  **3DExperience 3DDrive**, and repeated comparisons to **SolidWorks PDM**, **Siemens
  TeamCenter**, and **PTC Windchill/Pro-PDM** from posters with direct professional
  experience administering each.
- **[SparkleShare](http://www.sparkleshare.org/)** — mentioned once (2025) by
  Kartoffelpüre as an alternative "nice concept" to VirtualFS-style sync, not pursued.
- **The "Tracking FreeCAD collaboration tools" forum thread**
  (`forum.freecad.org/viewtopic.php?f=8&t=62080`, started 2021-09-09 by Kunda1) —
  a standing, continuously-updated list that ties together far more of this census
  than any single GitHub search did. Confirms **CADCloud** (§10.3) was **@vejmarie's**
  project, publicly demoed at `justyour.parts`; confirms **CollaborativeFC** (the
  archived real-time-editing attempt) was **ickby's** own project, with ickby posting
  in this very thread in 2021 inviting alpha testers with a specific reproducible test
  protocol (open a shared document in two FreeCAD instances, try every workbench
  workflow, and report any desync). The thread's "Improve/Facilitate other tools"
  section lists a much wider adjacent ecosystem than this census had otherwise
  captured: **CadQuery2 Workbench, Sverchok, BlenderBIM** (Dion Moult), **OpenIFC,
  Topologic, IFC.js**, glTF/Blender/SweetHome3D import-export bridges, and —
  notably — **Speckle**, flagged as relevant by a FreeCAD BIM contributor
  (bernd) as early as **2021**, three years before the EPFL/ETH
  `speckle-opencascade` prototype (`freecad_pdm_plm_ecosystem_census.md` §11.2)
  actually attempted to build that bridge. It also lists real, named open-hardware
  projects proposed as PDM/collaboration testcases: **Thor** (FOSS robotic arm),
  **PUMA** (3D-printed multimodality microscope), **Index Machine** (FOSS
  pick-and-place), **Monster Kossel**, **M19O2** (DIY oxygen concentrator),
  **SnakeOil-XY**, and **Cenital** (`github.com/Bibliohack/Cenital`) — real projects
  using FreeCAD at a scale where PDM-shaped problems would actually bite, independent
  of any of the PDM tools this census catalogues elsewhere.
- **`furti/FreeCAD-Reporting`** — already listed in the official Addon Manager catalog
  research (§10 intro) without much comment; this thread clarifies it was considered
  and set aside by grd specifically as a BOM/reporting building block, architecture
  (BIM/Arch)-focused rather than general-purpose.
- **The Toponaming problem, with a concrete worked example.** dan-miel's citation of a
  real forum thread (a user's A2plus/Assembly4 assembly crashing after a part
  modification, with dan-miel diagnosing exactly which constraints broke and chrisb
  pointing to "TNP") independently corroborates a specific, reproducible technical
  obstacle to any file-version-based PDM for FreeCAD: modifying a part changes its
  internal feature/face/edge naming, silently breaking any assembly constraint that
  referenced the old names — meaning a PDM's "here's version 3 of this part" isn't
  enough; the assemblies referencing it need their own re-validation pass on every
  version bump, a problem no project in this census (including FreePDM) has a solution
  for as of this research.

## 16.3a Two more independent precursor attempts, uncovered while verifying this thread's own citations

Neither of these was mentioned inside the FreePDM thread itself, but both surfaced
while tracing citations the thread and its neighbors pointed to, and both are
substantive enough to record as their own entries — not just footnotes.

**2020 — a detailed, filesystem-native PDM design proposal ("openfablab"), two years
before FreePDM existed.** Posted to the same forum thread that would later become
FreePDM's genesis site, a contributor working under the handle "openfablab" laid out
a remarkably complete design for a lightweight, **UUID-based, filesystem-native
PLM** — explicitly rejecting server-based solutions (OpenPLM et al.) as too heavy —
built around a small `info.plm` JSON sidecar file per part/assembly folder (UUID +
type + list of sub-part UUIDs), assemblies referencing sub-parts **by UUID rather
than filesystem path** (so folder reorganization never breaks a link), and a Python
script that walks a directory tree, finds every `info.plm` file, and reconstructs the
part-of hierarchy independent of physical folder layout. The proposal explicitly
reaches for a **Zoomable User Interface** (citing the open-source **EagleMode**
project) to browse the resulting hierarchy without opening every FreeCAD file. This
is, in miniature, the same "stable identity independent of file location" principle
`ondsel_project_proposal_research.md` Stage 0 recommends in the abstract for
Lens/Ondsel-Server — arrived at independently, by a different author, two years
before that document was written, and never referenced by grd or anyone else in the
later FreePDM thread despite being posted to the exact same forum section.

**2021 — a sidecar-text-file-plus-search-index proposal, with a "user1234" cameo.**
A separate contributor ("thomas-neemann") proposed pairing every FreeCAD file with a
same-named sidecar text file holding searchable metadata (status, author, etc.),
using the Linux desktop-search tool `recoll` to query across them, and demonstrated
the workflow with a video. onekk suggested SQLite instead for more robust structured
search; **the same "user1234" who would later become one of FreePDM's most prolific
early debaters** (§16.1) already appears in this thread arguing for PostgreSQL over
SQLite specifically so that check-out/write-permission locking could be modeled in
the database — the identical position user1234 would take again, at much greater
length, in the FreePDM thread six months later. This thread is the earliest trace in
this research of that specific design position being argued, and confirms user1234
was independently circulating PDM-adjacent ideas on the forum before FreePDM's
genesis thread existed to attach them to.

## 16.4 What this resolves and what it revises in earlier sections

- **Confirms** §10.9's "single-maintainer personal ecosystem" pattern with a first-hand
  admission from inside one of the projects, not just an outside inference from commit
  graphs.
- **Confirms**, independently and in 2022 real-time, the "OpenPLM is dead since ~2018"
  finding this research separately reached in §10.5 via cold GitHub archaeology in
  2026 — two completely different methods, four years apart, converging on the same
  fact.
- **Revises** the strong claim (made in §15 and elsewhere) that FreePDM and Ondsel/Lens
  are simply unaware of each other — they are aware of each other **by name** (per the
  thread's final unanswered question), even though there's no evidence of actual
  coordination. The "discovery/coordination failure" framing throughout this document
  should be read as "awareness without coordination" at minimum in this one case, not
  necessarily total mutual ignorance everywhere.
- **Adds** a concrete, FreeCAD-specific technical obstacle — the Toponaming
  problem's interaction with assembly-file PDM tracking — that neither
  `optimistic_locking_research.md`'s academic literature survey nor any other document
  in this research had connected specifically to the PDM use case before.

---

# 17. GitHub topics search — the closing pass

The one item left on the "not yet done" list after section 16: a search by GitHub
**topic tag** rather than keyword, on the theory that some addon-manager-listed
projects might tag themselves `freecad-workbench`/`freecad-addon` without using the
words "pdm" or "plm" anywhere in their name or description.

**Method:** `api.github.com/search/repositories?q=topic:X` for
`freecad-pdm`, `freecad-plm` (both **zero results** — nobody uses these as an actual
GitHub topic tag, confirming the keyword-search approach in §10 wasn't missing a
topic-tagged parallel universe), `freecad-addon` (33 repos), `freecad-workbench` (72
repos, all reviewed), and the two generic tags `pdm` (212 repos, filtered for any
FreeCAD mention) and `plm` (218 repos, same filter).

**Result: no new PDM/PLM *system* was found.** The generic `pdm`/`plm` topic filters
returned only projects already fully catalogued (nanoPLM, EasyPDM,
we-have-PDM-at-home, HistoryWorkbench) — confirming this census's keyword-based
approach in §10 had already found everything reachable by topic tag too, for the
core PDM/PLM category specifically.

Two small, genuinely new, PDM-*adjacent* finds did surface in the broader
`freecad-addon`/`freecad-workbench` topic listings (neither uses "pdm" or "plm"
anywhere in its name or description, which is exactly the blind spot this pass was
designed to catch):

- **[sksk0529/forgeshelf-workbench](https://github.com/sksk0529/forgeshelf-workbench)**
  — "CAD Asset & Log Manager for FreeCAD." 1 star, MIT, created 2026-07-01, pushed
  2026-07-23. Explicitly *not* a multi-user PDM — it's a single-user, local-first
  "remembers what you were doing" tool: per-model TODOs, quick logs, work logs, design
  notes, and print results kept together in a normal local folder. Notably borrows
  PDM/Vault terminology directly for its own single-user feature — its first public
  release is tagged `v0.1.0-local-vault`. Worth recording as evidence that "PDM-shaped
  needs" (tracking why a design decision was made, what to do next) are felt even by
  solo makers with no collaboration requirement at all, a smaller-scale version of the
  same underlying problem this whole census has been mapping.
- **[royw/freecad_datamanager_workbench](https://github.com/royw/freecad_datamanager_workbench)**
  — "A FreeCAD workbench for managing varsets and aliases." Created 2026-01-05. On
  inspection this is a parametric-variable/spreadsheet-alias manager, not a data/file
  management tool despite the name — noted here only to rule it out, since "data
  manager" plus "FreeCAD" is exactly the kind of name-collision this census has hit
  before (§10.5's `openPLM/openplm`).

**Verdict on this specific gap:** closed. The topic-tag search confirms rather than
expands the core PDM/PLM findings of §10 — the ecosystem's fragmentation isn't hiding
behind an undiscovered tagging convention, it's genuinely spread across differently-
named, differently-described, mutually unaware projects exactly as the rest of this
document has already shown.
