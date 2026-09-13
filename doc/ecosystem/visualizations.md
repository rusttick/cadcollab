# Ecosystem Relationship Mapping — Stage 4 Visualizations

Focused views generated from `ecosystem_graph.yaml` (95 nodes / 61 edges,
Stage 3 full extraction), per `ecosystem_relationship_mapping_plan.md`'s
Stage 4 instruction: multiple small purpose-built diagrams instead of one
unreadable 30+-node graph. Each diagram below covers one relationship
pattern and stays under ~12 nodes. Node/edge ids in captions refer directly
to `ecosystem_graph.yaml` so every claim here is checkable against the
source structure (and, through it, the source prose).

See `ecosystem_relationship_findings.md` for the narrative that these
diagrams support.

## 1. Personal and institutional ecosystem clusters

The `same_author`/`contributes_to` pattern: one person (or, in one case,
one organization) driving a whole cluster of interlocking projects. The
census flags this pattern at three distinct scales — shown here as three
separate small diagrams rather than one tangled fan-out graph, since mixing
them would obscure that this is the *same pattern recurring*, not one big
cluster.

### 1a. `person:pieterhijma` — individual scale, FreeCAD core

```mermaid
flowchart LR
  pieterhijma(["pieterhijma"])
  fep0011["FEP-0011\nGeneric Collaboration Framework"]
  pr26306["PR #26306\nCollaboration module"]
  issue25681["#25681 cluster\n(#25681/#25682/#25685)"]
  pr28312["PR #28312\nFile format for versioning"]
  fep0013["FEP-0013\nVisual Diffs"]

  pieterhijma --> fep0011
  pieterhijma --> pr26306
  pieterhijma --> issue25681
  pieterhijma --> pr28312
  pieterhijma --> fep0013
```

*Source: `same_author` edge, `person:pieterhijma` → 5 project nodes,
`freecad_core_and_lens_collaboration_issues.md` §1-9. Sole driver of
FreeCAD core's entire official collaboration effort — a single-person
bus-factor risk sitting at the center of the most institutionally-endorsed
initiative in the whole census.*

### 1b. `person:alekssadowski95` — individual scale, small-manufacturer PDM

```mermaid
flowchart LR
  aleks(["alekssadowski95"])
  nanoplm["nanoPLM"]
  freebom["FreeBOM"]
  wehavepdm["we-have-PDM-at-home"]
  openparts["OpenPartsLibrary\n(+ variants)"]
  pypdm["PyPDM"]

  aleks --> nanoplm
  aleks --> freebom
  aleks --> wehavepdm
  aleks --> openparts
  aleks --> pypdm
```

*Source: `same_author` edge, `person:alekssadowski95` → 5 project nodes,
`freecad_pdm_plm_ecosystem_census.md` §10.9. A second independent
same-scale personal ecosystem, unrelated to pieterhijma's, built toward
"PDM for small manufacturers" rather than FreeCAD-core collaboration.*

### 1c. `person:ose-org` (Open Source Ecology) — institutional scale

```mermaid
flowchart LR
  ose(["OpenSourceEcology\n(org, not individual)"])
  vcslib["vcs-library"]
  workbench["ose-library-workbench"]
  site["ose-library-site"]

  ose --> vcslib
  ose --> workbench
  ose --> site
```

*Source: `same_author` edge, `person:ose-org` → 3 project nodes,
`freecad_pdm_plm_ecosystem_census.md` §10.9-10.10. Same fan-out pattern as
1a/1b, but the "author" is an institution, and all three repos were pushed
coordinatedly on the same day (2026-09-07) — the pattern generalizes from
individuals to organizations.*

### 1d. Smaller `contributes_to` webs around FreePDM and Ondsel-Server

Not full same-author clusters, but worth showing as a fourth, different
shape: several people sustaining one project without having authored it.

```mermaid
flowchart LR
  grd(["grd\n(author)"]) --> freepdm["FreePDM"]
  danmiel(["dan-miel"]) -.contributes_to.-> freepdm
  heda(["heda"]) -.contributes_to.-> freepdm
  zolko(["zolko"]) -.contributes_to.-> freepdm
  user1234(["user1234"]) -.contributes_to.-> freepdm
  jeebee(["jee-bee"]) -.forked_from.-> freepdm
```

*Source: `same_author` (grd), `contributes_to` (dan-miel, heda, zolko,
user1234), and `forked_from` (jee-bee) edges all targeting
`project:freepdm`, `freecad_pdm_plm_ecosystem_census.md` §16.1-16.2. One
author, five distinct sustained non-authoring relationships — the
`contributes_to` edge type (added at the Stage 2 freeze) exists precisely
to make this shape visible instead of force-fitting everyone into
`same_author`.*

