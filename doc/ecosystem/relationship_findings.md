# Ecosystem Relationship Findings

This is the Stage 5 deliverable of `ecosystem_relationship_mapping_plan.md`
— the synthesis the structured graph exists to support. The graph itself
(`ecosystem_graph.yaml`, 95 nodes / 61 edges) and the six focused diagrams
built from it (`ecosystem_visualizations.md`) are supporting data; this
document is what that structure lets us say that the prose census docs,
read on their own, couldn't say as sharply. Every claim below cites a
specific node or edge id so it can be checked against the graph, and
through the graph's `source`/`source_doc` fields, against the original
forum posts and issues.

## The ecosystem keeps rebuilding the same handful of ideas

If there is one throughline finding across Stages 1-3, it's this: the
FreeCAD PDM/PLM space does not lack ideas. It has, independently, arrived
at the same small set of ideas — stable item identity outside the file,
centralized revision control, and "lock rather than merge" conflict
avoidance — somewhere between five and eleven times, depending on how
narrowly you count. What it lacks is any mechanism for one attempt to learn
that a previous attempt existed.

The clearest instance spans sixteen years and four completely unrelated
origins. In 2009, FreeCAD co-founder Jürgen Riegel filed ticket
`discussion:freecad-issue-5539`, sketching UUID-based stable item identity,
a central repository with revision control, and HTTP-accessible resource
discovery. That ticket sat open for fourteen years before being closed in
2023 "as replaced by Component Library" (`superseded_by` →
`project:freecad-parts-library`) — a real but much narrower answer: a
shared macro/part-file repository with no UUID scheme and no revision
control beyond whatever git happens to provide. Riegel's actual vision went
unbuilt. Then, in 2025-2026, FreeCAD's current official collaboration
effort re-derived essentially the same core ideas from scratch in
`discussion:ondsel-server-48` (the "Extent to which Lens is a PDM"
discussion) — an item/document/version/meta-document vocabulary that the
graph's `independently_reinvents` edge from `discussion:freecad-issue-5539`
marks as inferred, because neither side cites the other, but the
conceptual overlap (stable identity, a version chain, a document layer on
top) is not subtle. That discussion, in turn, independently reinvented a
third time: its item/document/version split turns out to match, almost
term for term, VDI 2770's Document/DocumentVersion model and the Asset
Administration Shell's Handover Documentation submodel — and this one *is*
marked `confidence: confirmed`, because a primary-source comment on the
issue itself draws the correspondence out explicitly (`source`: §5, comment
5.1). A fourth, independent voice lands on the same shape from yet another
direction: Open Source Ecology's `project:ose-vcs-library`, published
2026-09-07 with no visible awareness of Lens, FreePDM, or FEP-0011, defines
a parts→modules→assemblies→structures hierarchy that is structurally the
same part-of vocabulary. FreeCAD's own founder, FreeCAD's current team, an
international industrial standards body, and an open-hardware institution
— four origins, one idea, never linked (see diagram 2a in
`ecosystem_visualizations.md`).

A second, tighter lineage makes the same point at smaller scale and with
more human specificity. In 2020, a forum poster called `person:openfablab`
proposed a UUID-based, filesystem-native PDM design keyed off small
`info.plm` JSON sidecar files — explicitly rejecting server-based
approaches like OpenPLM as too heavy (`rejected_in_favor_of`, confirmed,
`freecad_pdm_plm_ecosystem_census.md` §16.3a). A year later,
`person:thomas-neemann` proposed almost the same shape from a different
angle: pairing FreeCAD files with sidecar text files and querying them with
the Linux `recoll` tool. Neither post references the other. Then, in 2022,
FreePDM's own genesis thread reran the same design conversation for a third
time — and here the graph surfaces something the prose treats as two
separate observations but is really one: `person:user1234`, a recurring
voice in the FreePDM thread arguing that "PostgreSQL" is the right answer
for check-out locking, turns out to be independently traceable to
thomas-neemann's 2021 thread, arguing the *identical* position there, six
months earlier. This is marked `confidence: confirmed` rather than the
usual inferred default for `independently_reinvents`, because the identity
match is directly evidenced, not pattern-matched by this research. One
person carried an idea across two communities without, as far as the
record shows, ever noticing it was the same idea twice.

A third pattern operates at the level of a specific mechanism rather than
an architecture: "lock, don't merge" as the answer to concurrent-edit
conflicts on large binary CAD files. `project:easypdm` — a hobbyist tool
whose README states it was "written for me by Claude," by a self-described
non-programmer — implements item check-out locking. `project:anchorpoint`,
a commercial "git for CAD" product, implements automatic file-locking on
`.FCStd` binaries layered on top of git. `project:freecad-omniverse-connector`,
a peer-reviewed, EUROfusion/EPSRC/UKAEA-funded fusion-energy research tool,
implements a checkpoint-token model that is the same strategy under a
different name. A hobbyist, a commercial vendor, and a government-funded
research consortium reached the same conclusion about how to handle
concurrent edits, with zero evidence any of the three knew the other two
existed (diagram 2c). And a fourth pattern — "make `.FCStd` git-friendly"
— shows the same convergence purely as a *popularity* signal: at least six
independent attempts exist in the census (`project:historyworkbench`,
`project:pr-28312`/FEP-0013 from inside FreeCAD core itself,
`project:freecad-gitproject-reox`,
`project:versioncontrol-workbench-pfriedrich`,
`project:freecad-git-tryout-levity0815`, and
`project:cadracks-freecad-workbench-git`). `project:historyworkbench` is,
not coincidentally, the single highest-starred project in the entire
census (144 stars) — the strongest revealed-preference signal available
that what the community actually wants is git-friendly diffing and
history, not server-based PDM, even though server-based PDM is what gets
the most sustained institutional attention (FEP-0011, Ondsel-Server).

