# Stage A — Candidate Generation (raw output)

Mechanical output of Stage A per `ecosystem_alliance_analysis.md`. This is a
query result, not an analysis: every entry below satisfies one of the three
Stage A enumeration rules, and nothing has been scored, ranked, or filtered
for plausibility yet. That's Stage B/C's job. Project/person/discussion ids
match `ecosystem_graph.yaml` exactly so each row is directly checkable.

One data gap surfaced while resolving edges through discussion nodes:
`discussion:ondsel-server-48` has no explicit `discussed_in` edge to
`project:ondsel-server` in the graph, despite the discussion's own title
("Extent to which Lens is a PDM") and prior narration making the
association clear. This candidate generation pass manually treats that
link as present for resolution purposes only — the graph file itself
hasn't been touched, per the plan's "Stage 4/5 are read-only" constraint
carrying forward. Worth a one-line fix to the graph itself at some point,
but not blocking here.

## 1. Edge-triggered project pairs (15)

Directly connected by a qualifying signal edge, or connected once a
`discussion`/`person` endpoint is resolved to the project(s) it belongs to
(e.g. an `independently_reinvents` edge between two discussion nodes
becomes a pair between the projects those discussions are about). Highest
confidence input to Stage B — these aren't guesses, they're graph facts.

| Pair | Trigger edge type(s) |
|---|---|
| `project:anchorpoint` ↔ `project:easypdm` | `independently_reinvents` |
| `project:anchorpoint` ↔ `project:freecad-omniverse-connector` | `independently_reinvents` |
| `project:bcf-plugin-freecad` ↔ `project:fep-0011` | `cites_as_prior_art`, `shares_standard` |
| `project:cadbaselibrary` ↔ `project:fep-0011` | `cites_as_prior_art` |
| `project:fep-0011` ↔ `project:freepdm` | `cites_as_prior_art` |
| `project:fep-0011` ↔ `project:nanoplm` | `cites_as_prior_art` |
| `project:fep-0011` ↔ `project:ondsel-lens-addon` | `cites_as_prior_art` |
| `project:fep-0011` ↔ `project:taack-plm` | `cites_as_prior_art` |
| `project:freecad-cloud-browser-hitclawagent` ↔ `project:freecad-cloud-browser-sabi137032` | `independently_reinvents` |
| `project:freecad-gitproject-reox` ↔ `project:versioncontrol-workbench-pfriedrich` | `independently_reinvents` |
| `project:freepdm` ↔ `project:ondsel-server` | `aware_of_uncoordinated` |
| `project:historyworkbench` ↔ `project:pr-28312` | `independently_reinvents` |
| `project:issue-25681-cluster` ↔ `project:ondsel-server` | `cites_as_prior_art`, `independently_reinvents` |
| `project:issue-25681-cluster` ↔ `project:openpartslibrary` | `independently_reinvents` |
| `project:ondsel-server` ↔ `project:ose-vcs-library` | `independently_reinvents` |

## 2. Flags — signal edges that name a project but don't form a pair (11)

Not candidates in themselves, but relevant context that Stage B/C should
check when scoring a candidate touching these projects (especially the
`rejected_in_favor_of` and `cites_as_prior_art` rows below, which bear
directly on `project:openplm` as a revival target — see §4).