## 2. `independently_reinvents`, grouped by `shared_idea`

The plan calls this "probably the single most compelling visual this data
could produce." All 11 `independently_reinvents` edges in the graph, split
into six idea-clusters so each stays legible. Dotted arrows mark
`confidence: inferred`; solid arrows mark `confidence: confirmed`.

### 2a. Identity, central revision control, and standards — three origins, 16+ years

```mermaid
flowchart LR
  riegel2009["FreeCAD#5539 (2009)\nRiegel: UUID identity,\ncentral repo, HTTP discovery"]
  ondsel48["Ondsel-Server#48 (2025-26)\nitem/document/version/\nmeta-document vocabulary"]
  vdi["VDI 2770 / AAS\nDocument/DocumentVersion split"]
  ose["OSE vcs-library (2026)\nparts→modules→assemblies→\nstructures hierarchy"]

  riegel2009 -.->|"UUID identity + revision control"| ondsel48
  ondsel48 -->|"Document/DocumentVersion,\nconfirmed term-for-term"| vdi
  ose -.->|"same part-of hierarchy"| ondsel48
```

*Sources: `discussion:freecad-issue-5539`→`discussion:ondsel-server-48`
(inferred, `freecad_core_and_lens_collaboration_issues.md` §0);
`discussion:ondsel-server-48`→`standard:vdi-2770-aas` (**confirmed** — the
one `independently_reinvents` edge with primary-source-stated
correspondence, §5 comment 5.1); `project:ose-vcs-library`→
`discussion:ondsel-server-48` (inferred, `freecad_pdm_plm_ecosystem_census.md`
§10.10). Four independent origins — FreeCAD's founder, FreeCAD's current
team, an international standards body, and an open-hardware institution —
converging on the same core vocabulary without citing each other.*

### 2b. Sidecar identity — one person, two threads, six months apart

```mermaid
flowchart LR
  openfablab["openfablab (2020)\nUUID JSON sidecar\n(info.plm)"]
  neemann["thomas-neemann (2021)\nsidecar text file\n+ recoll search"]
  freepdm["FreePDM (2022-23)\nnumbered-file-revision\nfilesystem"]

  openfablab -.->|"identity/metadata\noutside the file"| neemann
  neemann -->|"user1234 argues\nsame PostgreSQL-locking\nposition in both threads"| freepdm
```

*Sources: `person:openfablab`→`person:thomas-neemann` (inferred,
`freecad_pdm_plm_ecosystem_census.md` §16.3a); `person:thomas-neemann`→
`project:freepdm` (**confirmed** — `person:user1234`'s continuity across
both threads is directly traceable, not inferred, §16.3a). The most
concrete "one person unknowingly re-running their own argument" case in the
census.*

### 2c. "Lock, don't merge" — hobbyist, commercial, government-funded

```mermaid
flowchart LR
  easypdm["EasyPDM\n(hobbyist, AI-assisted)\nitem check-out locking"]
  anchorpoint["Anchorpoint\n(commercial 'git for CAD')\nautomatic .FCStd locking"]
  omniverse["FreeCAD-Omniverse Connector\n(UKAEA/EUROfusion-funded)\ncheckpoint-token model"]

  easypdm -.->|"lock, don't merge"| anchorpoint
  anchorpoint -.->|"lock/tag, don't merge"| omniverse
```

*Sources: `project:easypdm`→`project:anchorpoint` and
`project:anchorpoint`→`project:freecad-omniverse-connector`, both inferred,
`freecad_pdm_plm_ecosystem_census.md` §10.1, §14.2. Same conflict-avoidance
strategy across three unrelated funding models and contexts.*

### 2d. "Make .FCStd git-friendly" — the most duplicated sub-problem in the census

```mermaid
flowchart LR
  history["HistoryWorkbench (2026)\naddon, 144 stars"]
  pr28312b["PR #28312 / FEP-0013\ncore-side"]
  reox["reox/FreeCAD_gitproject (2017)"]
  pfriedrich["versioncontrol-workbench\n(p-friedrich, 2018)"]

  history -.->|"git-friendly .FCStd,\noutside-in vs. inside-out"| pr28312b
  reox -.->|"git-based VC,\nsame year-apart pattern"| pfriedrich
```

*Sources: `project:historyworkbench`→`project:pr-28312` (inferred,
`freecad_pdm_plm_ecosystem_census.md` §10.2) and
`project:freecad-gitproject-reox`→`project:versioncontrol-workbench-pfriedrich`
(inferred, same section). Per the HistoryWorkbench node's own note, at
least five other independent attempts exist in the census
(`freecad-git-tryout-levity0815`, `cadracks-freecad-workbench-git`, plus
these two) — only the two clearest pairs are edged here to keep the
diagram legible; see §10.2 in the census for the rest.*

