# Architecture notes — freecad_git_tryout (`levity0815/freecad_git_tryout`)

Read against `doc/technical_convergence_plan.md`'s 8-field template. The
repository was read in full: `README.md` (95 lines — the entire content
is a lab notebook of merge experiments, not a tool's documentation),
`zippey.py` (198 lines, a third-party utility vendored in, not authored
here), `LICENSE`, and the directory structure of the two sample models
(`Toyota_Yaris_-_freecad_assembly/`, `Toyota_Yaris_-_freecad_single_file/`,
`Cardboard_Car_-_freecad_assembly/`). No git commands were run for this
pass.

This is not a tool or workbench — it is an **experiment log**, recording
a real, hands-on attempt to answer the exact question
`doc/optimistic_locking_research.md` surveys from the academic
literature and `doc/technical_convergence_plan.md` frames as this
phase's purpose. Its value to this census is almost entirely in §3 and
§7: it is the single most direct, empirical, reproducible-command-line
evidence anywhere in this series of *why* naive git-plus-FCStd merging
fails and *what* a purely structural (not code-level) workaround buys.

## 1. Repo & basic facts

- **Nature**: a "playground" (README's own word) — no application code
  of its own beyond a vendored third-party script (`zippey.py`, from
  `bitbucket.org/sippey/zippey`, not written by this repo's author).
  Everything else is sample FreeCAD assembly data (two real-world-derived
  models: a Toyota Yaris body-in-white, sourced from NHTSA's public
  LS-DYNA crash-simulation models via the ANSA meshing tool; a simple
  "Cardboard Car" Lego-style toy assembly) plus a README documenting a
  sequence of manual git experiments performed against that data.
- **Dependencies on other projects in this census**: explicitly built on
  top of **`versioncontrol-workbench` (p-friedrich)** — another node
  already in `doc/ecosystem/graph.yaml`
  (`project:versioncontrol-workbench-pfriedrich`) — via its
  `scripts/open_from_folder.py`, and on the third-party **Assembly3**
  FreeCAD workbench (a fork by "realthunder," needed for the specific
  Link-based assembly behavior the experiment relies on). This is a
  genuine, direct dependency edge between two nodes already in the
  census graph that's worth confirming is captured (see §6).
- **License**: the `LICENSE` file's own content is internally
  inconsistent — its first line declares `CC-BY-SA-4.0`, but the URL on
  the very next line points to
  `creativecommons.org/licenses/by-nc-sa/4.0/legalcode.txt` (**NC** =
  NonCommercial — a materially different, more restrictive license than
  plain CC-BY-SA). `doc/ecosystem/graph.yaml` currently records `license:
  unknown`; given the source file's own internal contradiction, "unknown"
  is arguably still the more honest state than picking one of the two
  conflicting labels — flagged in §6 as a case needing a human call
  rather than a mechanical correction.
- **Status/maturity**: `doc/ecosystem/graph.yaml`'s existing note, "Explicit
  experiment/prototype, one month of activity," is accurate and, if
  anything, understates how self-aware the repo is about its own
  purpose — the README never claims to be a tool, only a record of
  trying something out.
- **The README ends mid-experiment**: its final section header,
  "creating and resolving conflicts while working with an assembly of
  multiple files on two branches," has no content beneath it except a
  single external link ("Note: This might lead to something..." pointing
  to a GitHub gist on Git merge-driver customization) — the experiment
  series was never finished or written up. Consistent with the
  one-month activity window already noted in the graph.

## 2. Identity/versioning model

**Plain git, on plain files — but the file *granularity* is the whole
experiment.** No custom identity scheme, no metadata, no numbering — the
entire methodology under test is: **does splitting one large assembly
into one FreeCAD file per part, linked together via `App::Link`
references (the same mechanism EasyPDM's and the Omniverse connector's
macros walk for their own assembly-detection logic), change how well
plain git merge behaves?** This reduces the "identity" question to its
simplest possible form: a part's identity is its own `.FCStd` file's
path, exactly as in every other git-native project in this census
(`HistoryWorkbench`, GitPDM, `freecad-gitproject-reox`) — the novel
contribution here isn't identity scheme design, it's testing the
consequence of **assembly decomposition granularity** on git's own,
completely unmodified merge algorithm.

## 3. Conflict/concurrency strategy — the core finding

This is the most information-dense section of this note; the README's
own experiment log is worth summarizing precisely rather than
paraphrased away:

- **Experiment 1 — single shared file, two branches, two independent
  edits** (`Toyota_Yaris_-_freecad_single_file/...PID0.fcstd`; Alice
  changes one part's color, Bob changes a different part's color, same
  file). Result: **`git merge` fails outright**, and the actual failure
  mode is worth stating precisely because it's not just "binary files
  don't merge" — it's a **compounding failure of the two techniques
  layered on top of raw git**: the repo uses Zippey (a git smudge/clean
  filter transparently unzipping FCStd's zip container so its internal
  XML/text becomes visible to git as plain text) to make the file
  diffable at all; when git's merge machinery produces the standard
  `<<<<<<< HEAD` / `=======` / `>>>>>>>` conflict-marker text and asks
  Zippey to re-encode it (since Zippey's `clean` filter runs on
  merge-driver output too), **Zippey's own decoder crashes** — its
  frame-header parser expects the exact `data_len|raw_len|mode|name`
  format it wrote on encode, and instead receives the literal string
  `<<<<<<< HEAD` where a length integer was expected
  (`ValueError: invalid literal for int() with base 10`). This is a
  **concrete, reproducible instance of the transparent-diff-filter
  pattern colliding with git's own three-way-merge conflict-marker
  convention** — worth flagging as a specific, nameable failure mode any
  future "make FCStd git-mergeable via a smudge/clean filter" proposal
  (a natural-sounding idea) needs to explicitly design around, not just
  discover the hard way as this experiment did.
- Independent of the Zippey crash, manually inspecting the two
  unzipped documents showed the underlying substantive problem clearly:
  **`Document.xml` and `GuiDocument.xml` both changed in ways that
  textually conflict**, even though the two edits (different parts'
  colors) were semantically independent — this is a direct, hands-on
  confirmation of exactly the failure mode
  `doc/optimistic_locking_research.md`'s literature survey frames
  abstractly ("CAD conflicts are... geometric/topological
  incompatibilities... resolving them requires domain expertise a
  generic VCS can't encode" — except here the actual git-level conflict
  isn't even geometric, it's **incidental textual proximity** inside a
  single flat XML document listing every object's properties together,
  regardless of how unrelated those objects are design-wise. **This is
  an important nuance the literature survey's framing slightly
  understates**: at least part of the "CAD conflicts are hard" problem
  demonstrated here is a *storage-layout* problem (unrelated objects'
  data interleaved in one file), not a fundamentally *semantic* one — a
  distinction with real implications for field 8 (see below).
- **Experiment 2 — the assembly-of-linked-files structure, same
  scenario** (edit two different linked part files, one edit each, on
  two branches). Result: **merges cleanly, automatically, with git's
  ordinary file-level merge** — because the two edits landed in two
  entirely separate files. No Zippey crash (implied, not explicitly
  restated, since the files being merged are different), no manual
  conflict resolution needed; opening the merged assembly showed both
  edits correctly present in the combined model. This is a real,
  demonstrated, positive result: **decomposing a monolithic assembly
  into one FCStd file per part, linked rather than embedded,
  reduces CAD merge conflicts to ordinary git file-level conflicts** —
  which only occur when two people edit literally the *same* part, not
  merely two parts that happen to share a container file.
- **The explicitly acknowledged remaining gap**: the README is honest
  that this doesn't solve the *same-part* conflict case — "this will
  not help if two persons work on the same part. This will still result
  in a conflict" — and floats one concrete, never-executed idea for that
  residual case: mechanically construct an assembly containing *both*
  conflicting versions of the disputed part side by side, turning "git
  merge conflict" into "a visible, inspectable geometric choice the user
  resolves in the CAD tool itself," explicitly analogized to how a
  developer resolves conflicting lines in a text merge. This idea was
  never implemented or tested here (the README ends before reaching it)
  — worth flagging as a genuinely interesting, unexplored direction
  rather than a proven technique.

## 4. File format / serialization touchpoints

- **Zippey** (vendored `zippey.py`, third-party, from
  `bitbucket.org/sippey/zippey`) is the one real piece of tooling in
  this repo: a generic (not FreeCAD-specific) git smudge/clean filter
  that transparently unzips a zip-format file for git's benefit (diffing,
  storage) and re-zips it on checkout — installed via ordinary git
  attribute/filter config
  (`git config filter.zippey.smudge/clean`), no FreeCAD-side code
  required at all. This is a **storage/tooling-level answer** to the
  "FCStd is an opaque zip" problem, distinct from and predating PR
  #28312's core-level `DocumentCacheDir` approach and
  `HistoryWorkbench`'s own snapshot-extraction approach — three
  independent answers to closely related problems, worth cross-
  referencing explicitly (see §7).
- **A real, documented gotcha for anyone trying to reproduce or extend
  this**: "Downloading the fcstd files from the website will only
  provide the unzipped content. In order to recover them you have to..."
  run Zippey's decoder manually — i.e. once Zippey's filter is active,
  **the files as they exist in the git working tree/GitHub web UI are
  not directly usable `.FCStd` files** without the filter (or a manual
  decode step) — a real practical friction point of the
  transparent-filter approach, independent of the merge-crash problem
  in §3.
- **Assembly structure**: relies on `App::Link`-based multi-file
  assemblies (same underlying FreeCAD mechanism the EasyPDM and
  Omniverse-connector macros both walk for their own BOM-detection
  logic) plus the Assembly3 third-party workbench specifically — not
  FreeCAD's later native Assembly workbench (this repo predates it;
  Assembly3 was the community's leading Link-based assembly tool at the
  time).

## 5. Dependencies & integration points

- **`versioncontrol-workbench` (p-friedrich)** — a direct, load-bearing
  dependency for opening the multi-file assembly structure
  (`open_from_folder.py`), not just a citation. Both projects are
  already separate nodes in `doc/ecosystem/graph.yaml`; this confirms a
  real, concrete relationship between them worth checking is captured as
  an edge (see §6).
- **Assembly3 (realthunder's FreeCAD fork)** — required for the
  Link-based assembly behavior the whole experiment depends on; not
  independently a node in the census graph as far as checked in this
  pass (not confirmed either way).
- **Zippey** (§4) — a generic, FreeCAD-agnostic git filter tool, not
  FreeCAD ecosystem-specific at all; its own upstream (Bitbucket) is
  outside this census's scope.
- **Real-world data provenance is unusually well-documented for a
  throwaway experiment repo**: the Toyota Yaris geometry is explicitly
  sourced from NHTSA's public crash-simulation LS-DYNA model archive,
  converted via the ANSA meshing tool — the README is careful to caveat
  "this is not a real CAD model as designers would generate it,"
  correctly flagging that FE-mesh-derived geometry may not stress the
  same failure modes as genuinely parametric, feature-tree-based CAD
  data would. Worth keeping in mind when weighing how directly this
  experiment's specific merge-conflict findings generalize to
  parametric-modeling workflows (Sketcher/PartDesign-style, as opposed
  to imported/meshed geometry).

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:freecad-git-tryout-levity0815`
(line 893): category `[version-control]`, `technical_approach: [git]`,
status `dead` as of 2018-08-09, `first_seen: 2018-07-08`, license
`unknown`, existing note "Explicit experiment/prototype, one month of
activity" — confirmed accurate in every particular by this full read;
the one-month window and experimental framing are, if anything,
understated relative to how candidly the README documents its own
incompleteness.

**License left uncorrected, flagged instead**: the `LICENSE` file itself
contradicts its own declared identifier (§1 — labeled `CC-BY-SA-4.0` but
linking to the `by-nc-sa` legal text). Rather than mechanically pick one
of the two conflicting values, this is flagged for a human editorial
call — `graph.yaml`'s current `unknown` is defensible as-is given the
source's own ambiguity, though a note explaining *why* it's unknown
(rather than simply unresearched) would be more informative than a bare
`unknown`. Not applied as an edit in this pass, since asserting either
`CC-BY-SA-4.0` or `CC-BY-NC-SA-4.0` risks being confidently wrong about
a licensing detail that actually matters (NC materially restricts
commercial reuse).

**A likely-missing edge, not yet applied**: this project's direct,
functional dependency on `project:versioncontrol-workbench-pfriedrich`
(§5 — not just a citation, an actual required script dependency) doesn't
appear to be captured as an edge between those two nodes based on the
excerpt read in this pass (full edge list not exhaustively searched).
Worth a targeted `grep` for both node IDs together in a future
graph-consistency pass to confirm whether a `depends_on` or similar edge
already exists; if not, this read is direct-source evidence such an edge
should be added.

## 7. Friction points observed firsthand

- **This repo is the closest thing in the entire census to a
  controlled experiment with a clean negative and a clean positive
  result on the exact question Cluster 2 is asking.** Negative: raw git
  merge on a single shared FCStd fails, compounded by a specific,
  nameable filter-tooling crash (§3). Positive: the same edit scenario,
  restructured as one-file-per-linked-part, merges cleanly with zero
  custom tooling — pure, unmodified git. This is stronger, more directly
  actionable evidence than any prose-only census document could provide,
  and stronger in a specific way than this census's other git-native
  projects: `HistoryWorkbench` and GitPDM both build *tooling* to work
  around FCStd's merge-unfriendliness; this experiment demonstrates a
  **pure workflow/structure change that needs no tooling at all** for
  its positive result.
- **A genuinely important nuance for any future proposal**: the
  root-cause diagnosis here (unrelated objects' data physically
  interleaved in one shared XML document) suggests that at least part of
  "why CAD merge is hard" is a **storage-layout choice**
  (monolithic-document-per-assembly) rather than an irreducibly semantic
  one. This directly complements — and slightly sharpens — the framing
  in `doc/optimistic_locking_research.md`'s literature survey: the
  academic literature focuses on feature-tree/parametric-dependency
  conflicts (a genuinely semantic problem), but this hands-on experiment
  shows a good fraction of *practical* merge pain may be solvable by
  the much simpler, non-semantic move of "don't put unrelated parts in
  the same file" — which is exactly the same underlying instinct behind
  PR #28312's `DocumentCacheDir` (don't put regenerable content in the
  versioned file) and `ose-vcs-library`'s parts/modules/assemblies
  directory-per-entry structure (don't put unrelated designs in the same
  file, ever, from the start).
- **The Zippey/conflict-marker crash (§3) is a specific, previously
  undocumented-in-this-census failure mode** worth carrying forward
  explicitly: any transparent unzip-for-diffing filter approach to FCStd
  needs an explicit strategy for what happens when git's own merge
  conflict markers get fed back through the filter's decoder — this
  experiment shows the naive answer (assume clean, filter-encoded input
  always) breaks in exactly the situation the filter exists to help
  with.

## 8. Minimal-patch hypothesis

- **The single most actionable, low-cost finding in this whole note**:
  the "prefer per-part linked files over a monolithic multi-part
  assembly file" convention, demonstrated here as sufficient (on its
  own, no code) to make git merge behave acceptably for the
  common case (different people editing different parts). This is
  **not a code patch at all** — it's a **workflow/authoring convention**
  any FreeCAD user or team could adopt today, unilaterally, with zero
  tooling, zero cross-project coordination, and zero risk (it degrades
  gracefully to "you still have the monolithic-file problem if you
  don't follow it"). This is about as close to the plan's own opening
  definition of a target patch ("small, low-cost... adopted
  independently... without requiring anyone to merge efforts... beyond a
  tiny shim or convention") as anything found in this entire census —
  except the "shim" here is a documentation/convention recommendation,
  not code.
- **A genuinely valuable, moderate-cost follow-on this experiment
  explicitly floats but never builds**: a git merge driver
  (`.gitattributes` `merge=` custom driver, the exact mechanism the
  README's own closing gist link points toward) that, on detecting an
  unresolvable same-part conflict, **constructs a side-by-side assembly
  of both conflicting versions** rather than emitting raw text conflict
  markers into a binary file. Cost: **moderate** — a real, non-trivial
  merge-driver implementation, but scoped narrowly (only invoked on
  the residual same-file case, since the per-part-file convention above
  already eliminates the common case) and with a clear, testable success
  criterion (does the resulting assembly open and show both variants).
  Worth flagging as a concrete, well-motivated next step for anyone
  picking this specific thread back up — more specific and more
  actionably scoped than "solve CAD merging" in the abstract.
- **The Zippey-plus-conflict-marker interaction (§3, §7)** is a
  narrower, purely defensive fix worth naming for completeness: any
  future transparent-diff-filter design for FCStd (or any zip-based CAD
  format) needs to detect and gracefully bail out of decoding
  merge-conflict-marker text rather than crash — a small, well-scoped
  robustness fix, though only relevant if someone actually revives the
  Zippey-style approach rather than PR #28312's alternative
  cache-dir-based one.
