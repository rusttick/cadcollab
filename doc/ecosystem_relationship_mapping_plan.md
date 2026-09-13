# Ecosystem Relationship Mapping — Methodology and Starting Plan

Purpose: the breadth-first search (`freecad_pdm_plm_ecosystem_census.md`,
`freecad_core_and_lens_collaboration_issues.md`) answered "what exists?" —
roughly 27+ projects, plus the people, threads, and standards connecting
them. The next phase answers a different question: **how do these things
relate to each other?** This doc recommends a concrete data model and a
staged plan for building that map, rather than attempting the mapping
directly — the schema needs a pilot before it's trustworthy at scale.

## Why this needs its own structure, not just more prose

The census docs already contain a huge amount of relationship information
buried in prose — "grd examined SnowFS and rejected it," "wandererfan pointed
at the same wiki page Malkov's thread later cited," "three independent
projects converged on a UUID-sidecar-file approach without citing each
other." That information is real and already-extracted, but it's currently
only queryable by re-reading the documents. The next phase's job is to lift
those relationships out into a structure that can answer questions the prose
form can't easily answer, e.g.:

- Which projects were built by the same person, and did that person carry
  ideas or code between them?
- Which projects independently reinvented the same solution to the same
  sub-problem, and how many times did that happen?
- Which projects explicitly considered and rejected another project, and
  why? (This is more informative than a simple "aware of" edge — the reason
  is often the most reusable insight.)
- Which standards or prior-art references appear across multiple projects
  without those projects citing each other?
- Are there cluster boundaries — e.g., "everyone doing filesystem-permission-
  based PDM" vs. "everyone doing database-backed PDM" — that don't line up
  with the chronological or geographic groupings already used in the census?

None of those are answerable by grep-ing prose reliably. They're answerable
by a small graph with typed edges and a few tables.

## The data model

Three kinds of information, kept separate because they answer different
questions and get built at different times:

### 1. Node facets (attributes on each project — enables *overlap* questions)

Every project gets tagged along a fixed set of facets. Facets are what let
you ask "show me every project that does X" without needing an explicit edge
between every pair of X-doing projects.

Recommended facet schema per **project** node:

| Facet | Example values | Purpose |
|---|---|---|
| `category` (multi-valued) | `pdm`, `plm`, `version-control`, `bom`, `cloud-sharing`, `real-time-collab`, `academic-connector`, `browser-port` | The "what kind of thing is this" question the census already organizes around |
| `technical_approach` | `git`, `svn`, `custom-filesystem-permissions`, `sql-database`, `nosql-database`, `server-checkin-checkout`, `p2p-crdt`, `other` | The actual mechanism — this is where independent reinvention shows up |
| `status` | `active`, `stalled`, `dead`, `academic-prototype`, `commercial`, `revived` | As of last research pass; needs a `status_as_of` date since this changes |
| `scope` | `freecad-native`, `opencascade-generic`, `multi-cad-platform` | Distinguishes FreeCAD-specific work from broader OCCT/CAD-agnostic work |
| `origin_country` / `origin_language` | e.g. `RU`, `CN`, `US` | From the international sweep |
| `license` | SPDX id or `unlicensed`/`unknown` | Already gathered for most entries |
| `first_seen` | date | Earliest confirmed activity, not "when this research found it" |
| `source_doc` | e.g. `freecad_pdm_plm_ecosystem_census.md §16` | Where the full write-up lives — the graph should never duplicate prose, only point at it |

Add facets as you discover you need them (e.g., a `funding_source` facet
would have been useful the moment COMET/ESA and the Manchester/UKAEA
connector turned up) — this list is a starting point, not a fixed schema.

### 2. Typed edges (relationships between nodes — enables *lineage/influence* questions)

This is the part worth getting right, because a vague `related_to` edge type
throws away exactly the information that makes this phase worth doing. Below
is a starting taxonomy, derived from patterns **already observed** in the
existing research — don't invent edge types speculatively; add a new type
only when you hit a relationship the existing types can't express.

