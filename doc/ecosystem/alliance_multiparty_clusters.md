# Multi-Party Convergence Analysis

Extends the pairwise Stage D/E/F work to ask: where do **three or more**
projects converge, not just two? Method: treat the overlap-tier edges from
`ecosystem_alliance_overlap_ranking.md` as a graph and find connected
components of size ≥ 3, computed separately per tier so a genuine N-way
convergence isn't confused with a "one comprehensive project cites
everyone" hub effect — those are different findings and need different
outreach framing.

## The two genuine multi-party clusters (Tier S only — observed convergence)

Restricting to `independently_reinvents`/`shares_standard` edges only
(the strongest, observed-not-inferred signal) produces exactly two
clusters of size ≥ 3, and nothing larger:

### Cluster 1 — Stable item/part identity vocabulary (4 projects)

`project:issue-25681-cluster` — `project:ondsel-server` — `project:ose-vcs-library`,
plus `project:issue-25681-cluster` — `project:openpartslibrary`

Four independent origins converging on the same core idea (UUID-based
stable identity, a revision/version chain, an item→assembly hierarchy):
FreeCAD core's own official collaboration effort (pieterhijma),
Ondsel's commercial Lens product, Open Source Ecology's institutional
part-library, and alekssadowski95's hobby OpenPartsLibrary project.

**Structure matters here**: this isn't a symmetric 4-way clique — it's a
chain through `ondsel-server` and `issue-25681-cluster` as the two hubs,
each carrying one strong dyad and one weaker one:

| Sub-pair | Prior tier/score (from Stage D/E) |
|---|---|
| `issue-25681-cluster` ↔ `ondsel-server` | **Tier 2** — strongest pair in the whole analysis |
| `ondsel-server` ↔ `ose-vcs-library` | **Tier 2** |
| `issue-25681-cluster` ↔ `openpartslibrary` | **Tier 4** — the underlying edge is itself flagged speculative in the graph |

**Recommended framing**: don't propose a single 4-way meeting. Start the
`issue-25681-cluster`/`ondsel-server`/`ose-vcs-library` conversation as
already planned (Tier 2, Stage F) — this core trio is strong on its own —
and treat `openpartslibrary` as an optional fourth invite once that
conversation has real momentum and the speculative edge has been checked
against openpartslibrary's actual design. Overselling the openpartslibrary
link risks diluting a strong 3-way case with a weak 4th.

### Cluster 2 — "Lock, don't merge" conflict strategy (3 projects)

`project:anchorpoint` — `project:easypdm`, and
`project:anchorpoint` — `project:freecad-omniverse-connector`

Three independent origins landing on the same conflict-avoidance strategy
(lock/checkpoint rather than merge): a commercial "git for CAD" product,
a solo AI-assisted hobbyist tool, and EU/UKAEA-funded fusion-research
infrastructure. `project:anchorpoint` is the hub connecting both dyads —
`easypdm` and `freecad-omniverse-connector` have no direct signal between
themselves, only through Anchorpoint.

| Sub-pair | Prior tier/score |
|---|---|
| `anchorpoint` ↔ `easypdm` | **Tier 1** — easy, low-cost |
| `anchorpoint` ↔ `freecad-omniverse-connector` | **Tier 4** — real convergence, but institutionally the most distant pairing found anywhere in this analysis |

**Recommended framing**: same pattern as Cluster 1 — this reads better as
two separate conversations than one 3-way pitch. Anchorpoint is genuinely
the only party positioned to see the whole picture (it's the hub); a
plausible move is telling Anchorpoint itself "you're not the only one who
solved this the same way" and letting them decide whether cross-domain
outreach to a fusion-research project is worth their time, rather than you
brokering a 3-way conversation between parties who have nothing else in
common.

## Not a genuine cluster: the FEP-0011 hub (Tier S+A combined)

Adding Tier A (`cites_as_prior_art`/`aware_of_uncoordinated`) edges merges
Cluster 1 into an 11-project blob: `bcf-plugin-freecad`, `cadbaselibrary`,
`fep-0011`, `freepdm`, `issue-25681-cluster`, `nanoplm`,
`ondsel-lens-addon`, `ondsel-server`, `openpartslibrary`, `ose-vcs-library`,
`taack-plm`. **This is not a convergence cluster and shouldn't be read as
one** — it's a star graph. `fep-0011` alone accounts for 5 of the 6 Tier A
edges holding it together, because FEP-0011's Motivation section is the
one place in the whole census that already did a systematic prior-art
survey. The other 10 projects mostly don't know about each other; they're
each individually known to FEP-0011.

**What this *is* useful for**: FEP-0011 (and by extension `person:
pieterhijma`, already flagged for high fan-out/possible capacity limits)
is the one plausible convening point for a broader multi-party working
group, precisely because it's the only node with an existing relationship
to nearly everyone else in this list. If a genuine multi-party
consolidation effort beyond Clusters 1-2 is wanted later, this is the
node to build it through — but that's a deliberate convening decision,
not something the data forces, and it further loads a person already
flagged as possibly over-extended.

## A third, lower-confidence cluster type: shared mechanism, no signal edge (Tier B)

Two dense clusters exist purely on shared `category`+`technical_approach`,
with **no** `independently_reinvents`/citation edge connecting any pair —
weaker evidence than Clusters 1-2, but large enough to be worth naming:

- **The git/version-control cluster (7 projects)**: `anchorpoint`,
  `cadracks-freecad-workbench-git`, `freecad-git-tryout-levity0815`,
  `freecad-gitproject-reox`, `gitpdm`, `historyworkbench`, `pr-28312`.
  Already flagged in Stage A as likely under-tiered — several of these
  pairs probably deserved `independently_reinvents` edges that the census
  prose didn't have enough detail to support. The two live, high-value
  members (`historyworkbench`, `pr-28312`) are already a Tier 2 pair in
  their own right (Stage E); the other five are mostly dead scaffolds
  whose only remaining value is as evidence of how many times this exact
  idea got attempted, not as active parties to include in outreach.
- **The server-backed PDM/PLM cluster (8 projects)**: `cadracks-opm`,
  `comet-cdp4`, `docdoku-plm`, `freecad-omniverse-connector`,
  `grabcad-workbench`, `odooplm`, `ondsel-server`, `openplm`. The
  "central database + check-in/check-out" architecture camp. Notably,
  `ondsel-server` and `freecad-omniverse-connector` both appear here *and*
  in the Tier S clusters above — they're each simultaneously part of a
  strong observed-convergence pair and a weaker facet-only cluster,
  meaning they're the two most "architecturally central" projects in the
  whole dataset. `openplm` is already Tier 0 (not recommended) from Stage
  D and shouldn't be revived just because it shares an architecture family
  with active projects.

Neither Tier B cluster is recommended for direct multi-party outreach as a
group — per the plan's own caution about Tier B/C, these are background
facts about the ecosystem's shape (there's a real "central-server camp"
and a real "git-wrapper camp"), useful context for framing the Tier
S-based conversations above, not standalone action items.

## Summary

**Two real 3+/4-way convergence clusters exist**, both hub-shaped rather
than fully symmetric, both already partially covered by existing Tier 1/2
pairwise proposals in `ecosystem_alliance_proposals.md`. Recommended
action: run the strong dyads within each cluster as already planned, and
treat the weak member of each (`openpartslibrary` in Cluster 1,
`freecad-omniverse-connector` in Cluster 2) as an opportunistic addition
once the core conversation has traction — not a simultaneous 4-way or
3-way pitch from a cold start.
