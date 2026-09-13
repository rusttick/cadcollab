# Ecosystem Alliance Proposals (Stage F)

The Stage F deliverable per `ecosystem_alliance_analysis.md`: one entry per
candidate that survived tiering (Stage E), in tier order, each with a
contact recommendation (per "Identifying who to contact") and an outreach
recommendation (per "Outreach architecture") — the first point in this
whole process where either of those methodologies is actually applied,
per the plan's sequencing.

**A pattern worth flagging up front**: half of the 14 surviving candidates
have *no* person or organization node in the graph for one or both
projects — not because Stage 1-3 called them `unknown`, but because their
maintainer's handle sits unextracted inside the project's own `name` field
(e.g. `"HistoryWorkbench (eblanshey/HistoryWorkbench)"`). Where that's the
case, this doc uses the handle directly from the name field — a zero-cost
read of data already gathered, not new research — and flags it as
`(from project name, not yet a person node)` rather than presenting it as
equivalent-confidence to a properly extracted `same_author` edge. No new
research was performed to fill any gap beyond this.

---

## Tier 2 — Propose active consolidation

### `project:issue-25681-cluster` ↔ `project:ondsel-server`
**Scores**: Goal alignment High · Connection cost Very low · Friction Low · Payoff High
**Contact**: `person:pieterhijma` (creator, `same_author` — flag: high
fan-out across 5 projects, credible but possibly stretched) for the
cluster side; `person:pierreporte` (`contributes_to`) for Ondsel-Server —
**and pierreporte is the better first contact of the two**, since he
already posts in both threads.
**Outreach**: Disclosed "ecosystem connector" identity — the honest framing
here is "you two are already informally overlapping through pierreporte;
worth making that explicit." Channel: reply in whichever of the two
threads (`discussion:fep-0011-discussion-40` or
`discussion:ondsel-server-48`) is more recently active. Message ladder:
Tier 2 rung (propose a concrete next step, not just an introduction — e.g.
suggest the two efforts jointly document their shared vocabulary).
Team-formation first step: a shared thread/Discussion explicitly cross-
linking both efforts' existing docs.

### `project:historyworkbench` ↔ `project:pr-28312`
**Scores**: Goal alignment High · Connection cost Low-Medium · Friction Low · Payoff High
**Contact**: `person:pieterhijma` (`same_author`, `pr-28312`) — same
capacity flag as above. HistoryWorkbench's maintainer, `eblanshey`, has no
person node — handle taken directly from the project name field, not
independently verified.
**Outreach**: Disclosed connector identity — "your addon is the most-
starred project in this whole survey, and it's solving almost exactly what
PR #28312 is trying to do from the other direction; have you two talked?"
This is a genuinely strong, evidence-backed opener. Channel: GitHub, on
whichever of the two repos/PRs is more active. Team-formation first step:
suggest `eblanshey` weigh in directly on PR #28312's discussion — the
lowest-friction way to start real technical cross-pollination.

### `project:ondsel-server` ↔ `project:ose-vcs-library`
**Scores**: Goal alignment High · Connection cost Medium · Friction Low · Payoff High
**Contact**: `person:pierreporte`/`person:creymore` (`contributes_to`,
Ondsel-Server side) — no `same_author` edge exists for Ondsel-Server at
all (it reads as institutional). `person:ose-org` (`same_author`,
Open Source Ecology side) — also institutional; OSE is described in the
graph as coordinating three repos pushed the same day, suggesting an
organized team rather than a single point of contact.
**Outreach**: Disclosed connector identity, since this is a genuinely cold
cross-institutional introduction with real evidence behind it (near-
identical vocabulary, independently arrived at). Channel: OSE's public
project page/forum, since no shared discussion thread exists yet. Team-
formation first step: a joint comparison document mapping the two
vocabularies term-for-term — concrete, low-commitment, and plays to what
both sides already have (structured schemas).

---

## Tier 3 — Revival via reframing