| Edge type | From | To | Note |
|---|---|---|---|
| `cites_as_prior_art` | `person:wmayer` | `project:openplm` | wmayer posted four old forum links to openPLM inside the FreePDM thread |
| `cites_as_prior_art` | `person:openfablab` | `project:openplm` | openfablab's 2020 proposal explicitly rejects server-based solutions "(OpenPLM et al.)" as too heavy |
| `rejected_in_favor_of` | `person:openfablab` | `project:openplm` | same rejection, typed as rejection rather than citation |
| `rejected_in_favor_of` | `project:freepdm` | *(bespoke — SnowFS)* | git-like internals wouldn't suit FCStd's zip-based format |
| `rejected_in_favor_of` | `project:freepdm` | *(bespoke — OpenPLM)* | "I don't care about fancy stuff, such as what OpenPLM is about" |
| `rejected_in_favor_of` | `project:freepdm` | *(bespoke — no named target)* | abandoned SVN/git-as-substrate framing; needed to control filesystem semantics directly |
| `independently_reinvents` | `person:openfablab` | `person:thomas-neemann` | the sidecar-identity chain's first link |
| `independently_reinvents` | `person:thomas-neemann` | `project:freepdm` | user1234's cross-thread continuity (confirmed, not inferred) |
| `independently_reinvents` | `discussion:ondsel-server-48` | `standard:vdi-2770-aas` | confirmed — term-for-term correspondence stated in a primary source |
| `shares_standard` | `discussion:ondsel-server-48` | `standard:vdi-2770-aas` | same standard alignment |
| `cites_as_prior_art` | `project:ifcopenshell` | `standard:bcf` | standard citation, not a project |

`project:openplm` is named in three of these rows as something explicitly
rejected by two different people (wmayer implicitly, openfablab
explicitly) — worth weighing heavily as Friction if `openplm` shows up as a
revival candidate in Stage B.

## 3. Facet-matched pairs, no existing edge — strong (35)

Same `category` *and* a shared `technical_approach` value more specific
than `other` (git, sql-database, server-checkin-checkout, svn,
custom-filesystem-permissions, p2p-crdt, nosql-database, or
server-checkin-checkout). These are "should probably know about each
other" candidates with no citation, awareness, or reinvention edge on
record.

| Pair | Shared category | Shared approach |
|---|---|---|
| `project:pr-28312` ↔ `project:gitpdm` | version-control | git |
| `project:pr-28312` ↔ `project:freecad-gitproject-reox` | version-control | git |
| `project:pr-28312` ↔ `project:freecad-git-tryout-levity0815` | version-control | git |
| `project:pr-28312` ↔ `project:cadracks-freecad-workbench-git` | version-control | git |
| `project:pr-28312` ↔ `project:anchorpoint` | version-control | git |
| `project:openplm` ↔ `project:odooplm` | plm | sql-database |
| `project:openplm` ↔ `project:docdoku-plm` | plm | sql-database |
| `project:openplm` ↔ `project:cadracks-opm` | plm | sql-database |
| `project:ondsel-server` ↔ `project:grabcad-workbench` | cloud-sharing, pdm | server-checkin-checkout |
| `project:ondsel-server` ↔ `project:odooplm` | pdm | sql-database |
| `project:ondsel-server` ↔ `project:cadracks-opm` | pdm | sql-database |
| `project:ondsel-server` ↔ `project:freecad-omniverse-connector` | cloud-sharing, real-time-collab | server-checkin-checkout |
| `project:ondsel-server` ↔ `project:comet-cdp4` | pdm, real-time-collab | server-checkin-checkout |
| `project:gitpdm` ↔ `project:historyworkbench` | version-control | git |
| `project:gitpdm` ↔ `project:freecad-gitproject-reox` | version-control | git |
| `project:gitpdm` ↔ `project:freecad-git-tryout-levity0815` | version-control | git |
| `project:gitpdm` ↔ `project:cadracks-freecad-workbench-git` | version-control | git |
| `project:gitpdm` ↔ `project:anchorpoint` | pdm, version-control | git |
| `project:historyworkbench` ↔ `project:freecad-gitproject-reox` | version-control | git |
| `project:historyworkbench` ↔ `project:freecad-git-tryout-levity0815` | version-control | git |
| `project:historyworkbench` ↔ `project:cadracks-freecad-workbench-git` | version-control | git |
| `project:historyworkbench` ↔ `project:anchorpoint` | version-control | git |
| `project:freecad-gitproject-reox` ↔ `project:freecad-git-tryout-levity0815` | version-control | git |
| `project:freecad-gitproject-reox` ↔ `project:cadracks-freecad-workbench-git` | version-control | git |
| `project:freecad-gitproject-reox` ↔ `project:anchorpoint` | version-control | git |
| `project:freecad-git-tryout-levity0815` ↔ `project:cadracks-freecad-workbench-git` | version-control | git |
| `project:freecad-git-tryout-levity0815` ↔ `project:anchorpoint` | version-control | git |
| `project:cadracks-freecad-workbench-git` ↔ `project:anchorpoint` | version-control | git |
| `project:grabcad-workbench` ↔ `project:freecad-omniverse-connector` | cloud-sharing | server-checkin-checkout |
| `project:grabcad-workbench` ↔ `project:comet-cdp4` | pdm | server-checkin-checkout |
| `project:odooplm` ↔ `project:docdoku-plm` | plm | sql-database |
| `project:odooplm` ↔ `project:cadracks-opm` | pdm, plm | sql-database |
| `project:docdoku-plm` ↔ `project:cadracks-opm` | plm | sql-database |
| `project:collaborativefc-ocp` ↔ `project:jupytercad` | real-time-collab | p2p-crdt |
| `project:freecad-omniverse-connector` ↔ `project:comet-cdp4` | real-time-collab | server-checkin-checkout |