### 2e. Duplicate scaffolding within days

```mermaid
flowchart LR
  hitclaw["freecad-cloud-browser\n(hitclawagent, 2026-05-04)"]
  sabi["freecad-cloud-browser\n(sabi137032, 2026-05-07)"]

  hitclaw -->|"same name, same\nfeature set, 3 days apart"| sabi
```

*Source: `project:freecad-cloud-browser-hitclawagent`→
`project:freecad-cloud-browser-sabi137032` (**confirmed**,
`freecad_pdm_plm_ecosystem_census.md` §11.4). The tightest-timescale
reinvention in the census — not an idea reinvented years apart, but a
near-identical project rebuilt from scratch three days after the first one
existed.*

### 2f. Speculative overlap (lowest-confidence edge in the graph)

```mermaid
flowchart LR
  issue25681c["#25681 cluster"]
  openparts["OpenPartsLibrary"]

  issue25681c -.->|"UUID/stable-identity\npart-library layer (weak)"| openparts
```

*Source: `project:issue-25681-cluster`→`project:openpartslibrary`
(inferred, `freecad_pdm_plm_ecosystem_census.md` §10.4, §10.9). Kept
separate from the other clusters deliberately — this is the pilot's own
"test whether the taxonomy handles a lower-confidence case" edge (see
`ecosystem_relationship_mapping_results.md`), and it shouldn't be visually
conflated with the much better-evidenced clusters above.*

## 3. `technical_approach` facet overlap

A table, not a graph, per the plan's explicit instruction — better suited
to spotting overlap. Grouped by approach (rather than one 57-row × 8-column
incidence matrix) so clusters are visible at a glance; the full per-project
facet values live in `ecosystem_graph.yaml` for anyone who wants the raw
matrix. Counts sum to 58 because `project:ondsel-server` carries two
approach values.

| `technical_approach` | Count | Projects |
|---|---|---|
| `other` | 23 | fep-0011, pr-26306, fep-0013, issue-25681-cluster, bcf-plugin-freecad, ondsel-lens-addon, easypdm, pistock, cascadia-freecad-connector, cascadia-app, taack-plm, cadbaselibrary, freecad-parts-library, cadcloud, ose-vcs-library, ose-library-workbench, ose-library-site, speckle-opencascade, freecad-web-virtastic, magiknet-freecad-port, freecad-cloud-browser-hitclawagent, freecad-cloud-browser-sabi137032, ifcopenshell |
| `unknown` | 15 | nanoplm, freebom, openpartslibrary, pypdm, pdmworkbench-zhurbav, pdmforfreecad-danmiel, schumbi-freecad-plm, fc-plm-ojo42, cadracks-freecad-workbench-plm, versioncontrol-workbench-pfriedrich, billofmaterials-wb, cadracks-openplm, openplm-cpg-retail, uesoft-autopdms, plmore |
| `git` | 7 | pr-28312, gitpdm, historyworkbench, freecad-gitproject-reox, freecad-git-tryout-levity0815, cadracks-freecad-workbench-git, anchorpoint |
| `sql-database` | 4 (+ ondsel-server) | openplm, odooplm, docdoku-plm, cadracks-opm, **ondsel-server** |
| `server-checkin-checkout` | 4 (+ ondsel-server) | grabcad-workbench, freecad-omniverse-connector, comet-cdp4, **ondsel-server** |
| `p2p-crdt` | 2 | collaborativefc-ocp, jupytercad |
| `custom-filesystem-permissions` | 1 | freepdm |
| `svn` | 1 | we-have-pdm-at-home |
| `nosql-database` | 0 | *(no project in the census confirmed as using this — a gap, not a finding: several forum proposals, e.g. Malkov's 2017 NoSQL pitch, never shipped)* |

**What this surfaces:** `git` (7) and the combined
`sql-database`/`server-checkin-checkout` "central server" family (8, once
`ondsel-server`'s double-count is resolved) are the two real architectural
camps with enough members to call clusters — exactly the "filesystem-
permission-based vs. database-backed" split the plan's opening questions
anticipated, except `git` turns out to be the dominant camp, not a footnote
to either. `custom-filesystem-permissions` has exactly one member
(FreePDM itself) — its own Phase 4 pivot, per §16.2, was not joining an
existing camp but inventing a third one, alone. The `other`/`unknown`
combined total (38 of 58, roughly two-thirds) is itself a finding: the
`technical_approach` facet as currently defined is too coarse to
discriminate most of the census — see the recommendation in
`ecosystem_relationship_mapping_results.md` for whether this facet's
enumerated values need revisiting in a future pass, since this pass (Stage
4) is read-only against the frozen graph and doesn't correct it directly.