None of this is a story about incompetence. It's a story about discovery
cost. Every one of these reinventions happened inside the same forum, the
same GitHub organization, or a directly adjacent open-source niche — and
still didn't connect. The tightest-timescale instance in the whole census
underscores this: `project:freecad-cloud-browser-hitclawagent` and
`project:freecad-cloud-browser-sabi137032` are two repositories with the
*same name* and the *same feature set*, created three days apart in May
2026, with the graph's `independently_reinvents` edge between them marked
`confidence: confirmed` because the naming collision itself is direct
evidence, not inference (diagram 2e). If two people can independently
publish identically-named projects three days apart without either finding
the other, the sixteen-year Riegel-to-Ondsel gap is less surprising than
it first sounds — it's the same failure mode, just stretched further in
time.

## Bus-factor risk is structural, not anecdotal, and it recurs at three scales

The census prose already notes, in passing, that individual contributors
sometimes build several related projects. The graph makes visible that
this isn't an occasional curiosity — it's the dominant organizational shape
in this ecosystem, and it recurs at three distinct scales with the same
signature: one `same_author` node with high fan-out, one active thread of
development, and no visible succession plan.

At the individual scale, `person:pieterhijma` singularly drives FreeCAD
core's entire official collaboration effort — five interlocking nodes
(`project:fep-0011`, `project:pr-26306`, `project:issue-25681-cluster`,
`project:pr-28312`, `project:fep-0013`), all active as of this research
pass (diagram 1a). This is the most institutionally significant cluster in
the census — it's the officially adopted FreeCAD Enhancement Proposal path
— and it rests on one person. Separately, and with no connection to
pieterhijma's work, `person:alekssadowski95` drives a second five-project
cluster toward "PDM for small manufacturers" (`project:nanoplm`,
`project:freebom`, `project:we-have-pdm-at-home`,
`project:openpartslibrary`, `project:pypdm` — diagram 1b). Two people,
independently, are each single-handedly responsible for roughly a fifth of
all *actively maintained* projects in this census.

The third instance generalizes the pattern past the individual level
entirely. `person:ose-org` — Open Source Ecology, an open-hardware
nonprofit, not a person — pushed three coordinated repositories
(`project:ose-vcs-library`, `project:ose-library-workbench`,
`project:ose-library-site`) on the same day, 2026-09-07 (diagram 1c). The
`same_author` edge type, designed around individual authorship, turns out
to describe institutional authorship just as cleanly — which suggests the
underlying risk (one committed party's continued interest is the only
thing keeping a cluster of interlocking tools alive) is a property of *how
this space gets built*, not of any one contributor's working style.
`FreePDM` shows the inverse shape and is worth naming for contrast: one
author (`person:grd`) but five distinct sustained `contributes_to`
relationships (`person:dan-miel`, `person:heda`, `person:zolko`,
`person:user1234`) plus a parallel exploratory fork
(`person:jee-bee`, `forked_from`) — diagram 1d. It is the only cluster in
the census with a real, if informal, community around a single author,
rather than a single author working in isolation. That the `contributes_to`
edge type had to be invented at the Stage 2 taxonomy freeze specifically to
describe this shape (see `ecosystem_relationship_mapping_results.md`) is
itself informative: the original ten-type taxonomy, built from real
observed patterns, still had no vocabulary for "sustained community support
without authorship" until FreePDM's own history forced the gap into view.

## Awareness is asymmetric, and the asymmetry has a shape

The `aware_of_uncoordinated` edge type was designed to capture something
weaker than `cites_as_prior_art` but stronger than pure independent
reinvention: explicit mention without follow-through. After a full pass
across both census documents, exactly one edge in the entire 60-edge graph
qualifies — and its solitary status turns out to be more informative than
if it had ten siblings.

FreePDM's own thread ends with an unanswered question. In its
second-to-last post, someone asks grd "does you work base on ondsel or
another Opensource PDM?" — and grd's reply sidesteps it (`aware_of_uncoordinated`,
`project:freepdm` → `project:ondsel-server`, confirmed, source: §16.2 Phase
6, §16.4). FreePDM's community was, at the very end of a multi-year thread,
aware that Ondsel existed and never actually engaged with the comparison.
Meanwhile the graph shows the opposite direction running in the other
order entirely: `project:fep-0011`'s Motivation section explicitly names
`project:freepdm` as one of its "current known initiatives"
(`cites_as_prior_art`, confirmed) — FreeCAD's official effort did its
homework on the hobbyist one, but the hobbyist one only ever got as far as
wondering aloud about the official one, unanswered, at the very end of its
own story.

