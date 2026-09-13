# Technical Convergence Plan — Methodology and Repository List

Purpose: the alliance-analysis phase (`ecosystem_alliance_analysis.md`,
`ecosystem_alliance_multiparty_clusters.md`) answered "who should talk to
whom." This phase asks a narrower, more technical question: **are there
small, low-cost code changes that, adopted independently by several
projects, would remove a recurring friction point** — without requiring
anyone to merge efforts, adopt a shared codebase, or coordinate on
anything beyond a tiny shim or convention?

This is a different kind of work than everything done so far. Every prior
document in this series was built entirely from forum/issue *prose* — it
can tell you *where* two projects solved the same problem separately, but
not *how similar their actual code is*, which is the only thing that
determines whether a tiny patch could really bridge them. Answering that
requires reading source code, not more census synthesis.

## Selection methodology for the repository list

Per your instruction: start from every project with at least some overlap
signal with at least one other project, then remove anything without
downloadable source.

**Step 1 — overlap participation.** Of the 57 project nodes in
`ecosystem_graph.yaml`, **43 participate in at least one overlap
relationship** (a Tier S/A/B/C pair from `ecosystem_alliance_overlap_ranking.md`,
or a `rejected_in_favor_of`/`cites_as_prior_art` flag from
`ecosystem_alliance_candidates.md` §2). **14 are isolates** — no overlap
signal with anything else in the census — and are excluded up front:
`billofmaterials-wb`, `cadracks-freecad-workbench-plm`, `cadracks-openplm`,
`fc-plm-ojo42`, `fep-0013`, `freebom`, `openplm-cpg-retail`,
`pdmforfreecad-danmiel`, `pdmworkbench-zhurbav`, `plmore`, `pypdm`,
`schumbi-freecad-plm`, `uesoft-autopdms`, `we-have-pdm-at-home`. (Several
of these do have real, findable repos — they're excluded on the overlap
criterion, not on downloadability.)

**Step 2 — downloadable source.** Of the 43 remaining, four don't have
source code to compare and are excluded here instead:

| Excluded | Why |
|---|---|
| `project:anchorpoint` | Proprietary commercial product — confirmed no public repo (only a product blog post, `anchorpoint.app/blog/git-with-freecad`, describing the feature). Still relevant to Cluster 2 below; its architecture doc will have to be written from public documentation, not source. |
| `project:grabcad-workbench` | Proprietary (Stratasys), dead, discontinued — the only public trace found is a third-party retrospective blog post, not a repo. |
| `project:fep-0011` | This is an RFC/proposal document (`FreeCAD-Enhancement-Proposals` repo), not a software project — there's no implementation to read yet. |
| `project:issue-25681-cluster` | A GitHub issue/discussion cluster on `FreeCAD/FreeCAD`, not a separate repo. Its associated *code* (if any exists yet) lives inside `pr-26306`/`pr-28312`, both already in the list below as diffs against FreeCAD core. |

That leaves **39 repositories** to download and compare.

## The repository list

Confirmed URLs are pulled directly from the two census documents (grep'd,
not retyped from memory). Where a URL wasn't found there, it's marked
accordingly — verify before downloading rather than trusting a guess.

### Priority group 1 — the two confirmed multi-party convergence clusters

Read these first; they have the strongest existing evidence of a shared
friction point (`ecosystem_alliance_multiparty_clusters.md`).

| Project | Repo | Confirmation |
|---|---|---|
| `easypdm` | https://github.com/pawelcel/EasyPDM | confirmed in census |
| `freecad-omniverse-connector` | https://github.com/Metaverse-Colab-for-Fusion-Energy/FreeCAD-Omniverse | confirmed in census |
| *(`anchorpoint` — no source; see exclusions above)* | | |
| `ondsel-server` | https://github.com/FreeCAD/Ondsel-Server | confirmed in census |
| `ose-vcs-library` | https://github.com/OpenSourceEcology/vcs-library | confirmed in census |
| `openpartslibrary` | https://github.com/alekssadowski95/OpenPartsLibrary (+ `-Flask`, `-packages`, `-website` variants) | confirmed in census |