| Edge type | Direction | Meaning | Example already in the docs |
|---|---|---|---|
| `same_author` | person → project (one-to-many) | The clearest, highest-confidence edge type | pieterhijma → {FEP-0011, PR #26306, #25681 cluster, PR #28312, FEP-0013}; alekssadowski95 → {nanoPLM, FreeBOM, we-have-PDM-at-home, OpenPartsLibrary, PyPDM} |
| `cites_as_prior_art` | project A → project B | A's author/thread explicitly names B as existing work | wmayer's 2022 post citing openPLM to grd; FEP-0011's Motivation section citing CADBaseLibrary/Taack/Ondsel-Lens-Addon |
| `rejected_in_favor_of` | project A → project B, with a `reason` field | A considered B and chose a different path — the reason is the valuable part | grd examining SnowFS, concluding "its git-like internals wouldn't suit FCStd's zip-based format"; grd calling Taack PLM "alien" (Java, unknown maintainers) |
| `superseded_by` | project A → project B (temporal) | A's own conversation treats B as its resolution | FreeCAD issue #5539 (2009) → FreeCAD Parts Library ("closed as replaced by Component Library") |
| `independently_reinvents` | project A ↔ project B, with a `shared_idea` field | A and B solve the same sub-problem the same way, with no citation between them | openfablab's 2020 `info.plm` UUID-sidecar idea ↔ thomas-neemann's 2021 sidecar-text-file idea ↔ FreePDM's eventual numbered-file-revision filesystem (2023) — three independent convergences on "stable identity outside the file, in a sidecar" |
| `aware_of_uncoordinated` | project A ↔ project B | Explicit mention exists but no evidence of follow-through | FreePDM's own thread, final question: "does you work base on ondsel or another Opensource PDM?" — never actually answered |
| `shares_standard` | project A ↔ project B, with a `standard` field | Both adopt/reference the same external standard | FEP-0011 ↔ podestplatz/BCF-Plugin-FreeCAD (both BCF); the AAS/VDI-2770 mapping in `ondsel_48_research.md` ↔ Ondsel-Server#48 |
| `mentors` / `mentored_by` | person → person, scoped to a project | Distinct from `same_author` — captures the GSoC-style relationships | yorikvanhavre, hardeeprai → pPodest (BCF plugin, GSoC 2019) |
| `discussed_in` | project ↔ discussion (forum thread / GitHub issue) | Ties a project to its primary-source origin thread | FreePDM ↔ forum t=68350; BCF plugin ↔ forum t=35465 |
| `forked_from` / `mirror_of` | project A → project B | Literal git relationship, when one exists | CADBase's GitHub repo → its canonical GitLab home |

Every edge should carry:
- `confidence`: `confirmed` (directly stated in a primary source) vs.
  `inferred` (your own pattern-matching — e.g., "independently reinvents" is
  almost always inferred, not stated)
- `source`: which doc/section the claim traces to, so the edge is never more
  authoritative than the prose backing it

### 3. Non-project nodes worth including

Don't limit nodes to projects. The research already shows that **people**,
**discussion threads**, and **standards** are themselves load-bearing nodes
— several of the most interesting findings (the pieterhijma cluster, the
alekssadowski95 cluster, the OSE institutional cluster, the recurring
FreeCAD-co-creator cameos across 2009/2010/2014/2017) are really statements
about *people*, not projects. Recommended additional node types:

- **`person`** — id, handle(s) across platforms (a person may post as
  different names/emails across forum vs. GitHub — e.g., confirm dan-miel ==
  DanMiel/PDMforFreeCAD, as already done), known affiliations
- **`discussion`** — forum threads and GitHub issues that function as the
  connective tissue between projects (the 2009 ticket, the 2022 FreePDM
  thread, the 2019 BCF proposal thread) — these deserve their own nodes
  because multiple projects/people attach to the same thread
- **`standard`** — BCF, VDI 2770, AAS, IEC 81346 — external reference points
  multiple projects converge on (or fail to converge on) independently

## A worked example, using real data already gathered

To make the schema concrete before committing to it at scale, here's what
three real nodes and their edges would look like:

```yaml
nodes:
  - id: person:pieterhijma
    type: person
    handles: {github: pieterhijma}
    note: "Sole driver of FreeCAD core's official collaboration effort"

  - id: project:fep-0011
    type: project
    name: "FEP-0011 Generic Collaboration Framework"
    category: [pdm, real-time-collab]
    technical_approach: [other]
    status: active
    scope: freecad-native
    first_seen: 2026-01-25
    source_doc: "freecad_core_and_lens_collaboration_issues.md §8.3"

  - id: project:pr-26306
    type: project
    name: "Collaboration module (PR #26306)"
    category: [pdm]
    status: active
    scope: freecad-native
    source_doc: "freecad_core_and_lens_collaboration_issues.md §8.5"

  - id: project:freepdm
    type: project
    name: "FreePDM"
    category: [pdm]
    technical_approach: [custom-filesystem-permissions]
    status: revived
    scope: freecad-native
    first_seen: 2022-04-28
    source_doc: "freecad_pdm_plm_ecosystem_census.md §16"

  - id: standard:bcf
    type: standard
    name: "BIM Collaboration Format"

edges:
  - type: same_author
    from: person:pieterhijma
    to: [project:fep-0011, project:pr-26306]
    confidence: confirmed

  - type: shares_standard
    from: project:fep-0011
    to: project:freepdm
    standard: standard:bcf
    note: "FEP-0011 explicitly aligns with BCF v3.0; FreePDM's community
           separately considered and dropped BCF-plugin-style approaches"
    confidence: confirmed
    source: "freecad_core_and_lens_collaboration_issues.md §8.3 (Rationale)"

  - type: aware_of_uncoordinated
    from: project:freepdm
    to: project:fep-0011
    note: >
      No direct link found, but FreePDM's own thread shows its author (grd)
      aware of 'ondsel' by name (§16.4) — a lower-confidence but plausible
      extension is that FEP-0011's ecosystem awareness runs the same
      direction. Mark as inferred, not confirmed, until a direct citation
      is found.
    confidence: inferred
```

This is deliberately small. The point of the worked example is to check the
schema handles a case you already understand deeply *before* running it
across 27+ projects you understand less deeply.

## Recommended file format and location

- **One file**: `doc/ecosystem_graph.yaml` (nodes and edges together — at
  this scale, two files just adds a join step for no benefit). YAML over
  JSON for human diffability and inline comments (the `note` fields above
  are half the value).
- Keep it **strictly pointer-based** into the existing prose docs via
  `source_doc`/`source` fields — never duplicate a quote or a fact into the
  graph file that already lives in a census doc. The graph's job is
  structure, not content.
- Don't build a rendering/query script yet. At ~30 nodes and maybe 60-100
  edges, this is small enough to read directly or paste into a prompt for
  ad hoc questions ("show me every `independently_reinvents` edge") long
  before a script pays for itself. Revisit tooling if the graph grows past
  a size where that stops being true.

## Recommended staged plan

**Stage 1 — Pilot (8-10 nodes).** Don't start with all 27+ projects. Pick a
small set that's already deeply understood and known to be
richly-interconnected: the pieterhijma cluster (FEP-0011, PR #26306, PR
#28312, FEP-0013, the #25681 issue cluster), FreePDM, openPLM, and Ondsel/
Lens. Fully populate their facets and edges. This validates the schema
against real complexity (a person with 5 projects, a project with both
`rejected_in_favor_of` and `shares_standard` edges to different targets)
before the taxonomy calcifies.

**Stage 2 — Taxonomy review.** After the pilot, check which edge types
actually got used, which felt forced, and which relationships you wanted to
express but couldn't. Adjust the taxonomy once, deliberately, rather than
letting it drift edge-by-edge across the full extraction pass. Freeze it as
"v1" and note the freeze date.

**Stage 3 — Full extraction pass.** Go through
`freecad_pdm_plm_ecosystem_census.md` and
`freecad_core_and_lens_collaboration_issues.md` section by section (they're
already organized as one project/topic per subsection, which maps cleanly
onto "one extraction pass per node"). This is a rereading-and-structuring
task, not new research — no web searches should be needed for this stage.

**Stage 4 — Focused visualizations, not one big graph.** A single diagram
with 30+ nodes and 100 edges is unreadable. Instead, generate multiple
small, purpose-built views as needed:
- A `same_author` cluster diagram (shows the "personal ecosystem" pattern
  from census §16 §10.9 visually)
- An `independently_reinvents` diagram grouped by `shared_idea` (shows how
  many times the same idea was arrived at separately — this is probably the
  single most compelling visual this data could produce)
- A `technical_approach` facet matrix (projects × approach, as a table, not
  a graph — better suited to spotting overlap than a node-edge diagram)

Mermaid (`graph`/`flowchart` syntax) renders natively in this environment
and is the right tool once there's a specific sub-graph worth drawing —
don't reach for it before Stage 3 produces something worth visualizing.

**Stage 5 — A synthesis document.** Once the graph exists, write
`ecosystem_relationship_findings.md` (or similar) as a *prose* document —
same register as the rest of this research — that narrates what the
structured data actually revealed, backed by specific edges as citations.
The graph file itself stays as supporting data, not the deliverable; the
deliverable is what it lets you say that you couldn't say before.

## Open questions to settle before Stage 1 (your call, not mine to assume)

- **Should low-confidence `independently_reinvents` edges require a written
  rationale, or is a one-line note enough?** Recommend requiring rationale —
  this edge type is doing the most interpretive work and is the easiest to
  overclaim.
- **Should dead/abandoned projects get full facet treatment, or a lighter
  stub?** Recommend full treatment for anything with more than a few lines
  of write-up in the census (most of them), stub-only for the single-commit
  scaffolds.
- **Should this graph track confidence/date the way the prose docs do**
  (e.g., "as of 2026-09-13")? Recommend yes on `status` specifically, since
  that's the facet most likely to go stale.