### `project:bcf-plugin-freecad` ↔ `project:fep-0011`
**Scores**: Goal alignment High · Connection cost Low · Friction Low · Payoff Medium-High
**Contact**: `person:podestplatz` (`same_author`; GSoC 2019 student,
project dead since 2024 — reachability not verified, flag before assuming
he's still findable at the same handle) as the idea's origin; separately,
`person:pieterhijma` on the FEP-0011 side, since FEP-0011 is the active
party actually driving this.
**Outreach**: Personal/participant identity fits better than the connector
framing here — this is a genuine revival ask ("your work has a real home
now"), which reads better as someone who actually cares than as a neutral
observer. Channel: `discussion:bcf-gsoc-proposal-thread` if still open,
otherwise a fresh GitHub note referencing it directly. Team-formation
first step: point `podestplatz` at FEP-0011's own stated intent to rebuild
BCF workflows on the new framework — the ask is "come comment on how your
design should map," not "resume unfinished work."

---

## Tier 1 — Just make the introduction / close an open loop

### `project:freepdm` ↔ `project:ondsel-server`
**Contact**: `person:grd` (`same_author`, FreePDM). **Outreach**: Personal
identity — this is literally answering a question grd's own community
already asked and never resolved (`aware_of_uncoordinated`), so a genuine
"someone looked into this and here's what I found" tone fits better than a
neutral connector framing. Channel: `discussion:about-pdm-for-freecad`
directly.

### `project:anchorpoint` ↔ `project:easypdm`
**Contact**: Anchorpoint is a commercial product with no individual person
node (likely a company support/community channel, not an individual).
EasyPDM's author handle is `pawelcel` (from the project name field, not a
person node) — notable given the project's own README states it was
"written for me by Claude... by an author self-described as a mechanical
design engineer, not a programmer," which may affect how technical the
outreach framing should be. **Outreach**: Disclosed connector identity,
low-key — "you and a commercial product both landed on the same file-
locking approach independently." Channel: EasyPDM's GitHub, Anchorpoint's
public community/support channel.

### `project:cadbaselibrary` ↔ `project:fep-0011`
**Contact**: CADBaseLibrary's handle is `mnnxp` (from the project name;
real development is on GitLab per the graph's own note, not GitHub — worth
checking there rather than the GitHub mirror). `person:pieterhijma` on the
FEP-0011 side. **Outreach**: Disclosed connector — "FEP-0011 already
cites your work as prior art; worth a direct connection." Note the
origin_language is Russian; consider whether a bilingual framing helps.
Channel: GitLab (gitlab.com/cadbase), not the GitHub mirror.

### `project:fep-0011` ↔ `project:freepdm`
Same as `freepdm ↔ ondsel-server`'s `person:grd` contact, different
purpose — closing FEP-0011's own citation loop rather than the Ondsel
question. Can likely be handled in the same conversation as the item
above rather than as a separate outreach.

### `project:fep-0011` ↔ `project:nanoplm`
**Contact**: `person:alekssadowski95` (`same_author` — flag: high fan-out
across 5 projects, same capacity caveat as pieterhijma). **Outreach**:
Disclosed connector, low-stakes — mostly a courtesy citation-closing note,
per the Payoff rating.

### `project:fep-0011` ↔ `project:ondsel-lens-addon`
No separate contact — recommend folding this into the
`issue-25681-cluster ↔ ondsel-server` Tier 2 conversation rather than a
standalone outreach, since it's the addon-side sibling of that same
effort and no independent contact is recorded for it.

### `project:fep-0011` ↔ `project:taack-plm`
**Contact**: handle `Taack` (from the project name field). **Outreach**:
Disclosed connector, low-stakes courtesy note. (See Stage D's note: the
mapping plan's own worked example about grd calling Taack "alien" was
checked against the source text and doesn't hold up — no friction history
actually applies here.)

---

## Tier 4 — Needs a different framing before outreach

### `project:anchorpoint` ↔ `project:freecad-omniverse-connector`
**Contact**: Anchorpoint (commercial, no individual node);
`person:soemantoro` (`same_author`, Omniverse connector). **Why Tier 4,
not 2**: real convergence, but a commercial product and a peer-reviewed,
publicly-funded fusion-research infrastructure project are institutionally
about as far apart as two "yes" answers can get — worth thinking through
what a realistic ask even looks like before reaching out, rather than
defaulting to the same connector-intro framing used above.

### `project:issue-25681-cluster` ↔ `project:openpartslibrary`
**Contact**: `person:pieterhijma`; `person:alekssadowski95`. **Why Tier
4**: the underlying `independently_reinvents` edge is itself marked in the
graph as weaker/speculative. Recommend a closer read of both projects'
actual designs before proposing anything — the outreach itself should
probably be a question ("are these actually solving the same problem?"),
not a proposal.

### `project:freecad-cloud-browser-hitclawagent` ↔ `project:freecad-cloud-browser-sabi137032`
**Contact**: handles `hitclawagent` and `sabi137032` (from project names).
**Why Tier 4, and why anonymous-probe mode specifically**: same name,
overlapping features, created three days apart, "no visible relationship
between the authors" per the graph's own note. This could be coincidence,
an undisclosed fork, or something else — **recommend a low-key,
undisclosed-identity first probe to each author separately** (per the
Outreach architecture's Tier 4 mode) asking simply whether they're aware
of the other repo, before proposing any collaboration framing. Don't
assume good-faith parallel invention going in.

---

## Tier 0 — Not recommended (listed for traceability, not for action)

- `project:freecad-gitproject-reox` ↔ `project:versioncontrol-workbench-pfriedrich` —
  Payoff gate failed: both dead since 2018, and the idea is already well
  served by the Tier 2 `historyworkbench ↔ pr-28312` pair. No outreach
  recommended.

---

## Summary queue, in the order the plan's sequencing note suggests working it

1. **Tier 1** (7 items, low effort, several already-open citation loops —
   exhaust these first per the plan's "early wins before harder asks"
   logic)
2. **Tier 2** (3 items — the highest-payoff work, worth deliberate
   attention once warmed up)
3. **Tier 3** (1 item — the revival case, benefits from Tier 2's momentum
   existing as a credible "here's an active home for your idea")
4. **Tier 4** (3 items — hold until either more evidence surfaces or a
   specific reason to prioritize one arises)