### Priority group 2 — the git/version-control cluster (Tier B, 7 projects)

Weaker evidence (facet-match only, no observed edge — see Stage A's own
caveat that several of these likely deserved a stronger edge type), but a
dense, well-defined cluster worth checking directly against source.

| Project | Repo | Confirmation |
|---|---|---|
| `historyworkbench` | https://github.com/eblanshey/HistoryWorkbench | confirmed in census |
| `pr-28312` | `git fetch origin pull/28312/head:pr-28312` inside a clone of https://github.com/FreeCAD/FreeCAD (or `https://github.com/FreeCAD/FreeCAD/pull/28312.diff` for the diff alone) | confirmed mechanism — PR refs are fetchable directly from GitHub |
| `gitpdm` | https://github.com/nerd-sniped/GitPDM | confirmed in census |
| `freecad-gitproject-reox` | https://github.com/reox/FreeCAD_gitproject | confirmed in census |
| `freecad-git-tryout-levity0815` | https://github.com/levity0815/freecad_git_tryout | confirmed in census |
| `cadracks-freecad-workbench-git` | https://github.com/cadracks-project/freecad-workbench-git | confirmed in census |
| *(`anchorpoint` already listed above)* | | |

### Remaining Tier S/A participants (documented awareness or observed convergence, not yet in groups above)

| Project | Repo | Confirmation |
|---|---|---|
| `bcf-plugin-freecad` | https://github.com/podestplatz/BCF-Plugin-FreeCAD | confirmed in census |
| `freecad-cloud-browser-hitclawagent` | https://github.com/hitclawagent/freecad-cloud-browser | confirmed in census |
| `freecad-cloud-browser-sabi137032` | https://github.com/sabi137032/freecad-cloud-browser | confirmed in census |
| `versioncontrol-workbench-pfriedrich` | https://github.com/p-friedrich/versioncontrol-workbench | confirmed in census |
| `cadbaselibrary` | https://gitlab.com/cadbase (real dev home) / https://github.com/mnnxp/cadbaselibrary-freecad (FreeCAD-side mirror) | confirmed in census |
| `freepdm` | https://github.com/grd/FreePDM | confirmed in census |
| `nanoplm` | https://github.com/alekssadowski95/nanoPLM | confirmed in census |
| `ondsel-lens-addon` | https://github.com/FreeCAD/Ondsel-Lens-Addon | confirmed in census |
| `taack-plm` | https://github.com/Taack/taack-plm-freecad | confirmed in census |
| `pr-26306` | `git fetch origin pull/26306/head:pr-26306` inside a clone of https://github.com/FreeCAD/FreeCAD (or `https://github.com/FreeCAD/FreeCAD/pull/26306.diff` for the diff alone) | confirmed mechanism — PR refs are fetchable directly from GitHub |

### Remaining Tier B/C participants (weaker facet-only overlap)

