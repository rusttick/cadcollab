# Ecosystem Alliance Analysis — Methodology and Plan

Purpose: Stages 1-5 of `ecosystem_relationship_mapping_plan.md` answered "how
does this ecosystem relate to itself?" and produced a structured graph
(`ecosystem_graph.yaml`, 95 nodes / 61 edges) plus two derived documents
(`ecosystem_visualizations.md`, `ecosystem_relationship_findings.md`). This
next phase asks a different, action-oriented question: **given what we now
know about how these projects relate, which specific collaborations are
worth proposing, and in what order?**

This document is the plan for doing that analysis. It is deliberately not
the analysis itself — per the same reasoning that motivated a separate
pilot stage in the mapping plan, the scoring method needs to be designed and
sanity-checked against a few cases we already understand well before it's
run at full scale and turned into actual outreach. The proposals themselves
belong in a later document (`ecosystem_alliance_proposals.md` or similar,
named at Stage 4 below), out of scope here.

**Sequencing note**: the immediate priority is understanding *how much
overlap in content and stated goals* exists across the ecosystem — which
projects are duplicating the most work, or converging on the same ideas —
independent of whether or how anyone would be contacted about it. Who to
contact and how is a downstream question that only gets asked about
whichever candidates survive the overlap analysis; it is not investigated
speculatively for the whole candidate set. The "Identifying who to
contact" and "Outreach architecture" sections below are designed and kept
in this plan, but are explicitly **not entered until Stage E** produces a
tiered list of survivors — see the Staged plan.

## Why the existing graph is the right input, and what it's missing

The relationship graph already encodes most of what "shared goals" means,
just not phrased that way:

- **Two projects with the same `category` and `technical_approach`** are
  candidates for a straightforward merge-of-effort conversation — they are
  already doing the same thing the same way.
- **An `independently_reinvents` edge** *is* a shared-goal signal, discovered
  rather than declared: two parties arrived at the same idea without
  knowing about each other. This is arguably the single best-qualified
  input this analysis has, because it's not a guess about alignment — it's
  observed convergent behavior.
- **A `shares_standard` edge** is a shared-goal signal at the interoperability
  layer specifically — two projects that already agreed (independently or
  not) to speak the same external vocabulary have a lower-cost path to
  talking to each other than two projects that would need to invent a
  shared vocabulary from scratch.
- **An `aware_of_uncoordinated` edge** is close to the cheapest possible
  alliance opportunity: the parties already know of each other; nobody has
  to be introduced, only reconnected.
- **A `rejected_in_favor_of` edge is a shared-goal signal with a warning
  label.** Two projects considered the same problem and one deliberately
  chose not to build on the other. The `reason` field (and, since the v1
  freeze, `chose_instead`) usually explains why — and that reason is
  exactly the information needed to tell a *revival* proposal from a
  *waste-of-time* proposal (see Factor 5 below).
- **`same_author` and `contributes_to` edges** identify the people, not just
  the projects, worth including in any outreach — and identify bus-factor
  risk, which matters for a different reason here than it did for the
  mapping phase: a single-maintainer project is both the most fragile
  *and* the cheapest to reach (one conversation, not a community process).
- **`mentors`/`mentored_by` edges are direct, observed evidence of someone's
  ability to bring other people into a project**, not just an authorship
  credit. This turns out to be the graph's best existing proxy for "who can
  mobilize others," addressed properly below.

What the graph does **not** currently encode, and that this analysis will
need to either add as new facets or explicitly treat as unknown:

- **Maintainer reachability / current activity.** `status` (`active`,
  `stalled`, `dead`, ...) is "as of last research pass," not "as of today,"
  and says nothing about whether the person behind an active-looking repo
  is still responsive. This matters more here than it did for the mapping
  phase, where staleness was a caveat; here it's load-bearing, because a
  proposal to a maintainer who left the community two years ago just wastes
  everyone's time. **Recommend adding**: a `last_verified_active` date
  facet per project, populated by a lightweight check (last commit date,
  last forum/GitHub post by the linked person node) rather than new deep
  research — this is a verification pass, not a research pass.