Note the git/version-control cluster (rows 1-9 and the FreeCAD-native
scaffolds below them) is large mostly because "git wrapper for FreeCAD" was
attempted many times independently with almost no facet detail beyond
that — several of these should probably have been `independently_reinvents`
edges in Stage 3 and weren't, because the census prose didn't narrate them
in enough detail to support that stronger, more specific edge type. Worth
a second look before Stage B treats all nine as equally weighted.

## 4. Facet-matched pairs, no existing edge — weak (79)

Same `category` and both sides tagged `technical_approach: other` — the
catch-all bucket. `other` is not a real discriminating signal (it just
means "not git/svn/sql/nosql/p2p-crdt/filesystem-permissions/server-
checkin-checkout"), so a shared-category-only match here is much weaker
evidence than §3. Included for completeness per the enumeration rule, but
Stage B should treat this whole bucket as lower-confidence by construction,
not evaluate it item-by-item at the same depth as §1-3. Full list kept for
traceability:

`project:fep-0011`↔`project:pr-26306`, `project:fep-0011`↔`project:issue-25681-cluster`,
`project:fep-0011`↔`project:easypdm`, `project:fep-0011`↔`project:pistock`,
`project:fep-0011`↔`project:cascadia-freecad-connector`, `project:fep-0011`↔`project:ose-vcs-library`,
`project:fep-0011`↔`project:ose-library-workbench`, `project:fep-0011`↔`project:ifcopenshell`,
`project:pr-26306`↔`project:issue-25681-cluster`, `project:pr-26306`↔`project:easypdm`,
`project:pr-26306`↔`project:pistock`, `project:pr-26306`↔`project:cascadia-freecad-connector`,
`project:pr-26306`↔`project:cadbaselibrary`, `project:pr-26306`↔`project:ose-vcs-library`,
`project:pr-26306`↔`project:ose-library-workbench`, `project:issue-25681-cluster`↔`project:bcf-plugin-freecad`,
`project:issue-25681-cluster`↔`project:easypdm`, `project:issue-25681-cluster`↔`project:pistock`,
`project:issue-25681-cluster`↔`project:cascadia-freecad-connector`, `project:issue-25681-cluster`↔`project:cadbaselibrary`,
`project:issue-25681-cluster`↔`project:ose-vcs-library`, `project:issue-25681-cluster`↔`project:ose-library-workbench`,
`project:issue-25681-cluster`↔`project:ifcopenshell`, `project:bcf-plugin-freecad`↔`project:easypdm`,
`project:bcf-plugin-freecad`↔`project:cadbaselibrary`, `project:bcf-plugin-freecad`↔`project:freecad-parts-library`,
`project:bcf-plugin-freecad`↔`project:ose-vcs-library`, `project:bcf-plugin-freecad`↔`project:ose-library-site`,
`project:bcf-plugin-freecad`↔`project:ifcopenshell`, `project:ondsel-lens-addon`↔`project:cadbaselibrary`,
`project:ondsel-lens-addon`↔`project:cadcloud`, `project:ondsel-lens-addon`↔`project:ose-library-site`,
`project:ondsel-lens-addon`↔`project:speckle-opencascade`, `project:ondsel-lens-addon`↔`project:freecad-cloud-browser-hitclawagent`,
`project:ondsel-lens-addon`↔`project:freecad-cloud-browser-sabi137032`, `project:easypdm`↔`project:pistock`,
`project:easypdm`↔`project:cascadia-freecad-connector`, `project:easypdm`↔`project:cadbaselibrary`,
`project:easypdm`↔`project:freecad-parts-library`, `project:easypdm`↔`project:ose-vcs-library`,
`project:easypdm`↔`project:ose-library-workbench`, `project:easypdm`↔`project:ose-library-site`,
`project:easypdm`↔`project:ifcopenshell`, `project:pistock`↔`project:cascadia-freecad-connector`,
`project:pistock`↔`project:cascadia-app`, `project:pistock`↔`project:taack-plm`,
`project:pistock`↔`project:cadbaselibrary`, `project:pistock`↔`project:ose-vcs-library`,
`project:pistock`↔`project:ose-library-workbench`, `project:cascadia-freecad-connector`↔`project:cadbaselibrary`,
`project:cascadia-freecad-connector`↔`project:ose-vcs-library`, `project:cascadia-freecad-connector`↔`project:ose-library-workbench`,
`project:cascadia-app`↔`project:taack-plm`, `project:cadbaselibrary`↔`project:freecad-parts-library`,
`project:cadbaselibrary`↔`project:cadcloud`, `project:cadbaselibrary`↔`project:ose-vcs-library`,
`project:cadbaselibrary`↔`project:ose-library-workbench`, `project:cadbaselibrary`↔`project:ose-library-site`,
`project:cadbaselibrary`↔`project:speckle-opencascade`, `project:cadbaselibrary`↔`project:freecad-cloud-browser-hitclawagent`,
`project:cadbaselibrary`↔`project:freecad-cloud-browser-sabi137032`, `project:cadbaselibrary`↔`project:ifcopenshell`,
`project:freecad-parts-library`↔`project:ose-vcs-library`, `project:freecad-parts-library`↔`project:ose-library-site`,
`project:freecad-parts-library`↔`project:ifcopenshell`, `project:cadcloud`↔`project:ose-library-site`,
`project:cadcloud`↔`project:speckle-opencascade`, `project:cadcloud`↔`project:freecad-cloud-browser-hitclawagent`,
`project:cadcloud`↔`project:freecad-cloud-browser-sabi137032`, `project:ose-vcs-library`↔`project:ose-library-workbench`,
`project:ose-vcs-library`↔`project:ose-library-site`, `project:ose-vcs-library`↔`project:ifcopenshell`,
`project:ose-library-site`↔`project:speckle-opencascade`, `project:ose-library-site`↔`project:freecad-cloud-browser-hitclawagent`,
`project:ose-library-site`↔`project:freecad-cloud-browser-sabi137032`, `project:ose-library-site`↔`project:ifcopenshell`,
`project:speckle-opencascade`↔`project:freecad-cloud-browser-hitclawagent`, `project:speckle-opencascade`↔`project:freecad-cloud-browser-sabi137032`,
`project:freecad-web-virtastic`↔`project:magiknet-freecad-port`

## 5. Revival targets (17)

Every project at `status: stalled` or `status: dead`, per the plan's rule
— no filtering yet on whether a viable successor or contact exists; that's
Stage B's Payoff check and the "Identifying who to contact" step.

| Project | Category |
|---|---|
| `project:bcf-plugin-freecad` | real-time-collab, bom |
| `project:cadcloud` | cloud-sharing |
| `project:cadracks-freecad-workbench-git` | version-control |
| `project:cadracks-freecad-workbench-plm` | plm |
| `project:cadracks-openplm` | plm |
| `project:cadracks-opm` | pdm, plm |
| `project:collaborativefc-ocp` | real-time-collab |
| `project:fc-plm-ojo42` | plm |
| `project:freecad-cloud-browser-hitclawagent` | cloud-sharing |
| `project:freecad-git-tryout-levity0815` | version-control |
| `project:freecad-gitproject-reox` | version-control |
| `project:grabcad-workbench` | pdm, cloud-sharing |
| `project:openplm` | plm |
| `project:pdmforfreecad-danmiel` | pdm |
| `project:pdmworkbench-zhurbav` | pdm |
| `project:versioncontrol-workbench-pfriedrich` | version-control |
| `project:we-have-pdm-at-home` | pdm |

## 6. Known cases not captured by the mechanical rules above

Three cases the alliance-analysis plan names as required Stage B pilot
material don't actually fall out of §1-5, because they're shaped
differently than a project pair or a single-project revival target. Listed
here directly from the existing graph/findings docs (no new research) so
Stage B has real material for them:

- **The sidecar-identity idea lineage**: `person:openfablab` (2020) →
  `person:thomas-neemann` (2021) → `project:freepdm` (2023), all
  `independently_reinvents`. Neither `openfablab` nor `thomas-neemann` is
  credited with authoring a project (`same_author`/`contributes_to`) of
  their own — they're forum posters whose idea only ever got *built* by a
  third party. This is a **candidate shape neither "pair" nor "revival
  target" covers**: it's "credit and consult the idea's originators when
  approaching the project that actually built it." Recommend Stage B treat
  this as its own test case for whatever shape gets added to the model
  (see note below), with `project:freepdm` as the outreach target and
  `person:openfablab`/`person:thomas-neemann` folded into its contact list
  as idea-originators, not contributors.
- **OdooPLM/FEP-0011 mutual-unawareness**: named explicitly in
  `ecosystem_relationship_findings.md` as a documented non-edge — real per
  the prose (`freecad_pdm_plm_ecosystem_census.md` §10.8 point 4), not
  expressible as `aware_of_uncoordinated` (which requires evidenced
  awareness, not mutual absence of citation). Shares `category: pdm` but
  not `technical_approach` (`other` vs. `sql-database`), so it also misses
  the §3/§4 facet-match rule. Worth piloting specifically *because* it's
  the weakest-evidence case in this whole set — a good test of whether the
  scoring model produces a sensible low-but-nonzero result rather than
  either forcing a false edge or refusing to score it.
- **The `rejected_in_favor_of` cases** are real but are single-project
  flags, not pairs (see §2): FreePDM's three rejections are bespoke
  (`to: null` + `chose_instead`), and openfablab's targets
  `project:openplm` from a person with no project of his own. Stage B's
  Friction-filter pilot should use these against whichever candidate
  touches `project:freepdm` or `project:openplm` (e.g. `project:openplm`
  as a revival target, §5), not expect a standalone rejection *pair*.

**Model-shape note for Stage B**: the sidecar-identity case suggests the
alliance-analysis plan's two candidate shapes (pair, revival target) may
need a third — a multi-party "idea lineage" or "convergence acknowledgment"
shape, where the outreach target is a single active project but its
contact list should include people who never touched that project's code
but demonstrably originated its idea. Recommend deciding this as part of
Stage B's own rubric review (the same review that's already checking
whether the four-factor score discriminates well), not fixing it here in
Stage A.

## Totals

- §1 edge-triggered pairs: **15**
- §2 flags (context, not candidates): **11**
- §3 strong facet-matched pairs: **35**
- §4 weak facet-matched pairs: **79**
- §5 revival targets: **17**
- **146 total candidate items** (129 pairs + 17 standalone revival targets)
  entering Stage B.
