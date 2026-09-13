# Stage B — Pilot Scoring

Applies the four-factor rubric from `ecosystem_alliance_analysis.md` to the
four cases identified in `ecosystem_alliance_candidates.md` §6 (plus the
one clean §1 pair). This is a pilot, not the full pass — the point is to
check the rubric discriminates and the tiering rule behaves sensibly before
running it across all 146 Stage A candidates.

---

## Case 1 — The sidecar-identity idea lineage → `project:freepdm`

**Shape**: idea-lineage acknowledgment, not a pair or a revival target.
Outreach target: `project:freepdm` (active, `status: revived`). Idea
originators with no project of their own: `person:openfablab` (2020),
`person:thomas-neemann` (2021).

| Factor | Rating | Rationale |
|---|---|---|
| Goal alignment | **High** | Two `independently_reinvents` edges, one `confirmed` (`person:thomas-neemann` → `project:freepdm`, on the strength of `person:user1234`'s directly traceable cross-thread continuity), form a real three-hop chain converging on "stable identity outside the file." |
| Connection cost | **Low** | Both originators already posted to the same forum section FreePDM's own thread lives in (`discussion:forum-2020-openfablab-post`, `discussion:forum-2021-thomas-neemann-post`) — a reply citing specific old threads, not a cold introduction. |
| Friction | **Low** | No rejection edges anywhere in this chain — grd simply never saw these posts, not that he saw and dismissed them. |
| Payoff | **Medium** | FreePDM already shipped a working (if architecturally different — numbered-file-revision + filesystem permissions, not a JSON/text sidecar) answer to the same problem. Reconnecting the originators earns credit and may sharpen FreePDM's identity scheme, but isn't blocking anything. |

**Contact candidates for `project:freepdm`** (per "Identifying who to
contact"): `person:grd` (creator, `same_author`, still the clear first
call) as primary; `person:dan-miel`, `person:heda`, `person:zolko`,
`person:user1234` (all `contributes_to`, sustained multi-year engagement)
as a well-populated fallback bench — notably richer than most projects in
this census. `person:openfablab`/`person:thomas-neemann` belong on the
*recipient* side of this specific outreach, not the contact list for
FreePDM itself.

**Verdict**: proceeds as a low-cost, low-friction introduction — see the
tiering note below on where this actually belongs.

---

## Case 2 — `project:issue-25681-cluster` ↔ `project:ondsel-server`

**Shape**: a clean pair, both sides active.

| Factor | Rating | Rationale |
|---|---|---|
| Goal alignment | **High** | `cites_as_prior_art` (confirmed — #25681's problem statement names Lens as motivating context) plus an `independently_reinvents` lineage reaching back through `discussion:freecad-issue-5539` to the same item/identity/revision-control ideas; shared `category: pdm, real-time-collab`. |
| Connection cost | **Very low** | `person:pierreporte` already bridges both sides directly: `contributes_to` → `project:ondsel-server`, and per that same edge's note, "also weighed in on FEP-0011's discussion #40." This isn't a cold introduction — informal cross-pollination is already happening. |
| Friction | **Low** | No rejection edges between them; the only asymmetry is that #25681 cites Ondsel-Server and not (yet, on record) the reverse. |
| Payoff | **High** | These are the two largest, most active collaboration efforts in the entire FreeCAD-native PDM/PLM space, already independently converging on the same vocabulary. Formalizing coordination here has the highest ecosystem-wide leverage of any candidate found so far — it's the one place duplicate infrastructure at the *top* of the ecosystem is still avoidable. |

**Contact candidates**: for `issue-25681-cluster`, `person:pieterhijma`
(creator, `same_author`, flagged high-fan-out/possibly-low-capacity per
the analysis doc's capacity axis — he already carries five interlocking
efforts). For `ondsel-server`, no `same_author` edge exists at all in the
graph (Ondsel-Server reads as an institutional/company effort rather than
a single-founder one) — fall back to `person:pierreporte` and
`person:creymore` (`contributes_to`). **`person:pierreporte` is the
standout contact for this specific candidate**: he's already active in
both threads, so outreach here is closer to "make an existing informal
link explicit" than "introduce two strangers."

**Verdict**: this is the strongest candidate found in the pilot — high on
every factor that matters, and the connection cost is close to zero
because a bridge person already exists.

---

## Case 3 — `project:odooplm` ↔ `project:fep-0011`

**Shape**: a pair with no edge at all — the deliberately weakest-evidence
case in the pilot (see `ecosystem_relationship_findings.md`'s
asymmetric-awareness discussion).

| Factor | Rating | Rationale |
|---|---|---|
| Goal alignment | **Medium** | Shares `category: pdm` but not `technical_approach` (`sql-database`, server-backed, vs. FEP-0011's architecture-agnostic `other`), and differs in `scope` (`multi-cad-platform` vs. `freecad-native`) — real overlap, but weaker than Case 2's. |
| Connection cost | **High** | No edge, no shared person, no shared discussion node — genuinely cold outreach. OdooPLM's own node has no linked `person:` at all (see gap below). |
| Friction | **Low (absence of evidence, not evidence of absence)** | Nothing negative on record — but the census prose itself calls this "completely unaware of each other" as a *finding*, not a comfortable default; worth naming honestly as "no data" rather than a confident "low." |
| Payoff | **Medium** | OdooPLM is real, active, and well-adopted (152 stars, presented at Odoo Experience 2025) — more mature by usage than most FreeCAD-native attempts. A cross-citation would import real-world PLM experience into FEP-0011's design process, but the scope mismatch limits how directly OdooPLM's server-centric approach transfers to a framework meant to stay CAD-agnostic. |

**Contact candidates**: `fep-0011` → `person:pieterhijma` (creator). For
`odooplm` → **no contact candidate exists in the graph at all.** No
`same_author`, `contributes_to`, or `mentors` edge names a person behind
OdooPLM/OmniaGit — a genuine gap distinct from "unknown," since Stage 1-3
never captured this project's maintainer as a node. Flagged, not
guessed — filling this needs a lightweight check (the GitHub repo's own
contributor list), which is exactly the kind of mechanical verification
`ecosystem_alliance_analysis.md` already recommends for capacity/recency
data generally.

**Verdict**: the rubric doesn't collapse this to a flat "medium" — it
correctly produces a mixed profile (medium alignment and payoff, high
cost, no-data friction) distinct from both Case 2 and Case 4. This is the
candidate `ecosystem_alliance_analysis.md`'s outreach-architecture section
anticipated needing the **disclosed "ecosystem connector" identity** for:
a plain "I noticed you two are both solving PDM in FreeCAD-adjacent space
independently" framing fits a cold, evidence-thin case like this far
better than a personal/participant approach would.

---

## Case 4 — `project:openplm` as a revival target

**Shape**: revival target, tested against the plan's own "is reviving this
even worth proposing" gate before any pairing is attempted.

| Factor | Rating | Rationale |
|---|---|---|
| Payoff | **Low** | The idea space openPLM occupied — a full-featured, server-backed, database-driven PLM — is already actively populated by more modern, better-adopted successors already in this graph: `project:odooplm` (152 stars, active, presented in 2025), `project:docdoku-plm`, `project:ondsel-server`. Reviving openPLM itself adds nothing the graph shows is otherwise unavailable — the opposite of the FreePDM/Riegel-chain cases. |
| Friction | **High** | Two separate people independently treated openPLM as something to move *away* from: `person:wmayer` linked old openPLM threads early in the FreePDM discussion without adopting its approach, and `person:openfablab` explicitly rejected it (`rejected_in_favor_of`, confirmed) as "too heavy... fancy stuff." Both objections are about openPLM's fundamental weight/complexity, not a fixable resourcing gap — this is the kind of `reason` the plan's Friction factor is designed to catch as "still applies." |
| Connection cost | **High** | `status: dead` since 2024-01-30, no `same_author`/`contributes_to`/`mentors` edge for anyone still around. In fact **no person node in the graph is credited with authoring openPLM at all** — a second, more basic graph gap (distinct from Case 3's): the census never captured openPLM's own maintainer as a node, despite openPLM being cited/rejected by name three separate times by other people. Worth a targeted fix in a future extraction pass, not fabricated here. |
| Goal alignment | *(not scored — see verdict)* | Goal alignment is meaningless to score for a revival target whose Payoff has already failed the "is this worth proposing" gate; scoring it further would just be busywork. |

**Verdict**: **do not pursue.** This is exactly the case the plan's "Two
candidate types" section describes — Payoff fails its own gate before the
candidate ever reaches a pairing decision. Correctly filtered out by the
model, and correctly filtered out *before* Friction was even needed to do
the work — Friction here reinforces the same conclusion Payoff already
reached, rather than being what killed it. See the tiering fix below: this
reveals the tier model needs an outcome for "not recommended," distinct
from "needs reframing."

---

## Rubric-fit findings

**The four-factor profile discriminates well.** Across four cases: Case 1
is low-cost/medium-payoff, Case 2 is high on everything that matters with
near-zero cost, Case 3 is a genuine mixed medium/high-cost profile, and
Case 4 is low-payoff/high-friction/high-cost. Nothing clusters at a flat
"medium" — the rubric is doing real discriminating work, not just
producing noise dressed up as structure.

**The tiering model needs one addition: a "not recommended" outcome,
distinct from Tier 4 ("needs a different framing").** Case 4 exposed
this. The plan's current tiers assume every candidate that survives Stage
A is eventually worth *some* outreach, just with different framing or
timing. But Case 4's Payoff gate failed on its own terms — the idea itself
is no longer scarce, not just poorly timed or poorly framed. Recommend
adding:

- **Tier 0 — Not recommended**: Payoff independently low (the idea is
  already well-served by an active alternative elsewhere in the graph),
  regardless of Friction or Connection cost. Distinct from Tier 4, which
  is for candidates whose *idea* still has value but whose *history* makes
  a plain introduction premature. A revival target should only reach Tier
  0 after failing the Payoff gate specifically — Case 4 is the reference
  example.

This is a small, deliberate addition (one new tier, not a rubric
overhaul) — consistent with how the relationship taxonomy itself picked up
exactly one or two additions per pilot rather than being rebuilt.

**The idea-lineage shape (Case 1) doesn't need a third top-level candidate
type.** It fits inside Tier 1 once Tier 1's definition is read slightly
more broadly: "connect parties who should know about each other" rather
than strictly "introduce two live projects." Recommend widening Tier 1's
description in `ecosystem_alliance_analysis.md` to explicitly cover
idea-originator-to-builder acknowledgment, rather than adding a whole new
candidate shape for what turned out to be one case in the pilot. If Stage
C surfaces more idea-lineage cases that don't fit this reading
comfortably, revisit then — not preemptively.

**Two graph gaps surfaced, not fixed here** (out of scope for scoring,
flagged for a future extraction touch-up): OdooPLM has no linked person
node (Case 3), and openPLM — despite being cited/rejected by name three
times — has no `same_author` edge for its own creator (Case 4). Both are
completeness gaps in Stage 3's extraction, not scoring problems.

## Recommendation

Adopt both small model adjustments (Tier 0, and the widened Tier 1
description) before Stage C, then run the full 146-item Stage A list
through the now-adjusted rubric. No other change to the four-factor model
or the ranking rule is needed — three pilot cases behaved exactly as
designed, and the fourth (Case 4) is what caught the one real gap.