- **Recency and mobilization signal, per person, not just per project.**
  `status` tells you if a *project* looks alive; it says nothing about
  which specific person to write to, or whether that person can rally
  others rather than just merge a patch alone. This is developed as its own
  methodology below ("Identifying who to contact"), since it turned out to
  need more than a single new facet.
- **License compatibility as a structured fact, not prose.** The `license`
  facet exists per-project but nothing currently checks pairwise
  compatibility. A collaboration between a GPL project and a
  commercially-licensed one (e.g. Anchorpoint) needs that flagged before
  the proposal stage, not discovered during it.
- **Prior outreach history.** None of the existing edges capture "someone
  from project A already contacted project B about this" — because Stage
  1-3 predates any outreach. This analysis will generate candidates; it
  will not yet have any record of attempts. That's fine for a first pass,
  but the resulting proposals doc should have a place to record outcomes so
  a future re-run of this analysis doesn't re-propose something already
  tried and declined.

## Similarity / overlap scoring (the immediate priority)

Before any feasibility or contact question, every one of Stage A's 146
candidates gets a single, standalone **overlap rating** — how much genuine
technical/goal overlap exists, full stop, independent of connection cost,
history, or who might act on it. This is deliberately narrower than the
four-factor model below: it's one factor (Goal alignment), promoted to run
first and alone across the whole candidate set, because it's the one
factor Stage A's own bucketing already gives a mechanical, non-judgment
answer to. Four tiers, derived directly from what's already in the graph:

| Overlap tier | Criterion | What it means |
|---|---|---|
| **S — Observed convergence** | An `independently_reinvents` or `shares_standard` edge exists between the two | Not a guess — the graph already records that these two arrived at (or agreed to) the same thing |
| **A — Documented awareness** | A `cites_as_prior_art` or `aware_of_uncoordinated` edge exists, without an S-tier edge | They know of each other and the relationship is on record, but it isn't (yet) established as the *same* idea, only related ones |
| **B — Specific facet match** | No edge, but matching `category` **and** a shared `technical_approach` value more specific than `other`/`unknown` | Same kind of thing, same real mechanism — a plausible duplication nobody's flagged |
| **C — Thematic match only** | No edge, matching `category`, but `technical_approach` is `other` on both sides (or otherwise uninformative) | Weakest signal — same theme, no confirmation the actual approach overlaps; `other` is a catch-all, not a real match |

This reuses, rather than duplicates, Stage A's own candidate-generation
buckets almost exactly: §1 of `ecosystem_alliance_candidates.md` is Tier S
by construction, §3 is Tier B, §4 is Tier C, and the `cites_as_prior_art`/
`aware_of_uncoordinated` rows already inside §1 and §2 sort into Tier A.
Producing the overlap ranking is therefore close to a relabeling-and-
sorting pass over data already gathered, not new analysis — which is why
it can run across the full 146-candidate set immediately, rather than
needing its own pilot-then-full-pass staging the way the four-factor model
did.

**What this ranking is for**: identifying, before anything else, which
pairs represent the most duplicated effort (Tier S, where two projects
provably built the same thing) or the most defensible "these should talk"
cases (Tier A/B) — versus the long tail of thematic-only matches (Tier C)
that are unlikely to be worth anyone's time without much stronger
evidence. Only Tier S/A/B candidates (and Tier C candidates with an
unusually compelling individual case) proceed to the four-factor scoring
below; Tier C as a whole is a background fact about the ecosystem's
overall shape (a lot of "same theme, no evidence of matching approach"),
not a queue of things to act on.

## Identifying who to contact

*(Deferred — see the sequencing note above. This section defines the
methodology for later use; it is not run until Stage E has produced a
tiered list of survivors from the overlap and four-factor scoring below.)*

