# Ecosystem Relationship Mapping — Stage 1–3 Results

Executes Stages 1–3 of `ecosystem_relationship_mapping_plan.md`. The
structured data lives in [`ecosystem_graph.yaml`](ecosystem_graph.yaml);
this document summarizes what was built and what it revealed. Stage 2
(taxonomy freeze) and Stage 3 (full extraction) are now **complete**. Stages
4 (focused visualizations) and 5 (synthesis document) are **not started** —
see recommendation at the end.

## Stage 2: taxonomy frozen as v1 (2026-09-13)

Per the Stage 1 pilot's findings, two changes were made and then the
taxonomy was frozen — see the header comment block in
`ecosystem_graph.yaml` for the authoritative table:

1. **New edge type `contributes_to`** (person → project): sustained
   non-authoring involvement — design feedback, advising, attempted-then-
   abandoned implementation work. Retrofitted the pilot's dan-miel/FreePDM
   `same_author` workaround edge to this type, and used it for four more
   Stage 3 cases (heda, zolko, user1234 → FreePDM; pierreporte, creymore →
   Ondsel-Server).
2. **New field `chose_instead`** on `rejected_in_favor_of`, for "rejected X,
   built something bespoke" rather than "rejected X, adopted Y." Retrofitted
   all three of FreePDM's pilot-era `rejected_in_favor_of` edges (which had
   used `to: unknown` as a workaround) to `to: null` + `chose_instead`.

No edge types were removed — all 11 types (the original 10 plus
`contributes_to`) are used at least once in the full graph.

## Stage 3: full extraction — scope and counts

Went section-by-section through `freecad_pdm_plm_ecosystem_census.md`
(§10–17) and `freecad_core_and_lens_collaboration_issues.md` (full document,
including all issue/PR/FEP/forum content), extending the pilot's 34
nodes/27 edges to the full census.

**95 nodes**: 57 projects, 25 people, 11 discussions, 2 standards.
**61 edges**, all 11 taxonomy types represented:

| Edge type | Count |
|---|---|
| `discussed_in` | 14 |
| `independently_reinvents` | 11 |
| `cites_as_prior_art` | 10 |
| `same_author` | 9 |
| `contributes_to` | 6 |
| `rejected_in_favor_of` | 4 |
| `shares_standard` | 2 |
| `mentors` | 2 |
| `superseded_by` | 1 |
| `aware_of_uncoordinated` | 1 |
| `forked_from` | 1 |

Confidence split: 49 `confirmed`, 12 `inferred`. (One `discussed_in` edge
— `project:ondsel-server` → `discussion:ondsel-server-48` — was added after
initial Stage 3 extraction: the alliance-analysis Stage A pass found the
discussion node had no link to the project it's plainly about, a genuine
gap rather than a deliberate omission.) Two edges explicitly considered and rejected during extraction
are documented as YAML *comments*, not edges, precisely because no
primary-source support existed: a speculative `wmayer → CADBaseLibrary`
`cites_as_prior_art` edge, and an `OdooPLM ↔ FEP-0011` mutual-unawareness
relationship (real per the prose, but not expressible as
`aware_of_uncoordinated`, which requires evidenced awareness — see Stage 2
finding below). Leaving a documented non-edge is deliberate: it's the
taxonomy's own "don't invent edges speculatively" rule working as designed,
and it's worth a future reader knowing the case was looked at and declined,
not just silently absent.

