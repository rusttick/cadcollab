# Stage D — Full Feasibility Scoring, and Stage E — Tiering

Applies the four-factor model to the 15 candidates that cleared Tier S/A in
`ecosystem_alliance_overlap_ranking.md` (Goal alignment is inherited from
that ranking, not re-derived here). One of the 15 —
`issue-25681-cluster ↔ ondsel-server` — was already scored in the Stage C
pilot (`ecosystem_alliance_stage_b_pilot.md`, Case 2); its result is
carried forward unchanged. Tier B's 35 candidates are not scored
individually here, per the plan — they remain a secondary queue.

Contact identification is still deferred (per the plan's sequencing note)
— "Connection cost" below only asks *whether* a bridge exists, not who
specifically to contact.

## Tier S pairs

| Pair | Connection cost | Friction | Payoff | Notes |
|---|---|---|---|---|
| `issue-25681-cluster` ↔ `ondsel-server` | Very low | Low | High | Carried from Stage C pilot — `person:pierreporte` already bridges both. |
| `historyworkbench` ↔ `pr-28312` | Low-Medium | Low | **High** | HistoryWorkbench is the single highest-starred project in the entire census (144 stars) — the strongest revealed-preference signal that the community wants exactly what PR #28312 is trying to build into core, from the opposite direction (external addon vs. internal core change). No existing bridge, but both are highly visible. |
| `ondsel-server` ↔ `ose-vcs-library` | Medium | Low | **High** | Open Source Ecology is a resourced, active institution that just (2026-09-07) converged on nearly the same item/part/assembly vocabulary as Ondsel-Server#48. No existing bridge between the two organizations. |
| `bcf-plugin-freecad` ↔ `fep-0011` | Low | Low | Medium-High | Dead project, but FEP-0011 doesn't just cite it — it explicitly states intent to let BCF-based workflows be *rebuilt on top of* the new framework (`shares_standard` note). This is a revival case with an unusually strong, already-declared receiving side. |
| `anchorpoint` ↔ `easypdm` | Medium | Low | Medium | Commercial product vs. solo AI-assisted hobbyist tool — different worlds, no bridge, but easy to reach both. Genuine "lock, don't merge" convergence. |
| `anchorpoint` ↔ `freecad-omniverse-connector` | Medium-High | Low | Medium | Commercial product vs. EUROfusion/UKAEA-funded research infrastructure — furthest apart institutionally of any Tier S pair; convergence is real but the domains barely overlap otherwise. |
| `issue-25681-cluster` ↔ `openpartslibrary` | Medium | Low | Medium | The underlying `independently_reinvents` edge is itself flagged in the graph as "weaker/more speculative... included to test whether the taxonomy handles a lower-confidence case." Score inherits that uncertainty — worth a closer read before treating this as equal-strength to the others above it. |
| `freecad-cloud-browser-hitclawagent` ↔ `freecad-cloud-browser-sabi137032` | Low | Low (but see note) | Low-Medium | Same name, overlapping features, created three days apart, "no visible relationship between the authors" per the graph's own note — an odd case that could be coincidence, undisclosed forking, or something else. Worth clarifying the actual relationship before framing any outreach, not assuming good-faith parallel invention. |
| `freecad-gitproject-reox` ↔ `versioncontrol-workbench-pfriedrich` | High | Low | **Low** | Both dead since 2018. The idea itself ("git-wrapper for FreeCAD") is now far better served by `historyworkbench` and `pr-28312` — reviving either scaffold adds nothing unavailable elsewhere. Payoff gate fails on its own terms, same pattern as the Stage C pilot's openPLM case. |

## Tier A pairs

| Pair | Connection cost | Friction | Payoff | Notes |
|---|---|---|---|---|
| `freepdm` ↔ `ondsel-server` | Very low | Low | Medium-High | FreePDM's own thread already asked, unanswered, whether it's based on Ondsel. The door is objectively open — closing this loop is close to the cheapest possible win in the whole candidate set. |
| `cadbaselibrary` ↔ `fep-0011` | Low | Low | Medium | Already cited in FEP-0011's Motivation section. `cadbaselibrary`'s Russian origin (`origin_country: RU`, `origin_language: RU`) is worth flagging as a possible language-bridge consideration for later outreach, not a blocker now. |
| `fep-0011` ↔ `freepdm` | Low | Low | Medium | Already cited; architecturally distant (custom-filesystem-permissions vs. an abstract framework) limits deep-merge potential, but valuable as acknowledged prior art. |
| `fep-0011` ↔ `nanoplm` | Low | Low | Low-Medium | Already cited, but nanoPLM's own technical detail is thin (`technical_approach: unknown` — part of the lighter-treatment alekssadowski95 cluster), limiting confidence in payoff beyond a courtesy citation. |
| `fep-0011` ↔ `ondsel-lens-addon` | Low | Low | Medium-High | Effectively the addon-side sibling of the `issue-25681-cluster ↔ ondsel-server` Tier S pair — recommend folding into that same outreach thread rather than treating as a separate conversation. `ondsel-lens-addon` itself has no `same_author`/`contributes_to` edge in the graph at all — a minor completeness gap, not fabricated here. |
| `fep-0011` ↔ `taack-plm` | Low | Low | Low-Medium | Already cited. **Correction to a claim in the original mapping plan**: `ecosystem_relationship_mapping_plan.md`'s own worked taxonomy table cites "grd calling Taack PLM 'alien' (Java, unknown maintainers)" as an example `rejected_in_favor_of` case. Checked against the source census text for this pass — no such quote about Taack exists there; the only "alien" quote found is grd calling his *own* FreePDM code "too alien for the python guys" (§16, Phase 5), unrelated to Taack. The Java/PLM detail also matches `docdoku-plm`, not Taack. This looks like an inaccuracy in the plan's own illustrative example, not a missed extraction — no edge was fabricated to match it. |

## Resulting tier assignments (Stage E)

- **Tier 2 — propose active consolidation** (high alignment, both sides
  active, real leverage): `issue-25681-cluster ↔ ondsel-server`,
  `historyworkbench ↔ pr-28312`, `ondsel-server ↔ ose-vcs-library`. These
  three are the standout candidates in the entire dataset so far — two of
  them (`historyworkbench`/`pr-28312` and `ondsel-server`/`ose-vcs-
  library`) weren't visible as "the best candidates" until this scoring
  pass; Stage A's mechanical generation surfaced them as raw pairs but
  didn't distinguish them from the rest of Tier S.
- **Tier 3 — revival via reframing**: `bcf-plugin-freecad ↔ fep-0011`
  (FEP-0011 has already declared intent to build on it) and
  `freecad-cloud-browser-hitclawagent ↔ freecad-cloud-browser-sabi137032`
  is *not* placed here — see Tier 4.
- **Tier 1 — just make the introduction / close an open loop**:
  `anchorpoint ↔ easypdm`, `freepdm ↔ ondsel-server`,
  `cadbaselibrary ↔ fep-0011`, `fep-0011 ↔ freepdm`, `fep-0011 ↔ nanoplm`,
  `fep-0011 ↔ ondsel-lens-addon` (bundle with the Tier 2 Ondsel-Server
  thread), `fep-0011 ↔ taack-plm`.
- **Tier 4 — needs a different framing before outreach**:
  `anchorpoint ↔ freecad-omniverse-connector` (real convergence, but the
  two domains are institutionally and technically far apart),
  `issue-25681-cluster ↔ openpartslibrary` (the underlying edge is itself
  flagged low-confidence — worth a closer read first), and
  `freecad-cloud-browser-hitclawagent ↔ freecad-cloud-browser-sabi137032`
  (the relationship between the two near-identical repos needs clarifying
  before any collaboration framing is proposed).
- **Tier 0 — not recommended**:
  `freecad-gitproject-reox ↔ versioncontrol-workbench-pfriedrich` (Payoff
  gate fails — the idea is already well served by the Tier 2
  `historyworkbench ↔ pr-28312` pair above).

## What this surfaced beyond individual scores

The overlap ranking alone (Stage B) treated all 9 Tier S pairs as
equally strong. Running the feasibility layer on top of it reordered them
substantially: the three Tier 2 candidates weren't distinguishable from
the rest of Tier S until Connection cost and Payoff were actually
assessed. This is the clearest demonstration yet of why the plan kept
overlap scoring and feasibility scoring as two separate layers rather than
one blended score — collapsing them would have buried
`historyworkbench ↔ pr-28312` (a very high-payoff, currently-invisible
pairing) at the same apparent priority as
`freecad-cloud-browser-hitclawagent ↔ …-sabi137032` (a minor, ambiguous
scaffold duplication), just because both carry an `independently_reinvents`
edge.