Original authorship (`same_author`) is a necessary starting point but an
insufficient answer to "who do we talk to." A project's original author may
be long gone; its most consequential person today may be a later
contributor, and the person best positioned to *mobilize others* — get a
team moving rather than just accept a patch — is a distinct trait from
either "created it" or "currently commits to it." All three matter and
none subsumes the others, so this analysis produces a **ranked short list
of contact candidates per target, not a single pick**.

For each project or revival target, assemble contact candidates from:

- **Creator** — the `same_author` edge into the project. Highest default
  credibility (it's their idea), but not necessarily still present or
  still the right first call. **`same_author`'s `from` can now be an
  `organization` as well as a `person`** — some projects are maintained by
  a company, community association, or open-source development firm
  rather than an identifiable individual (e.g. `project:openplm` →
  `organization:linobject`, `project:odooplm` →
  `organization:omniasolutions`/`organization:odooplm-community-
  association`). Where the graph only has the organization and not its
  named lead developers, the contact candidate is the organization itself
  (a company contact channel, a community forum) rather than an
  unresolved "unknown person" gap — the two aren't the same kind of
  missing.
- **Sustained contributor** — `contributes_to` edges into the project.
  These people showed up repeatedly over time for a project that wasn't
  theirs, which is itself a costly signal of genuine interest — often a
  better bet for "will actually pick this back up" than the original
  author, who may have already moved on once (that's usually *why* the
  project stalled).
- **Most recent contributor** — not currently in the graph as an edge type,
  because it's a mechanical git-log/commit-history fact, not something the
  census prose narrates. **Recommend a lightweight, non-interpretive check**
  (last N commits' authors, last few forum/GitHub posts on the project's
  `discussion` node) rather than new research — this is the same kind of
  verification pass as `last_verified_active`, and can likely be gathered
  in the same pass. Where this check hasn't been run yet, treat "most
  recent contributor" as `unknown` rather than assuming it's the same
  as the creator.
- **Demonstrated mobilizer** — a person with an outgoing `mentors` edge
  (direct, observed evidence they've successfully brought someone else into
  a project before — e.g. `person:yorikvanhavre`/`person:hardeeprai` →
  `person:ppodest` for the BCF plugin) is qualitatively different from
  someone who has only ever worked solo. Where no `mentors` edge exists for
  anyone touching a project, this signal is simply absent — don't force a
  guess; note it as `unknown` and fall back to the other three candidate
  sources.

**Capacity is a separate axis from credibility, and matters as much.** A
person with high `same_author`/`contributes_to` fan-out (pieterhijma,
alekssadowski95, the Open Source Ecology cluster) is credible precisely
*because* they're already central — but that same fan-out means they may be
the least available person to take on one more thing. Flag high-fan-out
contact candidates explicitly as "high credibility, possibly low capacity"
rather than always ranking them first by default; for a Tier 1 quick
introduction this barely matters (a short note costs little of anyone's
time), but for a Tier 3 revival ask that implies sustained follow-through,
a less central but more available person may be the better first contact.

The output of this step, produced alongside the four-factor score below, is
a short ordered list (1-3 names) per candidate/target with a one-line
rationale each — e.g. "contact `person:grd` first (creator, still posting
as of last check); `person:dan-miel` as a fallback (`contributes_to`,
long-standing design engagement, no mobilizer signal but has stayed
involved longer than the average contributor)."