| Project | Repo | Confirmation |
|---|---|---|
| `openplm` | https://github.com/amarh/openPLM | confirmed in census |
| `odooplm` | https://github.com/OmniaGit/odooplm | confirmed in census |
| `docdoku-plm` | https://github.com/docdoku/docdoku-plm | confirmed in census |
| `cadracks-opm` | https://github.com/cadracks-project/opm | confirmed in census |
| `comet-cdp4` | https://github.com/STARIONGROUP/COMET-WebServices-Community-Edition (and related STARIONGROUP repos) | confirmed in census |
| `cadcloud` | https://github.com/opencomputeproject/CADCloud | confirmed in census |
| `ose-library-site` | https://github.com/OpenSourceEcology/ose-library-site | confirmed in census |
| `ose-library-workbench` | https://github.com/OpenSourceEcology/ose-library-workbench | confirmed in census |
| `cascadia-app` | https://github.com/Cascadia-PLM/Cascadia-App | confirmed in census |
| `cascadia-freecad-connector` | https://github.com/Cascadia-PLM/FreeCAD-Connector | confirmed in census |
| `pistock` | https://github.com/pistock-org/PiStock | confirmed in census |
| `freecad-parts-library` | https://github.com/FreeCAD/FreeCAD-library (from the project's `name` field — not independently found via a direct census URL match, verify before downloading) | **unconfirmed** |
| `freecad-web-virtastic` | https://github.com/Virtastic/freecad-web | confirmed in census |
| `magiknet-freecad-port` | https://magik.net/freecad/ (a WASM port site — check whether source is actually public, not just the demo) | **unconfirmed as an open repo** |
| `speckle-opencascade` | https://github.com/ENAC-CNPA/speckle-opencascade | confirmed in census |
| `collaborativefc-ocp` | not found in either census doc under a direct URL — GitHub handle `ickby` (Stefan Tröger) is confirmed as the author; search `github.com/ickby` at download time | **unconfirmed** |
| `jupytercad` | well-known project, canonical home is `github.com/jupytercad/JupyterCAD` from general knowledge — **not confirmed against these census docs**, verify before downloading | **unconfirmed** |
| `ifcopenshell` | well-known project, canonical home is `github.com/IfcOpenShell/IfcOpenShell` from general knowledge — **not confirmed against these census docs**, verify before downloading | **unconfirmed** |

**39 repositories total** (37 separate clones + 2 PR diffs against
`FreeCAD/FreeCAD`), 5 of which need a quick verification step before
downloading since this pass didn't find a confirmed URL in the source
docs.

## How I recommend we proceed (answering your structure question)

Free-form "read them all and discuss" works fine at the scale of the two
priority clusters (5-7 repos) — it's exactly how the pairwise alliance
analysis was piloted before scaling up. At 39 repos, I'd add **one small
piece of structure**: a short, fixed-field template for each
architecture doc, so that after the 30th repo the notes are still
comparable to the 1st, and so the final "is there a tiny shared fix"
judgment doesn't require re-reading everything. This isn't the heavier
multi-stage pipeline the alliance analysis used — just enough shape that
comparison doesn't degrade with volume:

1. **Repo & basic facts** — language(s), rough size, last real activity, license.
2. **Identity/versioning model** — how does it represent "this is the same
   part/file across edits or collaborators"? (The single most relevant
   question for Cluster 1's friction point.)
3. **Conflict/concurrency strategy** — lock, merge, versioning, none.
   (The key question for Cluster 2.)
4. **File format / serialization touchpoints** — what it reads/writes,
   any sidecar files, any embedded schema.
5. **Dependencies & integration points** — libraries used, FreeCAD API
   surfaces touched, external services.
6. **Graph cross-reference** — which `ecosystem_graph.yaml` node/edges this
   repo corresponds to, so the architecture doc and the relationship graph
   stay linked.
7. **Friction points observed firsthand** — does reading the code confirm
   the census's hypothesis, or reveal something the prose missed?
8. **Minimal-patch hypothesis** — a first-pass guess at what a tiny shared
   shim/spec/convention would need to look like for this repo to
   interoperate, with a rough cost label (trivial / small / moderate /
   large / not realistic).

Field 8 is the one that actually answers your original question, and it's
deliberately last — a guess about the patch is much more grounded once
1-7 are on paper for a repo, and premature guessing at field 8 is exactly
the kind of "false structure" worth avoiding.

**Process**: as you download each repo, tell me its local path and we go
one at a time, in the priority order above. I'll read it and produce the
architecture doc against this template (suggest saving them under
`doc/architecture/<project-id>.md`). After the two priority clusters are
done (7 repos), pause and check whether the template is actually
producing a usable "minimal patch hypothesis" before continuing through
the other 32 — same pilot-before-scaling logic used everywhere else in
this project.

## What this phase does not attempt

No proposal, patch, or spec gets written in this plan document — this is
the repository list and the reading template only, mirroring how
`ecosystem_relationship_mapping_plan.md` and `ecosystem_alliance_analysis.md`
were kept separate from their own downstream deliverables.
