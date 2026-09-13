# Stage B — Overlap / Similarity Ranking (full pass)

The priority deliverable per the resequenced `ecosystem_alliance_analysis.md`:
a standalone answer to "which projects overlap most in content or stated
goals," scored before any question of feasibility, cost, or who to
contact. All 129 project-pair candidates from Stage A are tiered below;
the 17 standalone revival targets aren't scored here — overlap is a
pairwise question, and a dead project's value only shows up once it's
compared against something (see the Tier S/A entries that already involve
`project:openplm`, `project:freepdm`, etc.).

## Tier S — Observed convergence (9 pairs)

An `independently_reinvents` or `shares_standard` edge exists — the graph
already records that these two arrived at, or agreed to, the same thing.
This is the strongest possible overlap signal: not inferred from facets,
but observed.

| Pair | Edge |
|---|---|
| `project:anchorpoint` ↔ `project:easypdm` | independently_reinvents |
| `project:anchorpoint` ↔ `project:freecad-omniverse-connector` | independently_reinvents |
| `project:bcf-plugin-freecad` ↔ `project:fep-0011` | shares_standard (BCF) |
| `project:freecad-cloud-browser-hitclawagent` ↔ `project:freecad-cloud-browser-sabi137032` | independently_reinvents |
| `project:freecad-gitproject-reox` ↔ `project:versioncontrol-workbench-pfriedrich` | independently_reinvents |
| `project:historyworkbench` ↔ `project:pr-28312` | independently_reinvents |
| `project:issue-25681-cluster` ↔ `project:ondsel-server` | independently_reinvents |
| `project:issue-25681-cluster` ↔ `project:openpartslibrary` | independently_reinvents |
| `project:ondsel-server` ↔ `project:ose-vcs-library` | independently_reinvents |

Read as duplicated-effort candidates, in rough order of how active both
sides currently are: `issue-25681-cluster ↔ ondsel-server` and
`ondsel-server ↔ ose-vcs-library` both have `ondsel-server` (active,
well-resourced) on one side — these are the two highest-leverage
consolidation opportunities in the whole set, since neither side needs to
be talked into caring, only introduced or formalized. The two dead-scaffold
pairs (`freecad-cloud-browser-*`, `freecad-gitproject-reox ↔
versioncontrol-workbench-pfriedrich`) are duplication of *historical*
effort — informative about how often this ecosystem reinvents the same
narrow thing, but neither side is live to consolidate with.

## Tier A — Documented awareness (6 pairs)

A `cites_as_prior_art` or `aware_of_uncoordinated` edge exists, without a
Tier S edge — they know of each other and it's on record, but not
(yet) established as the *same* idea.

| Pair | Edge |
|---|---|
| `project:cadbaselibrary` ↔ `project:fep-0011` | cites_as_prior_art |
| `project:fep-0011` ↔ `project:freepdm` | cites_as_prior_art |
| `project:fep-0011` ↔ `project:nanoplm` | cites_as_prior_art |
| `project:fep-0011` ↔ `project:ondsel-lens-addon` | cites_as_prior_art |
| `project:fep-0011` ↔ `project:taack-plm` | cites_as_prior_art |
| `project:freepdm` ↔ `project:ondsel-server` | aware_of_uncoordinated |

`project:fep-0011` dominates this tier (4 of 6 rows) simply because its
own Motivation section explicitly surveys prior art — it's the one project
in the census that already did this kind of comparison work for itself.
`freepdm ↔ ondsel-server` is the one asymmetric case: aware, unresolved,
on record in FreePDM's own thread.

## Tier B — Specific facet match, no edge (35 pairs)

Same `category` and a shared `technical_approach` more specific than
`other`/`unknown` — same kind of thing, same real mechanism, nobody's
flagged it. Full list in `ecosystem_alliance_candidates.md` §3. Two
clusters worth calling out:

- **The git/version-control cluster** (9 of the 35): `pr-28312`,
  `gitpdm`, `historyworkbench`, `freecad-gitproject-reox`,
  `freecad-git-tryout-levity0815`, `cadracks-freecad-workbench-git`,
  `anchorpoint` — nearly every pair among these seven projects. This is
  the highest-density duplication signal in Tier B: "wrap FreeCAD version
  control in git" was attempted independently at least seven times.
  Several of these arguably belong in Tier S (`independently_reinvents`)
  rather than B — see the caveat below.
- **The sql-database PLM cluster**: `openplm`, `odooplm`, `docdoku-plm`,
  `cadracks-opm` — four separate server-backed, database-driven PLM
  attempts with no cross-citation among them.

**Caveat carried over from Stage A**: several Tier B git-cluster pairs are
plausibly under-tiered — the census prose likely didn't have enough detail
to support promoting them to Tier S (`independently_reinvents`), not
because the duplication isn't real. Flagged for a possible targeted
re-read before this tier is treated as final, not fixed here.

## Tier C — Thematic match only (79 pairs)

Same `category`, but `technical_approach: other` on both sides — the
weakest tier by construction. Full list in `ecosystem_alliance_candidates.md`
§4. Treated as a background fact about the ecosystem's shape (there is a
*lot* of "same theme, unconfirmed approach" — mostly around `pdm` and
`cloud-sharing`), not a queue of 79 things to individually pursue. Per the
plan, only an unusually compelling individual Tier C case should be
hand-promoted for feasibility scoring; the tier as a whole isn't scored
further.

## What this does and doesn't tell you

This ranking is pure content overlap — it says nothing yet about whether a
pair is worth pursuing (that's the four-factor Payoff/Friction/Connection-
cost model, applied next, only to Tier S/A/B). It also doesn't yet reflect
organizational authorship: `project:openplm` (→ `organization:linobject`)
and `project:odooplm` (→ `organization:omniasolutions` /
`organization:odooplm-community-association`) now have recorded
maintaining entities in the graph, which matters once contact
identification is reached (deferred, per the plan) — it doesn't change
their overlap tier here.

## Totals

- Tier S: **9**
- Tier A: **6**
- Tier B: **35**
- Tier C: **79**
- Total pairs scored: **129** (matches Stage A's pair count exactly)

**15 pairs (Tier S + A) are immediate candidates for Stage D's
four-factor feasibility scoring.** Tier B's 35 are a secondary queue,
worth scoring if Tier S/A doesn't fill the outreach pipeline. Tier C
stays background.