The `mentors` edge is sparse across the graph — it's only recorded for one
case (the BCF plugin's GSoC pairing) — so for most targets the mobilizer
slot will come back `unknown` and the ranking will lean on the
sustained-contributor and recency signals instead. Worth watching during
Stage C whether that fallback is good enough, or whether it's worth a
targeted re-read of the census discussion threads specifically for
mobilizer evidence (visible enthusiasm, replies drawing others in) that
wasn't extracted the first time because Stages 1-3 weren't looking for it.

## The scoring model

*(Applied only to candidates that clear Tier S/A/B in the overlap scoring
above — see the sequencing note. This is the feasibility layer, entered
after the content-overlap question is already answered, not instead of
it.)*

Each candidate is a **pair** (project-project) or a **revival target**
(single dead/stalled project). Four factors, each scored independently so
the reasoning stays inspectable rather than collapsing into one opaque
number:

| Factor | What it measures | Signals from the graph |
|---|---|---|
| **Goal alignment** | How much genuine shared purpose exists | Shared `category` values; shared `technical_approach`; an `independently_reinvents` edge (strongest signal — it's observed, not inferred from facets); a `shares_standard` edge |
| **Connection cost** | How much work it takes to *start* the conversation | Existing edge between the two nodes (`aware_of_uncoordinated` = lowest cost; `cites_as_prior_art` = they already read each other's work; no edge at all = cold outreach, highest cost); whether a `same_author`/`contributes_to` person bridges them already |
| **Friction / history** | Reasons this might not work | A `rejected_in_favor_of` edge between the two, and specifically whether its `reason` is technical (e.g. "git-like internals don't suit FCStd's format" — likely still true, dampens the case for that *specific* technical merge, though not necessarily for a looser alliance) or circumstantial (e.g. "too heavy for a hobbyist project," which may no longer apply if resourcing changed); license mismatch; `origin_language`/`origin_country` gap where no bilingual bridge person exists in the graph |
| **Payoff** | What's gained if it works | Bus-factor reduction for a single-maintainer project; consolidation of an `independently_reinvents` cluster (fewer parallel maintenance burdens ecosystem-wide); progress toward a `shares_standard` project actually interoperating rather than just nominally targeting the same standard; for a revival target, whether the abandoned project's approach is one the graph shows *nobody else* has reinvented (i.e., reviving it adds something not otherwise available, rather than duplicating an already-thriving alternative) *and* whether the "Identifying who to contact" step above actually found a viable contact candidate — a revival with high idea-value but no findable, plausibly-mobilizing contact caps out at medium Payoff until that changes |

Each factor gets a simple **high/medium/low** rating with a one-line
rationale citing the specific edge/node ids that drove it — not a numeric
score. A numeric score invites false precision on data this qualitative;
a four-factor high/medium/low profile is enough to rank and, critically,
enough that a reader can see *why* two proposals with the same rank differ.

**Ranking rule**: rank primarily by (Goal alignment + Payoff) high, then
break ties by Connection cost low, then treat Friction as a filter rather
than a ranking input — a high-friction pair is demoted to a separate
"needs a different framing before outreach" list rather than simply scored
lower, because the friction reason itself is a data point (see Stage 3, the
per-item review) not just a penalty.

## Two candidate types need different treatment

**Alliance candidates** (both projects `status: active` or the equivalent):
prioritize pairs with an `independently_reinvents` or `shares_standard`
edge and low connection cost — these are the "you two are already doing
the same thing, here's the introduction" cases, which is the easiest kind
of proposal to write credibly.

**Revival candidates** (project `status: stalled`/`dead`, no active
successor doing the same thing): the question isn't "who should this ally
with" first, it's "is reviving this even worth proposing" — checked via the
Payoff factor's "does the graph show this idea is otherwise unavailable"
test. Only once that's established does the revival candidate get treated
like an alliance candidate, matched against whichever *active* project has
the closest goal alignment (frequently the project that independently
reinvented the same idea later — the graph already names these pairs).
A revival proposal is really "connect an abandoned idea to a currently-active
team solving the same problem, via whichever contact candidate from the
ranked list above is both credible and available," not "restart the old
repo in isolation" and not automatically "contact the original author" —
this framing should carry through to the proposals doc.

## Outreach architecture

*(Deferred — see the sequencing note above. Designed now, entered only
after Stage E produces a tiered list and, per "Identifying who to
contact," a specific contact is identified for each surviving candidate.)*

You will conduct all outreach personally, through whichever platform and
identity fits the specific contact — never impersonating a project,
maintainer, or third party, but not necessarily under a single fixed
identity either. This section architects *how that choice gets made and
sequenced* per candidate; it does not draft message copy (that's Stage F,
per-candidate, when there's a concrete person and proposal in front of it).

**Identity model.** Three modes, chosen per candidate, not fixed globally:

1. **Personal/participant identity** — you, disclosed as an interested
   party genuinely engaging with the project. Fits `aware_of_uncoordinated`
   reconnections and any case where the ask benefits from visible personal
   investment (a revival ask in particular reads better as "someone
   actually cares about this" than as an anonymous nudge).
2. **Disclosed "ecosystem connector" identity** — presented honestly as
   someone who has been mapping how these projects relate (which is
   literally true) rather than as a stakeholder in either project. Fits
   matchmaking outreach best: "I've been mapping the FreeCAD PDM/PLM space
   and noticed you and X arrived at the same approach independently" is
   credible precisely because it's disclosed and true, and it's neutral in
   a way that reduces any appearance that one party is recruiting the
   other on their own behalf.
3. **Anonymous/pseudonymous first probe** — a low-commitment, undisclosed-
   identity opening touch, reserved for Tier 4 ("needs reframing")
   candidates where the friction history makes a named, attributable
   approach premature. Used only to test interest before deciding whether
   a fuller, identified outreach is worth the effort — never as the
   permanent mode of engagement if the conversation continues.

In all three modes: always your own account/persona, always honest about
what it is (a connector, a person, or an intentionally minimal first
touch) — the distinction is disclosure and framing, never deception about
who's speaking.

**Channel selection.** Use the graph's own data on where each contact
candidate already is, rather than a uniform default:
- If the person has a live `discussion` node they're already active in
  (an open forum thread, a GitHub issue/PR they've posted to recently),
  reply there — it's lower-friction than a cold new thread and keeps prior
  context attached.
- Otherwise, use whichever `handles` field the graph records (forum,
  GitHub) for a direct message or a new issue, matching the register of
  that platform.
- Only fall back to email or another channel not present in the graph if
  neither of the above yields a live, checkable presence — at which point
  the "Identifying who to contact" step's reachability check should
  probably have flagged this candidate as lower-confidence to begin with.

**Message ladder, tied to the Stage E tiers below** (content drafted per
candidate at Stage F, not here — this fixes the *shape* of each rung):
- **Tier 1 (introduction)**: short, factual, low-ask — name the specific
  convergent idea, invite direct contact between the two parties, nothing
  more.
- **Tier 2 (active consolidation)**: similar opening, but proposes a
  concrete next step (a joint thread, a shared roadmap item) since both
  sides are active and can absorb a more direct ask.
- **Tier 3 (revival via reframing)**: addressed to the best available
  contact candidate, framed as "this idea independently resurfaced
  elsewhere, and it's active today" — an invitation to reconnect, not a
  request to single-handedly resume unfinished work.
- **Tier 4 (needs reframing)**: an exploratory, low-commitment probe,
  possibly under the anonymous/pseudonymous mode above, testing interest
  before a fuller identified approach is attempted.

**Team-formation ladder, after initial contact succeeds** (this is the
"get work moving" mechanism, distinct from the first message): (1) direct
introduction / mutual awareness established; (2) a shared low-commitment
venue — an existing or new forum thread / GitHub Discussion, not yet a
repo or roadmap commitment; (3) a concrete joint artifact — a shared doc,
a cross-linked issue, a joint FEP-style proposal — the first thing that
outlives the conversation itself; (4) sustained collaboration, which
becomes the projects' own trajectory and is deliberately not architected
further here. Each rung is a separate go/no-go point — nothing here assumes
a contact who responds to (1) will want (3); the ladder exists so a stall
at any rung is a legible, expected outcome rather than a failure to force
past.

**Outcome logging.** The graph currently has no record of outreach, because
none has happened yet. Recommend a lightweight companion log —
`doc/ecosystem_outreach_log.md`, one entry per attempt: candidate id, date,
channel, identity mode used, message summary, response, current
team-formation-ladder rung. This is what lets a later re-run of this
analysis avoid re-proposing something already tried, and lets a "declined"
or "no response" outcome demote a candidate the next time Stage D is
re-run. Building this log is part of Stage G below, not needed to start
Stages A-D.

## Staged plan

**Stage A — Candidate generation (mechanical, no judgment yet).** Walk
`ecosystem_graph.yaml` and mechanically enumerate:

- Every project pair connected by at least one qualifying signal edge:
  `independently_reinvents`, `shares_standard`, `aware_of_uncoordinated`,
  `cites_as_prior_art`, or `rejected_in_favor_of` (included specifically so
  Stage C has real candidates to pilot the Friction filter against).
- Every project pair with **no edge at all** but matching `category` *and*
  matching `technical_approach` values, **excluding any pair where either
  side's `category` or `technical_approach` is `unknown`** — with roughly
  two-thirds of the census sitting at `unknown`/`other` on these facets,
  matching on "both unknown" isn't a real alignment signal and would flood
  the list with noise rather than candidates.
- Every project with `status: stalled` or `status: dead` as a standalone
  revival target, full stop — no filtering at this stage on whether a
  viable successor or contact exists; that judgment belongs to the later
  feasibility scoring's Payoff check (Stage D) and "Identifying who to
  contact," not to mechanical generation.

This is closer to a query over the YAML than research; it produces a long,
unranked candidate list, not proposals.

**Stage B — Overlap scoring, full pass (all 146 candidates).** Apply the
S/A/B/C overlap-tier rule from "Similarity / overlap scoring" above to
every candidate from Stage A. As noted there, this is mostly a relabeling
of Stage A's own buckets, not new analysis, so it runs as a single full
pass rather than needing its own pilot-then-full staging. Output: a
ranked/tiered overlap list — the ecosystem's actual map of duplicated
effort and converging goals, and a complete, standalone answer to "which
projects overlap most," independent of anything about feasibility or
contact. This is the priority deliverable; Stages C onward are about
deciding what, if anything, to *do* with it.

**Stage C — Pilot the four-factor feasibility model (5-8 candidates).**
*(Note: this stage's pilot work was already completed, under the old
Stage-B label, before this plan was resequenced — see
`ecosystem_alliance_stage_b_pilot.md`. Its findings carry over unchanged;
only the stage letter and its position after overlap scoring have
changed.)* Before scoring everything,
apply the four-factor model from above to a small, well-understood set,
drawn from `ecosystem_alliance_candidates.md` (Stage A's output) —
recommend: the openfablab/thomas-neemann/FreePDM sidecar-identity chain
(§6 of that doc — an idea-lineage case that doesn't fit the pair/revival-
target model cleanly, which is exactly why it's worth piloting first), the
Riegel-issue-cluster/Ondsel-Server#48 pair (§1 — a revival-adjacent case,
since the original idea's champion is not necessarily still reachable but
the idea itself is unclaimed), the OdooPLM/FEP-0011 mutual-unawareness case
(§6 — tests whether the weakest-evidence case in the whole set produces a
sensible low-but-nonzero score rather than either a false edge or a
refusal to score), and `project:openplm` as a revival target scored against
its two recorded rejections (§2/§5 — tests whether the Friction filter
correctly separates "still applies" from "circumstances changed" using
real rejection reasons rather than a hypothetical pair). Check: does the
high/medium/low rubric actually discriminate between these cases, or does
everything cluster at "medium"? Does the Friction filter produce a
sensible "needs reframing" list rather than just suppressing valid
candidates? And, per the model-shape note in that doc's §6: does the
sidecar-identity case need a third candidate shape (an "idea lineage /
convergence acknowledgment" type, distinct from pair and revival target),
or does it fit well enough into "revival target" once `project:freepdm`'s
contact list is understood to include people who never touched its code?
Adjust the rubric — and the candidate-shape model, if needed — once,
deliberately, the same way the relationship taxonomy was adjusted once
after its own pilot.

**Stage D — Full feasibility scoring pass.** Apply the adjusted four-factor
rubric — **only to candidates that cleared Tier S/A/B in Stage B's overlap
scoring**, not the full 146. Tier C (thematic-only overlap) stays a
background fact about the ecosystem's shape, not a queue to score
individually, unless a specific Tier C candidate has an unusually
compelling case worth promoting by hand. This is mechanical application of
an already-validated rubric, not new design work — budget it like Stage 3
of the mapping plan (a structuring pass over already-gathered material, no
new web research required, since every input is either already in the
graph or is the lightweight `last_verified_active` check flagged above).

**Stage E — Ranking and tiering.** Sort the fully-scored list per the
ranking rule above. Group into tiers that map to different kinds of
next action, e.g.:
- Tier 1: "just make the introduction" (high alignment, low connection
  cost, low friction) — this covers both a straightforward two-project
  introduction *and* an idea-lineage acknowledgment (connecting an idea's
  originators to whoever eventually built it, even if that person has no
  project-authorship edge of their own — the Stage B pilot's Case 1
  confirmed this reading is broad enough without needing a separate
  candidate shape)
- Tier 2: "propose active consolidation" (high alignment, existing
  independently_reinvents edge, both sides currently active)
- Tier 3: "revival via reframing" (high payoff, needs the original idea
  reconnected to a currently-active effort)
- Tier 4: "needs a different framing before outreach" (the friction-filtered
  set — not rejected, just not ready as a plain introduction; the idea
  itself still has value, only the history or evidence is thin)
- Tier 0: "not recommended" (Payoff independently low — the idea is
  already well-served by an active alternative elsewhere in the graph —
  regardless of Friction or Connection cost; distinct from Tier 4, whose
  candidates still have live value. A revival target reaches Tier 0 only
  after failing its own Payoff gate, per the Stage B pilot's Case 4)
This tiering, not a single ranked list, is the actual deliverable structure
for Stage F, because "most likely to succeed" is meaningfully different
from "highest total value" and the two shouldn't be silently averaged
together.

Because you are a single person doing all outreach yourself, tiering also
functions as a personal queue: Tier 1 items are cheap enough to exhaust
before investing in Tier 3, both because they're low-effort and because an
early win or two is useful before attempting a harder revival ask.

**Stage F — Write the proposals document.** Only after Stages A-E, and
only for the tiered survivors: produce `ecosystem_alliance_proposals.md`,
one entry per candidate, in tier order, each citing the graph ids and
factor ratings that justify it. This is also the first point at which
"Identifying who to contact" and "Outreach architecture" actually get
applied — per-candidate, for the final surviving list only, not run
speculatively over the full 146 earlier. This is the document you'd
actually work from — it does not exist yet and is explicitly out of scope
for this plan document.

**Stage G — Outreach execution and logging (yours, not delegated).** You
send the messages, create whatever accounts/identities each candidate
calls for, and run the actual conversations. This plan's job ends at
handing you a worked queue; it doesn't extend into conducting the
conversations. What it does own: keeping `ecosystem_outreach_log.md`
current enough that a future re-run of Stages A-D can see what's already
been tried, and — periodically, not after every message — folding outcomes
back into the graph (e.g., a positive Tier 3 response might justify adding
a real `contributes_to` or even reviving a project's `status`, which is a
legitimate, welcome way for this graph to stop being purely historical).

## Open question going into Stage A

**Whether a `rejected_in_favor_of` edge should ever be a hard exclusion
rather than a Friction-tier demotion.** Recommend no blanket rule: decide
per-candidate at Stage C whether the rejection reason still applies to the
*specific* technical merge in question (e.g. FreePDM/SnowFS's architectural
incompatibility likely still holds) without excluding a looser alliance
between the same two projects on a different axis (shared documentation,
shared standards adoption, etc.).