A second near-miss is worth naming precisely because the graph *declines*
to model it as an edge, and that refusal is itself a finding. The census
prose (§10.8 point 4) describes `project:odooplm` and `project:fep-0011` as
"completely unaware of each other" — a strong claim, and `project:odooplm`
is not a marginal project: 152 stars, the highest of any platform-level PLM
in the census, presented at Odoo Experience 2025, older and more active
than most FreeCAD-native PDM attempts. But on inspection this is *mutual
absence of citation*, not *documented awareness without coordination*,
which is what `aware_of_uncoordinated` actually requires as a type. No
comment, in either project's history, shows anyone on either side naming
the other. The taxonomy currently has no edge type for this — a confirmed
negative relationship, "provably isolated from," is a different kind of
claim than every other edge type in the graph, which all assert some
positive connection (however weak). Rather than force this into
`aware_of_uncoordinated` and quietly lower that type's bar, both this case
and a second one (the OSE cluster's total non-reference to Lens, FreePDM,
or FEP-0011) are left as documented non-edges — noted in
`ecosystem_graph.yaml`'s comments, not silently dropped, so a future reader
knows the case was considered and not the record's gap.

Read together, these two findings point at the same practical
recommendation: **cross-linking effort has asymmetric value here.**
FEP-0011 already does real work citing prior art (five `cites_as_prior_art`
edges point away from it in the graph — to `project:bcf-plugin-freecad`,
`project:freepdm`, `project:nanoplm`, `project:cadbaselibrary`,
`project:taack-plm`, plus `project:ondsel-lens-addon`). What's missing is
the return path: nothing in the census shows FreePDM, OdooPLM, or OSE's
library tooling engaging with FEP-0011 or Ondsel-Server#48 in return. A
single well-placed comment — someone from FEP-0011 answering FreePDM's
unanswered question, or someone from OdooPLM's 152-star, Odoo-Experience-
presented project weighing in on FEP-0011's discussion — would very likely
be worth more than another independent architecture proposal, given how
many of those the graph shows this ecosystem is already capable of
producing on its own.

## What the reasons behind rejection reveal

The plan's original design made `reason` a first-class field on
`rejected_in_favor_of` rather than a note, on the theory that "why" is more
reusable than "what." That bet paid off clearly in FreePDM's own history.
Three `rejected_in_favor_of` edges originate from `project:freepdm`, and
all three, once the Stage 2 `chose_instead` field existed to say so
honestly, turn out to be "rejected X, built something bespoke" rather than
"rejected X, adopted Y." grd examined SnowFS — a purpose-built VCS for
large binary/3D files — and concluded its git-like internals wouldn't suit
FCStd's zip-based format; that's a specific, transferable technical fact
any future FreeCAD PDM/VCS effort evaluating a git-based large-binary-file
approach should know before repeating the investigation. grd also rejected
OpenPLM's complexity outright ("I want to make a 'poor man's' PDM... I
don't care about fancy stuff"), and, later, abandoned his own SVN/git
framing after `person:zolko` directly challenged the whole effort as wheel
reinvention ("Why don't you choose an existing one") — grd's actual answer
was that he needed to control filesystem semantics himself, which is a
real design position, not a dismissal. Modeled as three edges each
carrying its own `reason` and `chose_instead`, this reads as a single
project's evolving, defensible design philosophy rather than as noise; read
only as "FreePDM ≠ SnowFS, FreePDM ≠ OpenPLM," it would read as nothing at
all.

## Where the taxonomy still shows seams

Stages 1-3 already documented the two changes made at the Stage 2 freeze
(`contributes_to`, and `chose_instead` on `rejected_in_favor_of`) and the
soft observations made after full-scale use — see
`ecosystem_relationship_mapping_results.md` for the complete taxonomy-fit
review. The one gap worth restating here, because it bears directly on
this document's own claims, is the missing "provably isolated from" edge
type discussed above. It's the only asymmetry in the taxonomy: every
existing edge type asserts a connection, of varying strength and
confidence, and none can assert a confirmed *absence* of one. That's a
one-sided taxonomy for a research project whose central finding is about
absence of connection. Worth a deliberate Stage 2.5 design pass if this
project continues — not a same-day patch, since a negative-relationship
edge type changes what kind of claim the graph is allowed to make, not
just what vocabulary it uses.

## What this document is not

This is a synthesis of a 57-project, 95-node graph built by rereading two
existing prose documents — not new research, and not a complete picture of
FreeCAD PDM/PLM activity beyond what those two documents already covered.
Where a project got stub-level treatment in the graph (an `unknown`
`technical_approach`, no `first_seen` date), that reflects the source
prose's own depth of coverage, not a claim that the project itself is
thin. See `ecosystem_graph.yaml` directly for the full facet and edge data
behind every claim above, and `ecosystem_visualizations.md` for the
diagrams referenced throughout.