**Stub-level treatment**: per the plan's "Open Questions" recommendation,
single-commit scaffolds and unresolved/ruled-out leads (`PLMore`, several
`cadracks-project` scaffolds, the two duplicate `freecad-cloud-browser`
repos, `openPLM/openplm`'s name-collision entry) got minimal facets
(`unknown` explicit rather than omitted) rather than deep research — no new
web research was performed in Stage 3, per the plan's framing.

## Notable findings — some only visible at full scale

1. **The "independently reinvents" pattern is not confined to PDM
   architecture — it recurs at the level of specific, narrow technical
   choices.** Beyond the Riegel-2009 → Ondsel-Server#48 → VDI-2770/AAS chain
   already found in the pilot, Stage 3 surfaced:
   - A **three-hop identity-sidecar lineage**: openfablab's 2020 UUID-JSON-
     sidecar proposal → thomas-neemann's 2021 sidecar-text-file proposal →
     FreePDM's 2023 numbered-file-revision filesystem — with `person:user1234`
     directly traceable as the same individual independently arguing the
     identical "PostgreSQL for check-out locking" position in *both* the
     2021 thread and the 2022 FreePDM thread, six months apart, evidently
     without realizing (or at least without stating) they were re-running
     the same argument. This is the clearest "one person unknowingly
     carrying an idea across threads" case in the entire census — the plan
     doc's own worked example flagged this cluster as promising, and it
     turned out to be.
   - A **"lock, don't merge" pattern spanning three unrelated domains**:
     `EasyPDM`'s item check-out locking (hobbyist, AI-assisted FreeCAD PDM),
     `Anchorpoint`'s automatic `.FCStd` file-locking (commercial "git for
     CAD" product), and the FreeCAD-Omniverse connector's checkpoint-token
     model (EU/UKAEA-funded fusion-energy infrastructure) all independently
     converge on the same conflict-avoidance strategy — hobbyist tooling,
     commercial product, and government-funded research infrastructure,
     none aware of the other two.
   - **`OpenSourceEcology/vcs-library`'s parts→modules→assemblies→structures
     hierarchy** independently converges on essentially the same part-of
     vocabulary as `Ondsel-Server#48`'s item/part/assembly/sub-assembly
     terms — a fourth independent origin (an institutional open-hardware
     org) landing on the same idea as FreeCAD's founder (2009), FreeCAD's
     current team (2025-26), and an international standards body (VDI/AAS).

2. **`same_author` fan-out beyond pieterhijma**: the pilot already found
   pieterhijma (5 projects) and alekssadowski95 (5 projects) as the two
   largest personal clusters. Stage 3 confirms a third, institutional-scale
   instance — Open Source Ecology (3 coordinated repos, pushed the same day,
   2026-09-07) — matching the plan's own §10.9 framing almost exactly. Read
   together, the graph now shows the "single-maintainer/single-team personal
   ecosystem" pattern recurring at three different scales (individual,
   individual, institution) rather than being a pieterhijma-specific
   curiosity.

3. **`aware_of_uncoordinated` stayed at exactly one edge even after full
   extraction**, and the near-miss (OdooPLM/FEP-0011) is informative by its
   absence. The census prose (§10.8 point 4) calls OdooPLM and FEP-0011
   "completely unaware of each other" — a strong claim — but on inspection
   this is *mutual absence of citation*, not *documented awareness without
   coordination*, which is what the edge type actually models. The taxonomy
   currently has no edge type for "provably isolated from" (as opposed to
   "aware of, but uncoordinated with") — flagged as a real gap, not a
   research failure (see recommendation below).

4. **The `rejected_in_favor_of`/`chose_instead` split earns its keep
   immediately**: all three of FreePDM's `rejected_in_favor_of` edges turned
   out to be the "bespoke" case, not the "adopted a named competitor" case —
   grd rejected SnowFS, OpenPLM, and eventually his own SVN/git framing, and
   built something bespoke each time. Had the field not existed, all three
   edges would still be using the `to: unknown` workaround the pilot
   flagged as a taxonomy smell.

## Taxonomy-fit review after full-scale use

The v1 taxonomy (frozen after the pilot) held up well across the full
census — no further edge-type changes are recommended. Two soft
observations, neither urgent enough to warrant reopening the freeze:

- **`independently_reinvents` needed person→person and project→discussion
  targets**, not just project↔project, to express the openfablab/
  thomas-neemann/FreePDM chain and the OSE/Ondsel-Server#48 pair. The
  taxonomy table's original framing ("project A ↔ project B") was already
  loose enough to accommodate this without a schema change — worth noting
  explicitly for a future reader rather than silently relying on it.
- **A `provably_isolated_from` or similar edge type** would have let the
  OdooPLM/FEP-0011 mutual-unawareness finding be graphed rather than left
  as a YAML comment. Not added now — it would be the only edge type in the
  whole taxonomy that asserts a *negative* relationship (confirmed absence
  of connection), which is a different kind of claim from every other edge
  type here and deserves its own deliberate design pass rather than a
  same-day addition. Flagged for a future Stage 2.5 if this pattern recurs.

## Recommendation for next steps

1. **Stage 4 (focused visualizations)** is now well-supported by the data:
   the plan's own suggestions — a `same_author` cluster diagram, an
   `independently_reinvents` diagram grouped by `shared_idea`, and a
   `technical_approach` facet matrix — would all render meaningfully at the
   current 95-node/60-edge scale. Not attempted in this pass; still future
   work.
2. **Stage 5 (synthesis document)** — `ecosystem_relationship_findings.md`
   — is now well-positioned to be written: the four "notable findings" above
   are exactly the kind of graph-backed claims the plan describes that prose
   couldn't have made as sharply on its own. Not attempted in this pass.
3. Consider the `provably_isolated_from` edge-type question (above) before
   Stage 5, since at least one finding worth narrating (OdooPLM/FEP-0011)
   currently has no home in the graph itself.
