# FreeCAD Core & Lens Collaboration Issues — Raw Source Data

Purpose: complete, verbatim local copy of every FreeCAD-org and Ondsel-Server
GitHub issue, PR, FEP (FreeCAD Enhancement Proposal), and developer-meeting
mention specifically about **FreeCAD's own official collaboration/PDM design
effort** (the #25681 issue cluster, FEP-0011, and the people driving them), so
this research doesn't require repeated live fetches. All content below is
copied directly from source (GitHub REST API `api.github.com/repos/...`, raw
file content, or a web fetch/search where noted), not paraphrased or
summarized unless explicitly marked as such. Quoted blocks are the exact
`body` field of each issue/comment/PR/discussion post.

**This file covers only FreeCAD's/Lens's own official-channel design
conversation.** For the much broader census of every independent third-party
project (PDM/PLM tools, version-control addons, academic connectors,
international efforts, etc.) that also touches this space, see
[`freecad_pdm_plm_ecosystem_census.md`](freecad_pdm_plm_ecosystem_census.md).
An index tying all of this project's FreeCAD-collaboration research together
is at [`README.md`](README.md).

File renamed multiple times as scope evolved: `freecad-issue-25681.md` →
`freecad-pdm-issues.md` → `freecad-pdm-research.md` → (briefly)
`freecad_collaboration_ecosystem_research.md` → **split** into this file (the
official-channel issue/FEP/GSoC deep dive) plus
`freecad_pdm_plm_ecosystem_census.md` (the independent-project census, which
had grown too large and too topically distinct to stay in one file) once the
broader international/OpenCascade-wide search made clear the two needed to be
separate documents. All project doc filenames were standardized to lowercase
underscore-separated at the same time as the first rename in this chain.

Fetched: 2026-09-13.

## Table of contents

0. [Prehistory: a 2009 proto-PDM ticket from FreeCAD's own co-creator](#0-prehistory-a-2009-proto-pdm-ticket-from-freecads-own-co-creator) — 16 years before #25681
1. The #25681 issue cluster (FreeCAD/FreeCAD#25681, #25682, #25685, #23248, Ondsel-Server#47, #48)
2. [The old GSoC annotation-collaboration project](#7-the-old-gsoc-annotation-collaboration-project--identified) — identified: Patrick Podest's BCF-Plugin-FreeCAD, GSoC 2019
3. [The FreeCAD Enhancement Proposal (FEP) process and FEP-0011](#8-the-freecad-enhancement-proposal-fep-process) — the actual PDM/collaboration design document pieterhijma referenced
4. [Related work by the same author: FEP-0013 (Visual Diffs) and PR #28312 (versioning file format)](#9-related-work-versioning-and-visual-diffs)
10. [The "About PDM for FreeCAD" forum thread and the empty official roadmap placeholder](#10-the-about-pdm-for-freecad-forum-thread-and-the-empty-official-roadmap-placeholder) — a large (150+ post) community thread that predates and outlasts FEP-0011, never fully read in this research

## Cluster map

```
FreeCAD/FreeCAD#25681  "Core: Collaboration features"  (root/overarching issue)
├── FreeCAD/FreeCAD#25682  "Core: Improve annotations as a collaboration feature"  (sub-issue)
│     └── discussion forked into → FreeCAD/Ondsel-Server#48
├── FreeCAD/FreeCAD#25685  "Core: Add conversations to annotations"  (sub-issue)
│     └── references BCF standard/plugin (podestplatz/BCF-Plugin-FreeCAD)
└── references → FreeCAD/FreeCAD#23248  "Save LastModifiedDate in UTC is inconsistent"  (tangential, timestamp/collab-adjacent)

FreeCAD/Ondsel-Server#48  "Extent to which Lens is a PDM"  (spawned from #25682 discussion)
FreeCAD/Ondsel-Server#47  "Feature: Synchronize between Servers / Sync tool Library"  (related, same repo, cited in possible_freecad_collaboration.md, not directly linked from #25681 thread but same problem space)
```

No further FreeCAD-org issue links were found inside any of the bodies/comments
below beyond what's captured here — this is believed to be the full reachable
set from #25681 as of the fetch date. The one dangling external thread not
pulled in full: `podestplatz/BCF-Plugin-FreeCAD` pull request #10 (a
third-party plugin repo, not FreeCAD org) — noted at the bottom under "Not yet
fetched."

---

# 0. Prehistory: a 2009 proto-PDM ticket from FreeCAD's own co-creator

Found during a later international-language research pass (a German-language
search incidentally surfaced this English-language artifact) and confirmed
directly via the GitHub API. This predates the #25681 cluster by **sixteen
years** and appears to be the earliest on-record instance of anyone
articulating PDM-shaped thinking for FreeCAD.

## FreeCAD/FreeCAD#5539 — "Resource framework for FreeCAD"

https://github.com/FreeCAD/FreeCAD/issues/5539 (imported from the original
Mantis bug tracker, https://tracker.freecad.org/view.php?id=19 — ticket **#19**,
i.e. one of the first ~20 tickets ever filed against the project)

- **Reporter:** Jriegel — **Jürgen Riegel**, one of FreeCAD's original
  co-creators (alongside Werner Mayer and Yorik van Havre)
- **Date submitted:** 2009-09-28 (per the imported Mantis ticket metadata,
  "9/28/2009")
- **Status:** acknowledged, later closed (2023-10-23, by WandererFan, "Closed
  as replaced by Component Library")
- **Labels:** `Mod: Core`, `Type: Feature`

### Full original report text (verbatim)

> Often in 3D modeling we use Objects and files from 3rd party resources.
> This things are just used and rarely changed. like e.g. standard parts
> (ISO), complete products from a catalog and so on.
> We need in FreeCAD a framework to access/share/distribute such resources.
> It shut be possible to locate such resources locally or in the web and
> allow FreeCAD to find and access such resources referenced in a modeling
> document.
>
> Specification:
> * can be used loacally or in the web
> * unique id to find resources
> * central repository to collect resources
> * revision control
> * accessibly through http: and https:
>
> Design so far:
> * use SVN to host resources for revision control and restricted access
> * use webserver to host resources without revision control
> * use a XML description file to specify a resource, this file holds:
> - a UUID as global unique identifier
> - a name and description of the resource
> - a specifier it its a SVN or a PLAIN resource
> - addition meta infromation
> * use WGET or SVN to make the resources lokaly available
> * deliver a set of Resource description files with the installer/packages
> * Host on the FreeCAD website a repository of Resource description where
> FreeCAD can search for unknown UUIDs
>
> So far...
>
> Jürgen

### Comments (chronological, imported from Mantis + native GitHub)

**Kunda1 — 2018-06-13:** "Unassigning Jriegel"

**chennes — 2021-12-17:**
> It's not exactly what was envisioned here, but we do have the FreeCAD parts
> library now. It needs some work, but it may be worth closing this bug and
> re-opening more specific changes that need to be made to the parts library.
> This request doesn't really have any action items at this point.

**WandererFan — 2023-10-23:** "Closed as replaced by Component Library."

### Why this matters

Riegel's 2009 sketch already contains, in miniature, several ideas the
#25681/FEP-0011/Ondsel-Server#48 conversation re-derives independently 16
years later: a **UUID as global unique identifier** (→ Stage 0's "item
identity that outlives a file" problem in `ondsel_project_proposal_research.md`),
a **central repository with revision control** (→ the entire Lens/PDM
question), and **HTTP(S)-accessible, locatable-locally-or-on-the-web**
resources (→ the federation goal in Ondsel-Server#47). The eventual
"resolution" — closed in favor of the **FreeCAD Parts Library**
(`parts_library` addon, `github.com/FreeCAD/FreeCAD-library`) — is a real but
much narrower answer: a shared macro/part-file repository, with no UUID
scheme, no revision control beyond whatever git provides, and no resource
discovery framework. The original ambition was quietly scoped down to "a
folder of files," and the broader vision sat dormant for over a decade until
FEP-0011 and Ondsel-Server#48 re-opened essentially the same questions from
scratch, apparently without anyone in either 2025–2026 conversation citing
ticket #19.

## The chain continues: three more attempts across the following decade, each with a founder in the room

Riegel's 2009 ticket was not a one-off. Across the following fifteen years,
the FreeCAD forum saw at least three more rounds of "let's get PDM/PLM
working with FreeCAD," each time with one or more of the project's own
co-creators (Werner Mayer, Yorik van Havre, Jürgen Riegel) directly
participating — meaning the people best positioned to connect these dots to
Riegel's original ticket were present each time, and evidently didn't.

**2010 — openPLM's first FreeCAD plugin announcement.** A developer from the
openPLM project (self-identified only as "Philippe," `ninoofr@laposte.net`)
posted asking for a sample FreeCAD part to use in a demo video for "our
FreeCAD plugin" — the first-ever FreeCAD/PDM integration to actually ship
working code, years before anything else in this research. **wmayer**
responded within the hour with a sample file from FreeCAD's own SVN example
repository; **yorik** replied "Wow, fantastic work!!!"; **jriegel** replied
"Looks really great! Unfortunately I speak no French, so I have to wait to
dig deeper in that!" (openPLM's team and documentation were French). Docs and
a YouTube demo were linked in the same thread.

**2014 — a second, independent call for "Project Management/Collaboration
for FreeCAD."** A different community member (eukreign) proposed building
"a web based collaboration platform around FreeCAD" with a simple
check-out/check-in model and screen-sharing for camera-position sync during
review sessions. **jriegel** replied with pointers to two FreeCAD wiki pages
on the same topic (`Resource_framework_project` — almost certainly the same
page spawned by his own 2009 ticket — and a second, "...ly_project" page).
Another participant (jmaustpc) pointed at openPLM again. The thread produced
no shipped code and no further activity was found.

**2017 — a third call, explicitly citing docdokuplm and asking about NoSQL
architecture.** A community member (Malkov) proposed "a free enterprise
level PLM/PDM system integrated with FreeCAD," citing prior professional
experience with Siemens Teamcenter, and asked about NoSQL-based
architectures — the same database-technology debate the 2022 FreePDM thread
(`freecad_pdm_plm_ecosystem_census.md` §16) would re-run from scratch five
years later. **wandererfan** (a FreeCAD core contributor) again pointed at
the same "Resource_framework_project" wiki page. Another reply named
**docdokuplm** directly as existing prior art (the same project flagged as
an unresearched lead in `freecad_pdm_plm_ecosystem_census.md` §10.5) and
raised the idea of embedding a 3D viewer via an HTML5 frame. Crucially,
**NormandC** (another long-time contributor) confirmed in the same thread
that openPLM's FreeCAD plugin had gone four years without a new release and
**only ever supported FreeCAD 0.10/0.11** — i.e., by 2017, openPLM's 2010
integration was already obsolete and had never been updated to track
FreeCAD's own evolution.

**Pattern across all four (2009, 2010, 2014, 2017):** every single attempt
either stalled with no shipped code, or shipped code that was never
maintained past its initial release. Three different FreeCAD co-creators
show up across these threads, each time encouraging or contributing to the
effort in the moment, but none of them appear to have connected the threads
to each other after the fact — the 2014 and 2017 threads each independently
rediscover the existence of the same 2009 wiki page rather than anyone
maintaining a running list. This is the same discovery/coordination failure
`freecad_pdm_plm_ecosystem_census.md` §10.9 and §15 document at the
ecosystem-wide scale, now confirmed to be a **pattern inside FreeCAD's own
forum, spanning its own founders, for fifteen years before FEP-0011 existed.**

---

# 10. The "About PDM for FreeCAD" forum thread and the empty official roadmap placeholder

## The official placeholder

**`FreeCAD/FreeCAD-documentation` wiki page `PDM_Roadmap.md`**
(https://github.com/FreeCAD/FreeCAD-documentation/blob/main/wiki/PDM_Roadmap.md)
is an official FreeCAD documentation page whose **entire content** is a
placeholder: "Placeholder for FreeCAD PDM/PLM," linking out to the forum
thread below. This is itself a small, telling data point: FreeCAD's own
documentation acknowledges "PDM/PLM" as a topic that deserves a page, and
has left that page empty, pointing at a forum thread instead of anything
official, for as long as this placeholder has existed.

## The thread itself

**Forum thread "About PDM for FreeCAD"** —
`forum.freecad.org/viewtopic.php?f=8&t=68350` — turned out to be the
founding design-discussion and multi-year development journal of **FreePDM**
(`github.com/grd/FreePDM`, already catalogued in
`freecad_pdm_plm_ecosystem_census.md` §10.1): **156 posts, spanning
2022-04-28 to 2025-02-18** (almost three full years) — not a generic
community discussion, but *the* primary source for that specific project's
entire history, told largely in the founder's own words alongside a
recurring cast of FreeCAD community members, including production PDM
administrators sharing real operational experience.

A full synthesis, organized by arc/phase with the most load-bearing quotes
preserved, is in
[`freecad_pdm_plm_ecosystem_census.md`](freecad_pdm_plm_ecosystem_census.md)
**§16** (since it's fundamentally third-party-project history, not
FreeCAD's own official-channel effort, even though it surfaced via this
document's placeholder-roadmap lead). That section also captures several
more tools/projects named for the first time in this entire research
project (Perkeep/Camlistore, SnowFS, cad-plan.com's Cronos, pdm-studio.tech,
SparkleShare, and the "Tracking FreeCAD collaboration tools" forum thread
listing real open-hardware projects as potential PDM testcases), plus a
correction to this document's own "totally unaware of each other" framing:
FreePDM's author was directly asked, in the thread's second-to-last post,
*"does you work base on ondsel or another Opensource PDM?"* — meaning the two
communities are aware of each other **by name**, even though nothing found
in this research suggests they've ever coordinated.

If this single forum thread contains more raw community deliberation about
FreeCAD PDM requirements than every GitHub issue in section 1's cluster
combined, that in itself is a finding: none of it has been cross-referenced
by pieterhijma, pierreporte, or Creymore in any of the GitHub-side
conversation captured in this document.

---

# 1. FreeCAD/FreeCAD#25681 — "Core: Collaboration features"

https://github.com/FreeCAD/FreeCAD/issues/25681

- **State:** Open
- **Author:** [pieterhijma](https://github.com/pieterhijma) (CONTRIBUTOR)
- **Created:** 2025-11-26T09:28:56Z | **Updated:** 2026-03-16T14:35:03Z
- **Labels:** `GSoC`, `Mod: Core`, `Type: Feature`
- **Comments:** 6
- **Sub-issues:** 2 (#25682, #25685)

## Body

> ### Workbench affected?
>
> None
>
> ### Problem description
>
> Now that we have a [Lens server](https://github.com/FreeCAD/Ondsel-Server) and other initiatives for PLM/PDM systems, it would be good if FreeCAD would have collaboration features.
>
> I consider this an overarching issue discussing collaboration features that allow users to work in a team on models.
>
> ### GSOC project description
>
> This is an extensive, long-term subject that probably does not fit the scope of a GSoC project on its own. But it might anyway be interesting to develop a GSoC project that does part of the road to get there, for example:
>
> * Researching currently available PLM/PDM solutions in other software
> * Studying common behaviours  of those solutions
> * See how it compare to the Lens server
> * Define what would be missing features and how they could be implemented
> * Define a plan of an ideal PLM/PDM system for FreeCAD
>
> ### Project duration
>
> Long /  350h
>
> ### Project complexity
>
> Medium/Hard

## Comments (chronological)

### 1.1 [pieterhijma](https://github.com/pieterhijma) — 2025-12-14T12:04:04Z
https://github.com/FreeCAD/FreeCAD/issues/25681#issuecomment-3650793259

> https://github.com/FreeCAD/FreeCAD/issues/23248 is relevant for collaboration.

### 1.2 [Oval17](https://github.com/Oval17) — 2026-02-20T06:11:44Z
https://github.com/FreeCAD/FreeCAD/issues/25681#issuecomment-3931876386

> Hi @pieterhijma ,
> I am really interested in this research project, As I am going through docs and https://github.com/FreeCAD/Ondsel-Server
> Would really like to know a bit more expansion on the 5 points you mentioned in project description.
> cc : @yorikvanhavre
> Thanks

### 1.3 [pieterhijma](https://github.com/pieterhijma) — 2026-02-20T14:16:44Z
https://github.com/FreeCAD/FreeCAD/issues/25681#issuecomment-3935108235

> Hi @Oval17, @yorikvanhavre added the 5 points and made this a GSoC project. I'm already working on it and are in the process of defining a freecad enhancement proposal for it. In that sense, I'm not so sure if it's a very good target for a GSoC project.
>
> On the other hand, the 5 points are indeed relevant and it would be interesting to gain more knowledge about it. My recommendation for now is to browse the various issues that this issue is linking to. I can't find it currently, but I also have an issue on how Lens and PDM/PLM differ.

### 1.4 [Oval17](https://github.com/Oval17) — 2026-02-23T07:02:45Z
https://github.com/FreeCAD/FreeCAD/issues/25681#issuecomment-3943035934

> Ok got it , yes will see the points and start working if I can contribute to something.
> Thanks

### 1.5 [Creymore](https://github.com/Creymore) — 2026-03-15T14:02:59Z
https://github.com/FreeCAD/FreeCAD/issues/25681#issuecomment-4063054537

> >I can't find it currently, but I also have an issue on how Lens and PDM/PLM differ.
>
> This one ?
> https://github.com/FreeCAD/Ondsel-Server/issues/48

### 1.6 [pieterhijma](https://github.com/pieterhijma) — 2026-03-16T14:35:03Z
https://github.com/FreeCAD/FreeCAD/issues/25681#issuecomment-4068128879

> > > I can't find it currently, but I also have an issue on how Lens and PDM/PLM differ.
> >
> > This one ? [FreeCAD/Ondsel-Server#48](https://github.com/FreeCAD/Ondsel-Server/issues/48)
>
> Yes, that one, thanks!
>
> (this comment received 1 "laugh" reaction)

---

# 2. FreeCAD/FreeCAD#25682 — "Core: Improve annotations as a collaboration feature"

https://github.com/FreeCAD/FreeCAD/issues/25682

- **State:** Open
- **Author:** pieterhijma (CONTRIBUTOR)
- **Created:** 2025-11-26T09:34:40Z | **Updated:** 2026-01-26T14:32:47Z
- **Labels:** `Mod: Core`, `Type: Feature`, `Requires: UI/UX review`, `Requires: FEP`
- **Comments:** 7
- **Parent issue:** #25681

## Body

> ### Workbench affected?
>
> None
>
> ### Problem description
>
> I consider this part of #25681.
>
> This is a request for an improved feature that allows users to place annotations in specific parts of the 3D model to comment on the 3D model. This should be seen in the context of for example the [Lens server](https://github.com/FreeCAD/Ondsel-Server).
>
> ### Development version About Info (in Safe Mode)
>
> ```
> OS: Arch Linux (tty)
> Architecture: x86_64
> Version: 1.2.0dev.44440 (Git)
> Build date: 2025/11/25 10:47:05
> Build type: Unknown
> Branch: main
> Hash: 972ade948c41f789798bd49a9fa3945e7aa6e4a1
> Python 3.13.7, Qt 6.10.0, Coin 4.0.5, Vtk 9.5.2, boost 1_89, Eigen3 5.0.1, PySide 6.10.0
> shiboken 6.10.0, SMESH 7.7.1.0, xerces-c 3.3.0, OCC 7.9.2
> Locale: English/United States (en_US)
> Navigation Style/Orbit Style/Rotation Mode: CAD/Trackball/Drag at cursor
> Stylesheet/Theme/QtStyle: OpenDark.qss/OpenDark/
> Logical DPI/Physical DPI/Pixel Ratio: 96/92.6073/1.33333
> Installed mods:
>   * OpenTheme 2024.5.16
>   * fasteners 0.5.34
>   * WebTools 1.0.0
>   * osh-autodoc-workbench 0.2.3
>   * DynamicData 2.70.0
>   * Curves 0.6.61
>   * freecad.gears 1.3.0
>   * sheetmetal 0.7.24
>   * Ondsel-Lens 2025.11.21.01
> ```

## Comments (chronological)

### 2.1 [pieterhijma](https://github.com/pieterhijma) — 2025-11-28T09:56:08Z
https://github.com/FreeCAD/FreeCAD/issues/25682#issuecomment-3588622615

> FreeCAD currently has two document objects for annotations in the 3D view `App::Annotation` and `App::AnnotationLabel`. `App::Annotation` is not used, but it turns out that `App::AnnotationLabel` is accessible from the Tools menu and can be used to annotate 3D geometry. It is also used in BIM and Mesh:
>
> [screenshot image, not reproducible in text: `App::AnnotationLabel` accessible from the Tools menu]

### 2.2 [pieterhijma](https://github.com/pieterhijma) — 2025-11-28T10:48:01Z
https://github.com/FreeCAD/FreeCAD/issues/25682#issuecomment-3588811814

> This annotation feature is not well-known (At least, although I knew it was in the code, I didn't know it was accessible to users by means of a menu) and although the basic functionality is there, I think there are many drawbacks and the feature can be much improved.
>
> From the code, I get that the `App::AnnotationLabel` seems to be the preferred option, so for now, I'm not commenting on the `App::Annotation` (the yellow text appears at a specific location). In the sections below, I'm going to comment on the features.
>
> ### Placement
>
> What is good about the AnnotationLabel is that is possible to tie it to geometry. I selected the top face of the cube and then the annotation is "tied" to the cube. However, it does not store a reference to the geometry but absolute coordinates. This means that if I change the position of the cube, the label (the line) doesn't move with it. I think this is sometimes what you want, but in other cases, this is not what you want, so I think it would be good if you would have the option "link" it to geometry or use the absolute coordinates.
>
> The placement of the label is free, you can drag it all over the 3D view. Although this is nice, I think it would also be good to have it in a specific location, for example the top right. You can control the location of the base and the label separately.
>
> ### The contents or text
>
> The text and contents are very limited. The underlying data structure is a list of strings where each string is a line. The text is then transformed to an image, so, it is not possible to select the text to copy-paste it. I would prefer to allow more types of content, for example links to webpages, links to geometry in the model (for example if you want to point out a specific edge), links to versions, user names in the case of Ondsel Lens, etc.
>
> ### The visuals
>
> There are many things you can now control and changes, which is good:
> - font
> - color of font
> - size of font
> - color of background
> - whether you want a frame
> - whether you want a line or not
> - whether it should be visible
>
> ## Conclusion
>
> What I would like to see improved in the AnnotationLabel then is the following:
> - A flag that allows you to choose absolute or geometry-based placement
> - A way to store the location in case of geometry-based placement
> - A flag that controls how to drag the label
>   - in terms of 3D coordinates as is currently possible,
>   - at the top right
> - A way to place multiple comments in the top right.
>
> Since there is also #25685, this specific issue does not focus on the contents or text that I leave to that issue.

### 2.3 [pieterhijma](https://github.com/pieterhijma) — 2025-12-10T12:56:22Z
https://github.com/FreeCAD/FreeCAD/issues/25682#issuecomment-3636972291

> In #25685 I studied the BCF plugin and I identified as limitation of BCF that the comments are stored as separate files.
>
> I had a discussion with @Creymore and @pierreporte on the topic of annotations of collaborations where the central question was how annotations as collaboration feature as presented here and in #25685 affect the versioning of a file on platforms such as Lens.  In this post I will summarize the discussion where most of the information comes from @Creymore and @pierreporte.
>
> Firstly, @pierreporte makes a hard distinction between:
> - 3D annotations
> - collaboration annotations
>
> **3D annotations** are considered part of the 3D file and typically make it into drawings for example in the context of GD&T.  Having these kinds of annotations becomes more common in the industry.
>
> **Collaboration annotations** are used to discuss potential changes in the model.  Our general consensus is that they should not count as modifications to the model, for example because a discussion could lead to the conclusions to make no changes to the model.
>
> Having collaboration annotations in a separate file as in BCF is interesting, but may also be limiting in collaboration.
>
> Another option is a way to strip collaboration information from the file.
>
> The discussion continued regarding the question to what extent Lens is a PDM.  I will discuss this in https://github.com/FreeCAD/Ondsel-Server/issues/48.

### 2.4 [pieterhijma](https://github.com/pieterhijma) — 2025-12-11T12:22:15Z
https://github.com/FreeCAD/FreeCAD/issues/25682#issuecomment-3641672723

> I think for some users it would be good to have collaboration annotations as part of the file.  It could be the case that users actually want to track these comments, for example in the context of open source hardware that typically lacks documentation.
>
> In more professional settings, making use of PDMs, it may be good to separate the comments from the model to ensure that new comments do not affect the version of the model.
>
> The BCF plugin already has the separation of model and comments by means of a BCF file.  A drawback of this is that the comments can only be loaded in the BCF plugin and are only visible in a task panel.  I believe it would be nice to make collaboration annotations and comments visible in the document object tree as well, so to include it in the document tree to provide a more idiomatic user experience.
>
> Since FreeCAD has "transient" properties, I investigated if FreeCAD has transient document objects.  This is not the case and that makes sense because the whole point of document objects is that they are stored in the document.
>
> This would mean that annotation can only become part of the document and does not offer anything for the more professional settings.  However, there is quite a simple solution for this by means of App::Link and the notion of **meta-documents** that I described in https://github.com/FreeCAD/Ondsel-Server/issues/48#issue-3715105649.
>
> Suppose we have a file `model.FCStd` with a model, we have a PDM system, and we decide that we don't want comments on `model.FCStd` to affect the versioning of the item of which the model is part.  We can then make a file `model-topics.FCStd` with an `App::Link` to the model in `model.FCStd`.  We "check that in" into the PDM system and the PDM system should be able to mark this document as a meta-document linked to the original model.  The collaboration annotations can then be made in `model-topics.FCStd` simply as topics and comments as document objects in that file without affecting the versioning of `model.FCStd` or the item it belongs to.  I believe this is the most "FreeCAD way" to ensure that we can have both document objects as comments and separation of comments from models.  This requires a bit of logic from PDMs but this isn't a large problem, I believe.

### 2.5 [Reqrefusion](https://github.com/Reqrefusion) — 2025-12-19T21:11:28Z
https://github.com/FreeCAD/FreeCAD/issues/25682#issuecomment-3676637175

> There seems to be a lot of user experience here, maybe @FreeCAD/design-working-group has something to say for discussion.

### 2.6 [hkienle](https://github.com/hkienle) — 2026-01-05T16:14:15Z
https://github.com/FreeCAD/FreeCAD/issues/25682#issuecomment-3711115539

> > What I would like to see improved in the AnnotationLabel then is the following:
>
> I think it may be useful to have a way to make annotations collapsible/expandable.

### 2.7 [pieterhijma](https://github.com/pieterhijma) — 2026-01-26T14:32:47Z
https://github.com/FreeCAD/FreeCAD/issues/25682#issuecomment-3799907513

> > > What I would like to see improved in the AnnotationLabel then is the following:
> >
> > I think it may be useful to have a way to make annotations collapsible/expandable.
>
> I think the annotations themselves will only be the titles. I think the most FreeCAD way is to have the discussion in a task panel.
>
> (this comment received 1 "+1" reaction)

---

# 3. FreeCAD/FreeCAD#25685 — "Core: Add conversations to annotations (for collaboration purposes)"

https://github.com/FreeCAD/FreeCAD/issues/25685

- **State:** Open
- **Author:** pieterhijma (CONTRIBUTOR)
- **Created:** 2025-11-26T09:38:17Z | **Updated:** 2025-12-02T14:40:46Z
- **Labels:** `Mod: Core`, `Type: Feature`
- **Comments:** 9
- **Parent issue:** #25681
- **Related:** "an extension to" #25682

## Body

> ### Workbench affected?
>
> None
>
> ### Problem description
>
> I consider this part of
> * #25681
>
> and an extension to
> * #25682
>
> This is a request for a feature that allows users to have conversations in the annotations, meaning that an annotation has a list of comments by users with a date and time.
>
> ### Steps to reproduce
>
> n/a
>
> ### Expected behavior
>
> A button that allows you to add a comment in an annotation.
>
> ### Actual behavior
>
> n/a
>
> ### Development version About Info (in Safe Mode)
>
> ```
> OS: Arch Linux (tty)
> Architecture: x86_64
> Version: 1.2.0dev.44440 (Git)
> Build date: 2025/11/25 10:47:05
> Build type: Unknown
> Branch: main
> Hash: 972ade948c41f789798bd49a9fa3945e7aa6e4a1
> Python 3.13.7, Qt 6.10.0, Coin 4.0.5, Vtk 9.5.2, boost 1_89, Eigen3 5.0.1, PySide 6.10.0
> shiboken 6.10.0, SMESH 7.7.1.0, xerces-c 3.3.0, OCC 7.9.2
> Locale: English/United States (en_US)
> Navigation Style/Orbit Style/Rotation Mode: CAD/Trackball/Drag at cursor
> Stylesheet/Theme/QtStyle: OpenDark.qss/OpenDark/
> Logical DPI/Physical DPI/Pixel Ratio: 96/92.6073/1.33333
> Installed mods:
>   * OpenTheme 2024.5.16
>   * fasteners 0.5.34
>   * WebTools 1.0.0
>   * osh-autodoc-workbench 0.2.3
>   * DynamicData 2.70.0
>   * Curves 0.6.61
>   * freecad.gears 1.3.0
>   * sheetmetal 0.7.24
>   * Ondsel-Lens 2025.11.21.01
> ```
>
> ### Last known good version (optional)
>
> ```
>
> ```

## Comments (chronological)

### 3.1 [luzpaz](https://github.com/luzpaz) — 2025-11-26T12:38:35Z
https://github.com/FreeCAD/FreeCAD/issues/25685#issuecomment-3581142168

> Several years ago there was a GSoC (perhaps BIM workbench, actually Arch workbench at the time) that was tasked with a annotation collaboration effort. Does anyone remember ? I don't think it ever made it into core though. Maybe the code can be exhumed and re-used ?

### 3.2 [pieterhijma](https://github.com/pieterhijma) — 2025-11-28T10:52:54Z
https://github.com/FreeCAD/FreeCAD/issues/25685#issuecomment-3588830912

> @luzpaz, Great idea. However, I searched for it and couldn't find it.

### 3.3 [paullee0](https://github.com/paullee0) — 2025-11-30T07:01:47Z
https://github.com/FreeCAD/FreeCAD/issues/25685#issuecomment-3592290641

> Is it part of BCF objective?
>
> (this comment received 1 "+1" reaction)

### 3.4 [paullee0](https://github.com/paullee0) — 2025-11-30T07:03:45Z
https://github.com/FreeCAD/FreeCAD/issues/25685#issuecomment-3592291968

> Should merge this with https://github.com/FreeCAD/FreeCAD/issues/25682 ?

### 3.5 [pieterhijma](https://github.com/pieterhijma) — 2025-11-30T11:27:38Z
https://github.com/FreeCAD/FreeCAD/issues/25685#issuecomment-3592472996

> > Is it part of BCF objective?
>
> It is not, but thanks for the hint. I'm now aware of the GSoC project. The entry point of the documentation is [this wiki page](https://wiki.freecad.org/BCF_support).
>
> (this comment received 1 "+1" reaction)

### 3.6 [pieterhijma](https://github.com/pieterhijma) — 2025-11-30T11:32:24Z
https://github.com/FreeCAD/FreeCAD/issues/25685#issuecomment-3592475658

> > Should merge this with [#25682](https://github.com/FreeCAD/FreeCAD/issues/25682) ?
>
> No, I don't think so. I think these are separate concerns.

### 3.7 [luzpaz](https://github.com/luzpaz) — 2025-11-30T12:41:47Z
https://github.com/FreeCAD/FreeCAD/issues/25685#issuecomment-3592518492

> > Is it part of BCF objective?
>
> @paullee0 Yes! Thanks.
>
> @pieterhijma there was some good headway made. I remember seeing screenshots on the forum.

### 3.8 [pieterhijma](https://github.com/pieterhijma) — 2025-11-30T13:49:10Z
https://github.com/FreeCAD/FreeCAD/issues/25685#issuecomment-3592565641

> > @pieterhijma there was some good headway made. I remember seeing screenshots on the forum.
>
> I'm already checking it. Very interesting.
>
> (this comment received 1 "+1" reaction)

### 3.9 [pieterhijma](https://github.com/pieterhijma) — 2025-12-02T14:40:46Z
https://github.com/FreeCAD/FreeCAD/issues/25685#issuecomment-3602393261

> I've taken a good look at the BCF plugin and updated it where I could to FreeCAD 1.0: https://github.com/podestplatz/BCF-Plugin-FreeCAD/pull/10.
>
> It is great that this standard is here and that there is an implementation available as well!  A good entry point for the implementation is [this wiki page](https://wiki.freecad.org/BCF_support).  A good entry point to the standard is [this documentation page](https://github.com/buildingSMART/BCF-XML/tree/release_3_0/Documentation).
>
> So, the standard is the BIM Collaboration Format (or BCF) from the same organization behind the IFC file format.  What the BCF plugin and the standard offer comes close to what I would like to see.  It supports conversations (they call it topics), essentially a list of comments, it supports dates, email addresses as user names, screenshots (they call it snapshots), and viewpoints.
>
> Interestingly, there is only little information that is very BIM-specific, such as `IfcProject`.  Most of the other information is useful for generic CAD software as well.
>
> A large drawback of the BCF standard and the plugin is that the information is stored in separate .bcf files.  I would prefer to make the comments part of a FreeCAD file, in the same way that comments are part of text documents.
>
> A drawback of having a separate BCF plugin is that the code as such cannot be reused for -- for example -- collaboration on the Lens platform or for other PDM-like systems.
>
> Given these drawbacks and since there is a clear standard for collaboration available, I would propose to make a generic, extensible framework for collaboration features in FreeCAD that is compatible with the BCF standard.
>
> In such a scenario, with a generic collaboration framework, the Lens addon would extend the framework to allow for referencing Lens users and referencing specific versions stored on the Lens platform.
>
> The BCF plugin could extend the framework that FreeCAD provides to create topics and comments, extended with for example email addresses (instead of usernames) that BCF requires.  The collaboration info would be stored in a FreeCAD file, or if FreeCAD works natively in IFC files, in BCF files.
>
> (this comment received 4 "+1" reactions)

---

# 4. FreeCAD/FreeCAD#23248 — "Core: Save `LastModifiedDate` (and other dates) in UTC is inconsistent"

https://github.com/FreeCAD/FreeCAD/issues/23248

- **State:** Open
- **Author:** [recursivenomad](https://github.com/recursivenomad) (NONE)
- **Created:** 2025-08-20T13:43:23Z | **Updated:** 2026-03-06T11:18:04Z
- **Labels:** `Mod: Core`, `Type: Feature`, `Status: Needs feedback`, `Status: Confirmed`, `Priority: High`
- **Comments:** 3
- **Reactions:** 3 "+1"
- **Linked from:** #25681 comment 1.1 ("relevant for collaboration")

## Body

> ### Is there an existing issue for this?
>
> - [x] I have searched the existing issues
>
> ### Problem description
>
> When saving a FreeCAD document, it saves `LastModifiedDate` (and other dates, like `CreationDate`) in the local timezone time.  It would be beneficial to (at least have the ability to) instead save it in UTC for 2 reasons:
>
> 1) Avoids time-travel when collaborating across timezones
> 2) Prevents the revealing of location information of the author when paired with other timestamped metadata (ie. a git commit)
>
> To reproduce:
>
> - Create a new FreeCAD document
> - Save it
> - Extract the .FCStd file archive
> - Open `Document.xml`
> - Search for "LastModifiedDate"
> - Observe the date being presented in your local timezone
>
> ### Full version info
>
> ```
> OS: Windows 10
> Architecture: x86_64
> Version: 1.0.2.39319 (Git) Conda
> Build type: Release
> Branch: (HEAD detached at 1.0.2)
> Hash: 256fc7eff3379911ab5daf88e10182c509aa8052
> Python 3.11.13, Qt 5.15.15, Coin 4.0.3, Vtk 9.3.0, OCC 7.8.1
> Locale: English/United Kingdom (en_GB)
> Stylesheet/Theme/QtStyle: FreeCAD Dark.qss/FreeCAD Dark/Fusion
> ```
>
> ### Subproject(s) affected?
>
> None
>
> ### Anything else?
>
> _No response_
>
> ### Code of Conduct
>
> - [x] I agree to follow this project's Code of Conduct

## Comments (chronological)

### 4.1 [luzpaz](https://github.com/luzpaz) — 2025-08-23T11:49:39Z
https://github.com/FreeCAD/FreeCAD/issues/23248#issuecomment-3216786716

> cc @pieterhijma this may be useful for collaborative projects in LENS perhaps ?
>
> (this comment received 1 "+1" reaction)

### 4.2 [luzpaz](https://github.com/luzpaz) — 2025-12-12T10:48:41Z
https://github.com/FreeCAD/FreeCAD/issues/23248#issuecomment-3645964455

> @FreeCAD/design-working-group please consider discussing this. It makes sense. And if implemented would also need logic to convert UTC to the local TZ of the user's system for convenience.

### 4.3 [pieterhijma](https://github.com/pieterhijma) — 2026-03-06T10:17:21Z
https://github.com/FreeCAD/FreeCAD/issues/23248#issuecomment-4010849523

> Interesting, I can't confirm on Linux FreeCAD 1.0.2 (so UTC):
>
> On Linux 1.0.2:
>
> ```
> OS: Arch Linux (xcb)
> Architecture: x86_64
> Version: 1.0.2.39319 (Git)
> Build type: Release
> Branch: makepkg
> Hash: 256fc7eff3379911ab5daf88e10182c509aa8052
> Python 3.14.2, Qt 6.10.1, Coin 4.0.5, Vtk 9.5.2, OCC 7.9.3
> Locale: English/United States (en_US)
> Stylesheet/Theme/QtStyle: OpenDark.qss/OpenDark/
> Installed mods:
>   * OpenTheme 2024.5.16
>   * fasteners 0.5.34
>   * WebTools 1.0.0
>   * osh-autodoc-workbench 0.2.3
>   * DynamicData 2.70.0
>   * Curves 0.6.61
>   * freecad.gears 1.3.0
>   * sheetmetal 0.7.24
>   * AddonManager 2025.11.4
>   * manifest.json
>   * Assembly4 0.60.3
>   * BCFPlugin 1.0.0
>   * Ondsel-Lens 2025.11.21.01
> ```
>
> ```xml
>         <Property name="CreationDate" type="App::PropertyString" status="16777217">
>             <String value="2026-03-06T08:58:38Z"/>
>         </Property>
> ```
>
> On Linux dev, however, I can confirm (local timezone):
>
> ```
> OS: Arch Linux (xcb)
> Architecture: x86_64
> Version: 1.0.2.39319 (Git)
> Build type: Release
> Branch: makepkg
> Hash: 256fc7eff3379911ab5daf88e10182c509aa8052
> Python 3.14.2, Qt 6.10.1, Coin 4.0.5, Vtk 9.5.2, OCC 7.9.3
> Locale: English/United States (en_US)
> Stylesheet/Theme/QtStyle: OpenDark.qss/OpenDark/
> Installed mods:
>   * OpenTheme 2024.5.16
>   * fasteners 0.5.34
>   * WebTools 1.0.0
>   * osh-autodoc-workbench 0.2.3
>   * DynamicData 2.70.0
>   * Curves 0.6.61
>   * freecad.gears 1.3.0
>   * sheetmetal 0.7.24
>   * AddonManager 2025.11.4
>   * manifest.json
>   * Assembly4 0.60.3
>   * BCFPlugin 1.0.0
>   * Ondsel-Lens 2025.11.21.01
> ```
>
> ```xml
>         <Property name="CreationDate" type="App::PropertyString" status="16777217">
>             <String value="2026-03-06T10:32:33+01:00"/>
>         </Property>
> ```
>
> Weirdly enough, On Windows with FreeCAD 1.0.2 with same version as OP, I can't confirm (so UTC):
>
> ```
> OS: Windows 11 build 26100
> Architecture: x86_64
> Version: 1.0.2.39319 (Git) Conda
> Build type: Release
> Branch: (HEAD detached at 1.0.2)
> Hash: 256fc7eff3379911ab5daf88e10182c509aa8052
> Python 3.11.13, Qt 5.15.15, Coin 4.0.3, Vtk 9.3.0, OCC 7.8.1
> Locale: English/United Kingdom (en_GB)
> Stylesheet/Theme/QtStyle: FreeCAD Light.qss/FreeCAD Light/Fusion
> Installed mods:
>
> * AddonManager 2025.11.4
> * manifest.json
> * Ondsel-Lens 2024.11.29.01
> ```
>
> ```xml
>         <Property name="CreationDate" type="App::PropertyString" status="16777217">
>             <String value="2026-03-06T09:13:35Z"/>
>         </Property>
> ```
>
> On Mac OS, FreeCAD 1.1 dev (not very recent) UTC:
>
> ```
> OS: macOS 14.5
> Architecture: arm64
> Version: 1.1.0dev.40253 (Git) Conda
> Build type: Release
> Branch: main
> Hash: e16c462916a050909ad107e4f1ac2e0e5990389d
> Python 3.11.11, Qt 5.15.15, Coin 4.0.3, Vtk 9.3.0, IfcOpenShell 0.0.0, OCC 7.8.1
> Locale: C/Default (C)
> Stylesheet/Theme/QtStyle: unset/unset/Fusion
> Logical/physical DPI: 72/128.5
> Installed mods:
>
> * Ondsel-Lens 2024.7.5.02
> ```
>
> ```xml
>         <Property name="CreationDate" type="App::PropertyString" status="16777217">
>             <String value="2026-03-06T09:39:32Z"/>
>         </Property>
>
> ```
>
> On Mac OS, FreeCAD 1.0.2 UTC:
>
> ```
>         <Property name="CreationDate" type="App::PropertyString" status="16777217">
>             <String value="2026-03-06T10:03:28Z"/>
>         </Property>
>
> ```
>
> ```xml
> OS: macOS 14.5
> Architecture: arm64
> Version: 1.0.2.39319 (Git) Conda
> Build type: Release
> Branch: (HEAD detached at 1.0.2)
> Hash: 256fc7eff3379911ab5daf88e10182c509aa8052
> Python 3.11.13, Qt 5.15.15, Coin 4.0.3, Vtk 9.3.0, OCC 7.8.1
> Locale: C/Default (C)
> Stylesheet/Theme/QtStyle: unset/unset/Fusion
> Installed mods:
>
> * Ondsel-Lens 2024.7.5.02
> ```
>
> The code depends on a Qt library. It has changed recently by @chennes, but that doesn't explain the difference (in https://github.com/FreeCAD/FreeCAD/commit/6974c83f9a7a30c13d750543797a2e7520d251d0) between my Windows and the OP's Windows version. Probably my Windows system's system clock is on UTC (in BIOS) (corrected by Windows with timezone), whereas the OP's Windows' system time may be the local time.
>
> Well, actually, that doesn't explain why my Linux dev version shows the time with timezones, whereas 1.0.2 (with the same Qt version) shows the local time.
>
> In any case, there are two things we can conclude:
> - Since timezone information is stored, the times in the file are correct.
> - The way we store the time is inconsistent.
>
> Should we have a preference on how timezone info is stored?

---

# 5. FreeCAD/Ondsel-Server#48 — "Extent to which Lens is a PDM"

https://github.com/FreeCAD/Ondsel-Server/issues/48

- **State:** Open
- **Author:** pieterhijma (COLLABORATOR)
- **Created:** 2025-12-10T12:58:59Z | **Updated:** 2026-09-13T14:11:29Z
- **Labels:** none
- **Comments:** 1
- **Spawned from:** #25682 comment 2.3 ("The discussion continued regarding the question to what extent Lens is a PDM. I will discuss this in [this issue].")

## Body

> This issue is based on a discussion with @pierreporte and @Creymore regarding collaboration annotations.  The collaboration annotation discussion is summarized in https://github.com/FreeCAD/FreeCAD/issues/25682 and revolved around whether collaboration annotations should be versioned or not.  This led to an explanation how this relates to PDM systems.
>
> @pierreporte describes the PDM workflow as having two parallel structures:
> - models that have links between them (an assembly has a link to a part or other subassemblies)
> - items that represent parts or assemblies and are described by documents (essentially files) where 3D models can be documents, a drawing can be a document, specifications can be documents
>
> Lens currently only manages (versions of) files but items are critical for a PDM.  To make sure we all talk about the same things, we defined the terms that are in use in this context:
>
> - **model**: a design that defines geometry in 3D that can be realized in real life
> - **link**: a reference to (the geometry of) a model.  In FreeCAD a link is much more versatile but I think the definition is correct for this context.
> - **part**: a single well-defined component that cannot be broken down
> - **assembly**: a collection of parts or assemblies joined together to form one piece
> - **sub assembly**: an assembly that is a member of another assembly
> - **item**: a representation of a part or assembly in terms of documents
> - **document**: a single piece of documentation for an item that can take various forms such as models, drawings, specifications.  It consists of the primary content and possibly attachments.
> - **drawing**: A 2D representation of a 3D model, often used for specifications for manufacturing
> - **specification**: requirements for realization of the model
> - **file**: in this context a document is a file
> - **version**: a specific state of a document
> - **container**: a folder in which items or documents are stored
> - **primary content**: the main file that defines the document, for example a PDF.
> - **attachment**: supporting files that contribute to the primary contents, for example a word document that results in the PDF.
>
> A PDM (Product Data Management) system is mainly a collaboration tool that provides a central place for CAD models and related information to ensure that all collaborators have the last version.  It is typically a central server on premise and limited to one organization.
>
> For PDMs, item management is very important.  Since an item consists of documents, changing a document, changes its version which changes the item version.  The goal of a PDM is tracking these kinds of changes which is sometimes a legal requirement, for example in aerospace through EN 9100 compliance.  It is not required to track intermediate changes, so you have control over when to release a new version.
>
> Since PDMs are collaboration tools, they often offer ways to discuss items.  These discussions do not increase version numbers and are often not a document.  Most likely, PDM systems do not store them in a file, but simply on the platform.  Since FreeCAD/Lens wants to share this kind of information, it makes sense to include these kinds of discussions in a file in some form, possibly in the FreeCAD file itself or as a separate file similar to BCF.
>
> This led us to a new term:
>
> - **meta-document**: a document that does not contribute to the specification of an item but allows discussion of the item.
>
> At this stage, we can conclude that Lens currently does not support the notion of an item.  One possibility to have that is storing an item as a sub-directory in a workspace or the workspace as well and attach version information to it on the server.
>
> In addition, Lens has goals that are not typical for PDMs, namely the goal to make FreeCAD designs available to FreeCAD users and even to users that don't use FreeCAD.  The idea is that Lens can host designs that users can configure/parameterize themselves and download a FreeCAD, STL, or STEP file for further processing.
>
> Parameterization of parts is something that PDMs typically don't provide since one item is only for one object and the unique ID that identifies an object cannot be parameterized as that item should have a different ID then.  Typically in PDMs variants of parts have their own model, document, and drawing.
>
> This led to the insight that items in traditional PDMs are very static and that for compliance and provenance reasons, they take the pragmatic route to store each variant as a separate item.  However, this does not necessarily have to be the case if the ID can encode a specific set of parameters and the system ensures that the ID always results in the same item.  The documents could in principle be generated on the fly, although it may be difficult to provide guarantees for this kind of functionality.
>
> So, I think we have to come to the conclusion that PDMs and Lens share functionality, managing CAD files and versions, but Lens does currently not offer enough to call it a PDM system.  In addition, Lens has other goals beyond the goals of a PDM, for example sharing and configuring CAD files to FreeCAD users and even beyond FreeCAD users.

## Comments (chronological)

### 5.1 [rusttick](https://github.com/rusttick) — 2026-09-13T14:11:29Z **(this project's own account — already posted)**
https://github.com/FreeCAD/Ondsel-Server/issues/48#issuecomment-5653789263

> Hi! I stumbled onto this issue while doing some research into international standards and the state of PDM for FreeCAD. I hope that this AI slop may be helpful and relevant to what you are working on. If not, please disregard. Thanks!
>
> Your own item/document/version vocabulary already matches, almost term for term, the [Document](https://industrialdigitaltwin.org/wp-content/uploads/2025/07/IDTA-02004-2-0_Submodel_Handover-Documentation.pdf#page=3)/[DocumentVersion](https://industrialdigitaltwin.org/wp-content/uploads/2025/07/IDTA-02004-2-0_Submodel_Handover-Documentation.pdf#page=12) split formalized by [VDI 2770](https://industrialdigitaltwin.org/wp-content/uploads/2025/07/IDTA-02004-2-0_Submodel_Handover-Documentation.pdf#page=3) Blatt 1 and consumed by the Asset Administration Shell's Handover Documentation submodel ([IDTA 02004](https://industrialdigitaltwin.org/wp-content/uploads/2025/07/IDTA-02004-2-0_Submodel_Handover-Documentation.pdf)), which also already covers your "item = assembly described by documents" case via [Entities](https://industrialdigitaltwin.org/wp-content/uploads/2025/07/IDTA-02004-2-0_Submodel_Handover-Documentation.pdf#page=5) that reference a sibling part's own AAS by AssetId; your "model has links to models" structure is likewise a published template, [Hierarchical](https://industrialdigitaltwin.org/en/wp-content/uploads/sites/2/2024/06/IDTA-02011-1-1_Submodel_HierarchicalStructuresEnablingBoM.pdf) Structures enabling BoM; your invented "meta-document" for versionless discussion is exactly what buildingSMART's [BCF](https://technical.buildingsmart.org/standards/bcf/) format was built for (you already gestured at this yourselves); and the parameterization-vs-identity tension you flagged — "the unique ID... cannot be parameterized... unless the ID can encode a specific set of parameters" — is precisely what AAS's [type](https://industrialdigitaltwin.org/wp-content/uploads/2023/06/IDTA-01001-3-0_SpecificationAssetAdministrationShell_Part1_Metamodel.pdf#page=23)/[instance](https://industrialdigitaltwin.org/wp-content/uploads/2023/06/IDTA-01001-3-0_SpecificationAssetAdministrationShell_Part1_Metamodel.pdf#page=25) split resolves via the normative [derivedFrom](https://industrialdigitaltwin.org/wp-content/uploads/2023/06/IDTA-01001-3-0_SpecificationAssetAdministrationShell_Part1_Metamodel.pdf#page=25) relationship — a stable type asset holds the shared, unconfigurable design while each configured instance keeps its own stable ID and parameters and just points back, rather than one ID trying to encode both — all under the IEC 63278 standard maintained by [IDTA](https://industrialdigitaltwin.org/) with a free, [BaSyx](https://eclipse.dev/basyx/) reference implementation to build against, so none of this needs to be designed from scratch.

**Note:** this issue has received no reply from pieterhijma/pierreporte/Creymore yet as of the fetch date (2026-09-13); it is the live end of the thread.

---

# 6. FreeCAD/Ondsel-Server#47 — "Feature: Synronice between Servers / Sync tool Libary" [sic, title has typos in source]

https://github.com/FreeCAD/Ondsel-Server/issues/47

- **State:** Open
- **Author:** [Creymore](https://github.com/Creymore) (NONE)
- **Created:** 2025-12-05T21:31:59Z | **Updated:** 2026-03-10T20:40:03Z
- **Labels:** none
- **Comments:** 0

## Body

> The idea is to have the ability to synchronize between two servers.
>
> I had some thoughts of how it could work from a User perspective:
> Step 0:
> Have Account A on Server S1
> Have Account B on Server S2
> Be logged in on both
>
> Step 1:
> Open the synchronize Accounts feature on both accounts.
>
> Step 2:
> Copy the "synchronization link" from Account A on S1
>
> Step 3:
> Past the "synchronization link" into the Synchronize with filed of Account B on S2
>
> Step 4.0:
> Confirm the connection attamt from Account B Server S2 to Account A Server S1 on Account A Server S1
> (for extra safety)
>
> Step 4.1:
> Now the Login prompt opens for Account A through Server S2. Log in
>
> Step 5:
> Choose between synchronization options:
> - Couple (A => B and B => A)
> - One way (A => B or B =>)
> - One time
>
> Step 6: Confirm and be happy, you can Change it at any time from both accounts
>
> How would this be useful ?
> Migration of a Makerspace Hosted or FPA Server to a Personal or the other way around.
> Synchronization of Tool library like the nibbler bot, although this should work easier.
>
> Additional needed features:
> Organization Owners can disable this for any Data Associated with their Organization.
> A tool Libary can be shared without having two accounts, maybe a public sharing link. Or a "synchronization link" on the homepage of the lens server.
>
> Additional information:
> "synchronization link" = A string that tells the Server where to Synchronize to, maybe Server IP or website name and user name.

No comments on this issue as of the fetch date.

---

# 7. The old GSoC annotation-collaboration project — identified

In #25685 comments 3.1 and 3.7, [luzpaz](https://github.com/luzpaz) recalled "a
GSoC (perhaps BIM workbench, actually Arch workbench at the time) that was
tasked with a annotation collaboration effort" and remembered "screenshots on
the forum," but neither luzpaz nor pieterhijma (comment 3.2) could locate it
by searching FreeCAD/FreeCAD issues. It turned out not to be a FreeCAD/FreeCAD
issue at all — it's a separate, external plugin repo. Identified via web
search and cross-confirmed by an unrelated 2026-04-05 FreeCAD developer
meeting transcript (see below).

## Identity: Patrick Podest, GSoC 2019, "BCF Plugin for FreeCAD"

- **Student:** Patrick Podest ([podestplatz](https://github.com/podestplatz) on GitHub)
- **Project:** BCF (BIM Collaboration Format) support for FreeCAD
- **Program:** Google Summer of Code **2019**
- **Mentors:** yorikvanhavre ("yorik") and hardeeprai, both confirmed directly
  in the project's own forum thread ("BCF Support GSoC Proposal",
  `forum.freecad.org/viewtopic.php?t=35465`) — yorik mentors on documentation
  structure, UI/API design, and packaging throughout; hardeeprai mentors
  continuously from the acceptance announcement (2019-05-08) through the
  project's final wrap-up posts (2019-08-24), including workflow advice (the
  Pomodoro technique) and technical suggestions (auto-saving BCF state
  in-place rather than requiring an explicit save button).
- **Backstory:** per the same thread, Patrick originally wanted to do a
  FEM-related GSoC project, but that slot was already taken by another
  student, so the BCF implementation was suggested to him instead.
- **Proposal and timeline, from the primary source:** first complete draft
  posted 2019-04-06, with an application deadline of 2019-04-09 18:00 UTC.
  Acceptance announced by Patrick on 2019-05-08. The plugin was merged into
  FreeCAD's official Addon Manager on 2019-08-19 — yorik's own words in the
  thread: *"Your BCF plugin is now in the addons manager BTW! We can open the
  champagne."* hardeeprai's closing assessment: *"The project was handles
  [sic] in professional manner."*
- **Repository:** https://github.com/podestplatz/BCF-Plugin-FreeCAD
  - Created: 2019-05-11T12:06:56Z (consistent with a GSoC coding-period start)
  - Last pushed: 2024-02-09T21:19:40Z
  - 9 stars, LGPL-2.1
  - Now carries the banner: **"❗Not actively maintained❗ Maybe some of the [forks](https://github.com/podestplatz/BCF-Plugin-FreeCAD/network/members) are better maintained than this repo. I unfortunately don't have the time to support it."**
- **Development blog:** https://podestplatz.github.io/FreeCAD-blog/ — documents the full GSoC 2019 project including data model/file handling for BCF XML, a Qt-based GUI (topic selection, comments, snapshots, viewpoint activation), a programmatic (non-GUI) API, 3D integration (camera positioning, component highlighting, clipping-plane visualization), unit tests, and a v1.0 release in **August 2019**.

## Confirmation from the FreeCAD developer meeting (2026-04-05)

The FEP-0011 discussion (see section 8 below) directly ties this old project to
the *current* collaboration framework effort. From the FreeCAD developer
meeting minutes of 2026-04-05
(https://github.com/FreeCAD/FreeCAD-developer-meetings/blob/main/Minutes/minutes-2026-04-05.md),
under the heading "Collaboration Framework" (AI-generated, human-edited
minutes, quoted verbatim):

> Pieter provides an update on the generic collaboration framework, emphasizing its standalone nature and seeking feedback on the draft proposal. He is currently reviewing comments on clipping planes, which are not part of the BCF standard, and considering how to handle storing clipping planes as objects without affecting document versioning. Caio inquires if the generic collaboration framework is internal or external to FreeCAD, expressing interest in real-time collaboration. Pieter clarifies that the framework is more modest than real-time editing, focusing on topics and comments for file exchange, and was labeled GSOC due to a past BCF plugin project. He designed the proposal to allow for future extensions, including real-time editing.

The phrase **"was labeled GSOC due to a past BCF plugin project"** is
pieterhijma himself, in a live meeting, confirming that the FreeCAD/FreeCAD#25681
issue's `GSoC` label traces back to this exact 2019 BCF plugin history — i.e.,
the "old GSoC annotation collaboration project" luzpaz was trying to recall
and the reason #25681 carries a `GSoC` label at all are one and the same
lineage.

## Relationship to the currently-active thread

- pieterhijma has a standing, unmerged PR against the *old* plugin itself,
  updating it to FreeCAD 1.0: https://github.com/podestplatz/BCF-Plugin-FreeCAD/pull/10
  (referenced in #25685 comment 3.9 — see section 3 above). As of FEP-0011
  discussion comment (2026-03-07), pieterhijma notes: "the BCF plugin is not
  maintained anymore; my PR to update to FreeCAD 1.0 is still sitting there."
- The BCF plugin is explicitly listed as one of the "known initiatives" the
  new generic collaboration framework (FEP-0011) is designed to eventually let
  be rebuilt on top of (see FEP-0011 Motivation section below).

## The 2017 GSoC year question, settled

An earlier pass of this research flagged some ambiguity about which GSoC
year the BCF project belonged to, since a separate "GSoC 2017: accepted
proposals" thread (`forum.freecad.org/viewtopic.php?t=22229`) existed and
raised the possibility of a multi-year proposal-then-acceptance gap. Reading
that thread directly settles it: **the 2017 GSoC round's four accepted
proposals were Kurt Kremitzki (Part Design Workbench Refinement), Markus
Hovorka (Elmer Integration), Amritpal Singh (Rebar Addon for FreeCAD), and
Ajinkya Dahale (Topological Naming in FreeCAD)** — no BCF or PDM project of
any kind. Combined with the BCF proposal thread's own dates (first draft
2019-04-06, accepted 2019-05-08), this fully confirms **2019** as the only
correct year for Patrick Podest's project — the "2017" association in
earlier research was a false lead from an unrelated year's thread turning up
in the same search results.

---

# 8. The FreeCAD Enhancement Proposal (FEP) process

## 8.1 What a FEP is

Repository: https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals

Per the repo and FEP-0001 ("The new FEP Process"): a FreeCAD Enhancement
Proposal (FEP) is a design document used to discuss substantial changes to
FreeCAD. It is the primary mechanism for decision-making about important
FreeCAD development matters — both provoking discussion and reaching a
decision on how to proceed. Each FEP lives in its own directory
(`FEPs/FEP-XXXX-slug/README.md`) and has a structured front-matter table
(Type, Status, Author(s), Version, Created, Updated, Discussion link,
Implementation link) followed by Motivation / Rationale / Specification /
Impact / Backwards Compatibility / Implementation / Changelog sections. FEPs
are explicitly licensed CC0 1.0 Universal.

Every FEP has a companion **GitHub Discussion** (category: "FEP Discussion -
In-Process Proposal Discussion") where community feedback is gathered before
the FEP is accepted/rejected.

## 8.2 Full list of FEPs found (as of 2026-09-13)

Merged/numbered FEPs on `master` branch:

| FEP | Title | Status (per repo) |
|---|---|---|
| FEP-0000 | Template | — |
| FEP-0001 | The FEP Process (+ later amendment: "Formally require developer meeting discussion") | Active |
| FEP-0003 | Release Schedule and Process (CalVer) | Accepted |
| FEP-0006 | Materials Editor | Accepted |
| FEP-0007 | Consistent language across FreeCAD | Accepted |
| FEP-0008 | Project Group Structure | Accepted |
| FEP-0010 | Variant Parts | Accepted |

Open PRs proposing new/updated FEPs not yet merged (i.e., still in Draft/discussion), found via the repo's PR list:

| PR | FEP | Title | Author |
|---|---|---|---|
| #13 | FEP-0005 | Dependency and Platform Policy | (open) |
| #14 | FEP-0002 | Asynchronous Document Recompute and Multithreading Infrastructure | (open) |
| #17 | FEP-0004 | Python API Versioning | (open) |
| #26 | FEP-0009 | Sketch references (External Geometry & Attachment Supports) across Bodies and Parts | (open) |
| **#38** | **FEP-0011** | **Generic Collaboration Framework** | **pieterhijma** |
| #52 | FEP-0012 | Gating Network Access | (open) |
| #55 | FEP-0013 | Visual Diffs | pieterhijma |
| #59 | FEP-0014 | Forms | (open) |
| #61 | FEP-0015 | Icon Theming | (open) |

Note the numbering gap: no FEP-0011 PR/content exists on `master` yet — it is
still an open PR (#38) against `pieterhijma`'s fork branch
`generic-collaboration-framework`, not yet merged, meaning FEP-0011 is
**Draft** status, not yet Accepted.

## 8.3 FEP-0011: Generic Collaboration Framework — full text

Source: raw file on pieterhijma's fork,
https://raw.githubusercontent.com/pieterhijma/FreeCAD-Enhancement-Proposals/generic-collaboration-framework/FEPs/FEP-0011-generic-collaboration-framework/README.md
(the branch backing open PR #38, https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals/pull/38)

- **Status:** Draft
- **Author:** Pieter Hijma (@pieterhijma)
- **Version:** 0.2
- **Created:** 2026-01-25 | **Updated:** 2026-04-04
- **Discussion:** https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals/discussions/40

> # FEP-0011 Generic Collaboration Framework
>
> This document proposes to add a generic collaboration framework to FreeCAD.
> The generic collaboration framework is a new module inside FreeCAD that
> provides a standardized set of features for collaboration.  Although the
> generic collaboration framework is standalone and can be used as is, it can
> also be extended by external addons for specific use-cases while maintaining
> the possibility to provide collaboration between these external addons.
>
> ## Motivation
>
> Now that there are various initiatives (see below) that support or could
> benefit from collaboration features, it would be good to have a shared and
> standardized set of building blocks for collaboration.  These building blocks
> can then be reused and extended by various different initiatives.  This ensures
> that 1) not every project needs to reinvent the wheel and 2) collaboration
> between those different initiatives remains possible.
>
> Current known initiatives that could benefit from a generic collaboration framework are:
> - [BCF plugin for FreeCAD](https://github.com/podestplatz/BCF-Plugin-FreeCAD),
> - [CADBaseLibrary](https://cadbase.rs/en/) and its [Addon](https://github.com/mnnxp/cadbaselibrary-freecad),
> - [FreePDM](https://github.com/grd/FreePDM),
> - [nanoPLM](https://github.com/alekssadowski95/nanoPLM),
> - [Taack FreeCAD PLM](https://taack.org/en/app/Plm) and its [Addon](https://github.com/Taack/taack-plm-freecad), and
> - [Ondsel Server](https://github.com/FreeCAD/Ondsel-Server) and its [Addon](https://github.com/FreeCAD/Ondsel-Lens-Addon/).
>
> The following issues are relevant:
>
> - [Core: Collaboration features](https://github.com/FreeCAD/FreeCAD/issues/25681)
>   - [Core: Improve annotations as a collaboration feature](https://github.com/FreeCAD/FreeCAD/issues/25682)
>   - [Core: Add conversations to annotations](https://github.com/FreeCAD/FreeCAD/issues/25685)
>
> Issue [Core: Improve annotations as a collaboration feature](https://github.com/FreeCAD/FreeCAD/issues/25682) has already a PR associated with it:
> - [#26306: Collaboration: Add a generic collaboration framework](https://github.com/FreeCAD/FreeCAD/pull/26306).
>
> The BCF plugin of itself is already a collaboration framework that follows the
> BCF standard.  However, if any of the other projects would like to also acquire
> collaboration features, the code cannot easily be reused, for example because
> of licensing issues or because the code of the BCF plugin is tailored to
> BCF-specific features.  So, the main motivation for this generic collaboration
> framework is reuse of building blocks and a standardized set of collaboration
> features within FreeCAD.
>
> ## Rationale
>
> An important design choice is to make a generic collaboration framework that is
> useable as is.  However, as explained in the Motivation section, an important
> goal is that the collaboration framework can be extended by means of external
> addons that target specific use-cases.  As a collaborator of the Ondsel Server
> and its Addon, I would like to build on and add functionality to the
> collaboration framework for Lens-specific features.  As an example, Lens
> supports versions, so I would like to be able to extend the collaboration
> framework to allow users to refer to specific versions of a model.
>
> The collaboration framework will be written mostly in Python because this
> allows external addons to understand how the collaboration framework works and
> how to extend it.  Small parts need to be written in C++ because the framework
> makes use of `App::AnnotationLabel` for annotations.  Because
> `App::AnnotationLabel` only supports absolute positioning, it is extended to
> associate annotation labels with the geometry, for example with faces or edges.
> This allows to reposition the geometry with the annotation labels to reposition
> itself as well.
>
> Since an important goal is that the collaboration framework is used by external
> addons, the provided API is versioned to make sure that external addons can
> target a specific API version.  This allows the collaboration framework to
> evolve and introduce a new API version without breaking functionality of the
> external addons that use older API versions.
>
> An important design decision is that this generic collaboration framework is
> aligned with the [BIM Collaboration Format (BCF)
> standard](https://www.buildingsmart.org/standards/bsi-standards/bim-collaboration-format/).
> This standard is issued by [BuildingSMART](https://www.buildingsmart.org/), an
> organization for open standards in the BIM field that also introduced the [IFC
> Standard](https://www.buildingsmart.org/standards/bsi-standards/industry-foundation-classes/).
>
> FreeCAD has an (unmaintained) [external addon for
> BCF](https://github.com/podestplatz/BCF-Plugin-FreeCAD).  Since this was one of
> the first collaboration addons for FreeCAD, the collaboration features are
> baked in and cannot be easily reused.  The goal of this generic collaboration
> framework is to prevent such a situation and to make sure that the old external
> addon can be revised to build on top of this work.
>
> A distinct design decision of the BCF standard is that the comments are stored
> in `.bcf` files separate from the `.icf` files.  As opposed to that design
> decision, this generic collaboration framework stores the collaboration
> information in the FreeCAD document.
>
> However, storing collaboration information in the FreeCAD document has
> drawbacks.  For example, a new comment to a model would change the file, which
> could mean that the version of the file needs to be updated, for example in a
> PLM/PDM setting.  This means that there is a valid reason to store discussions
> on models separate from the model as the BCF standard prescribes.  This is
> perfectly possible with FreeCAD making use of `App::Link`: If the original file
> needs to stay unmodified, users can create a second FreeCAD document that
> contains a link to the model from the original file and make comments in that
> document.
>
> ## Specification
>
> We first provide an overview of the generic collaboration framework and then
> give details on how to customize the framework, an inherent goal of the
> framework.
>
> ### Overview
>
> The generic collaboration framework is a new module called "Collaboration" that
> is part of the FreeCAD source code.  The collaboration framework provides
> several constructs to foster collaboration among users.  The main construct is
> a Topic.  A **Topic** is document object that is created by a user.  Optionally
> a topic has an **Annotation** in the 3D view with the topic title that can
> refer to geometry in two ways: It can be globally positioned or it can be
> attached to geometry.
>
> A topic is a collection of **Comments** created by **Users**.  A sequence of
> comments is the main contents of a topic and forms a **History**.  A
> **Comment** is a string of text associated with a **Date**, **Time**, and
> **User**.  A **User** is someone who can be uniquely identified and
> distinguished from other users and is capable of making comments.  How users
> are represented can be customized (more on this later) but as a minimal
> implementation, users are identified by email addresses.  The history of
> comments inside topics allow users to discuss models.
>
> As collaboration feature, users can make **Snapshots** of a model.  A
> **Snapshot** is a screenshot of the model in its current state.  Another
> collaboration feature is a **Viewpoint**, a camera position that can direct the
> user to view a model in a particular way.
>
> The text of a comment can contain **Links** that can reference various items.
> **Links** have a standardized set of items they can refer to, but it is also
> possible to have **Custom Links** which will be discussed in the next section.
> The standardized set of items that a link can refer to is:
> - other topics,
> - other comments,
> - snapshots,
> - viewpoints, and
> - users.
>
> Comments are entered by users in a text field that supports a subset of
> [Markdown](https://www.markdownguide.org/).  This allows users to easily create
> links or minor markup features such as making text bold.
>
> Links are created using Markdown syntax.  The following URL schemes are used
> for the items above:
> - topics: `topic:identifier`,
> - comments: `comment:identifier`,
> - snapshots: `snapshot:identifier`,
> - viewpoints: `viewpoint:identifier`,
> - users: `user:email address`, and see below.
>
> ### Customization
>
> Although the proposed collaboration framework can be used as is, an important
> design goal is that the generic collaboration framework can be used by external
> addons to adapt the collaboration framework for specific use-cases.  We call
> these addons **Adaptors** that adapt the generic collaboration framework
> through an API.
>
> Since the API is specifically meant to be used by addons, the API is versioned
> to allow the API to evolve while ensuring that old versions of the API remain
> working for addons that make use of it.
>
> The main customization points are **Users** and **Links**.  For **Users** to
> make use of the collaboration framework, a user must register itself.
> Registrations can happen by means of the default mechanism, or the default
> mechanism can be overridden by adaptors.  The default mechanism allows users to
> provide their name and email address while adaptors can disable this and
> provide their own means of providing the user's identity.
>
> - `register_user(adaptor_id: str, name: str, username: str) -> None`: This
>   function registers the name and username for a specific adaptor and disables
>   the default way to authenticate a user.
> - `unregister_user(adaptor_id: str, username: str) -> None`: This function
>   unregisters the user and enables the default way to authenticate users.
> - `register_user_handler(adaptor_id: str, user_handler: Callable[[link: str,
>   context: str], None] -> None)`: This function registers a handler that is going to
>   be called if a user for the adaptor is clicked in a certain context.  The URL
>   schema for the user becomes `user:adaptor_id:username`.  The context is a
>   string that specifies the context for an action and can be one of the
>   following: `create-reply`, `view-user`.
> - `unregister_user_handler(adaptor_id: str)`: This function unregisters a
>   user handler.
>
> Another point for customization is **Links**.  Adaptors can introduce their own
> URL scheme and install handlers for that:
>
> - `register_url_scheme(adaptor_id: str, scheme: str, handler: Callable[[link:
>   str, context: str], None])`: Clicking a link of that type, will call the
>   handler with the provided link and the context.  This allows the adaptor to
>   handle the link according to the use-case of the adaptor.  The context gives
>   the adaptor an idea of where the link is used.  Valid values are `comment`
>   and `topic-title`.
>
> To improve the user experience, the `@` key will enable autocompletion for the
> various types of links.  The auto-completer will give suggestions based on user
> input.  Adaptors can register their own completer with the following function:
>
> - `register_completer(adaptor_id: str, completer: Callable[[prefix: str],
>   List[tuple[str, str]]])`: This function registers a completer for a specific
>   adaptor.  Given a prefix string (typed immediately after `@`), the completer
>   will return a list of matches.  A match is a tuple of two strings with the
>   first string being the representation for the user, for example a username,
>   and the second the link to be inserted.
> - `unregister_completer(adaptor_id: str) -> None`: This function unregisters
>   the completer for an adaptor.
>
> For example, if a user types `@use` or `@Pie` given the registered urls, an
> adaptor could provide the following list:
> ``` python
> [('Pieter Hijma', 'user:my-adaptor:pieterhijma')]
> ```
>
> Since it cannot be assumed that an adaptor is installed when a document is
> opened with custom url schemes, an adaptor has to register itself and its url
> schemas inside a document.  The API to register itself is:
>
> - `register_adaptor(adaptor_id: str, adaptor_name: str, url: str)`: This
>   function registers an adaptor with a specific identifier, a name, and a URL
>   where the adaptor can be found, ideally a link to the addon manager.
>
> When URL schemas are registered, the URL schemes are stored in the document
> where the topics occur.  Below are the properties and contents that are used
> for an adaptor with id `myadaptor` with name `My Adaptor` and url
> `https://myadaptor.org`, and a url schema `myadaptormodel`:
>
> ```
> Adaptors: [('myadaptor', 'My Adaptor', 'https://myadaptor.org')]
> Schemas: [('myadaptor', ['myadaptormodel'])]
> ```
>
> When a document is opened in FreeCAD that does not have "My adaptor" installed,
> clicking a link with scheme `myadaptormodel` will give the user a message
> indicating that to view this link, "My adaptor" will have to be installed.
>
> Please note that all collaboration information is accessible and readable to
> all FreeCAD users, even though they don't have the correct adaptor installed,
> but to make use of the links, it is required to install the correct adaptor.
>
> The collaboration framework supports all features that are required for the
> [BIM Collaboration Framework
> v3.0](https://github.com/buildingSMART/BCF-XML/tree/release_3_0/Documentation).
>
> ### Impact on existing features / subsystems
>
> Since the generic collaboration framework will be contained in its own module,
> there is almost no impact to existing features or the subsystem.  Since the
> generic framework builds on `App::AnnotationLabel`, this document object and
> its viewprovider may be changed minimally.
>
> ### Backwards Compatibility (only for Core Changes)
>
> This core change will not have any impact on backwards compatibility.
>
> ## Implementation (only for Core Changes)
>
> There is an initial PR available to study: [#26306: Collaboration: Add a
> generic collaboration
> framework](https://github.com/FreeCAD/FreeCAD/pull/26306).  This PR mainly
> introduces the new Collaboration module and its central document object
> `Collaboration::Topic`.  This is a subclass of the already existing
> `App::AnnotationLabel`.  Topics have a title and an optional label in the 3D
> view with the title.  Clicking on the label in the 3D view or on the document
> object in the object tree opens up a task panel for the topic.  There is no
> support for comments yet and this is planned for subsequent PRs.
>
> ## Changelog (once more versions are released)
>
> ### 0.2 - 2026-04-04
>
> - Emphasize that the generic collaboration framework can be used standalone as well.
>
> ### 0.1 - 2026-03-03
>
> - Initial version
>
> ## License / Copyright
>
> All FEPs are explicitly [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

## 8.4 FEP-0011 PR #38 comments (on the FEP repo)

https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals/pull/38 — 2 issue-level comments (separate from the linked Discussion #40 below):

### 8.4.1 [marcuspollio](https://github.com/marcuspollio) — 2026-03-06T10:46:24Z (later minimized as "outdated")

> Hi @pieterhijma nice write-up!
> What about introducing the concept of `Project` as a top-level construct? A Project could contain several Topics, could be used/shared across multiple files, help manage users and other general metadata, and would allow to follow the evolution/life-cycle of Topics resolution.
> Could this concept work or maybe is it not generic enough?

### 8.4.2 [pieterhijma](https://github.com/pieterhijma) — 2026-03-06T13:06:50Z

> Hi @marcuspollio, sorry I didn't incorporate the link to https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals/discussions/40. Perhaps it is good to repeat there.
>
> I wouldn't be against it, but in principle, you could create a project.FCStd file where you link to various topics in the documents within the project. This is also not something that BCF supports. Do you have experience with something like that in a BIM context?

## 8.5 PR #26306 "Collaboration: Add a generic collaboration framework" (the actual code)

https://github.com/FreeCAD/FreeCAD/pull/26306

- **State:** Open, **Draft** (not merged, not mergeable status "unknown")
- **Author:** pieterhijma
- **Branch:** `pieterhijma:generic-collaboration-framework` → `FreeCAD:main`
- **Created:** 2025-12-19T20:25:20Z | **Updated:** 2026-08-20T12:19:40Z
- **Labels:** `Mod: Core`, `Packaging/building`, `Type: Feature`, `Requires: UI/UX review`, `Requires: FEP`
- **Milestone:** 26.3 (due 2026-09-30)
- **Stats:** 4 commits, +1264/-16 lines, 30 files changed
- **Closes:** #25682
- **Issue-level comments:** 4 | **Review (diff) comments:** 6

### Body

> This module introduces generic functionality for collaboration among FreeCAD users.  The main document object is `Collaboration::Topic` that is a subclass of `App::AnnotationLabel`.  For now, a topic only has the label in the Tree, annotation that can be associated with geometry, and a title.
>
> Future work is to add comments to a topic which will be part of a subsequent PR.
>
> This feature was discussed in #25682 but this PR does not fully satisfy all of the requirements there. Sorting the labels in the top-right corner was very challenging and provided sub-par experience (the labels would not be in a stable position at the top-right), so this PR leaves that to future work.
>
> Compared to `App::AnnotationLabel`, the collaboration module provides the following additions:
> - `Collaboration::Topic` can be linked to geometry in such a way that if geometry is transformed, the label moves with it. Absolute positioning (as `App::AnnotationLabel` provides) is also still possible.
> - The topics that are created are organized in a group "Topics"
> - There is a task panel opened in which currently the label in the tree and the title of the topic can be changed. This panel will support comments (or conversations) in future work.
>
> ## Issues
>
> Closes #25682
>
> ## Before and After Images
>
> There is no before.
>
> [screenshot image, not reproducible in text]
>
> [video demo, not reproducible in text]

### Issue-level comments (chronological)

**[prokoudine](https://github.com/prokoudine) — 2025-12-20T21:29:02Z**
> Quick question. Seeing how the nomenclature uses "topics"... The state-of-the-art UX for discussions in documents is to have topics where you can have an original comment, replies to it, and being able to mark a topic as solved. This proposal offers something that looks more like separate annotations. Are you planning to expand the concept?

**[ickby](https://github.com/ickby) — 2025-12-22T10:20:29Z** (this is Stefan Tröger, author of the now-archived CollaborativeFC/OCP real-time collaboration project referenced in `possible_freecad_collaboration.md` §1)
> One of the biggest parts of such a framework would IMHO need to be a way to track identity, to be sure who did comment what exactly. Without knowing Author, Date etc. of the comments, and having them somewhat verified, they become less useful.
>
> As identity verification is a very tough topic, and FreeCAD should never be bound to any fixed service, I would assume a API is required to add multiple ID verification providers, one being "local unverified" (set everything just in preferences), maybe one could be the lens server ID, maybe some hacker-space has its own implementation? Then the topic description could show user information, and if it could be verified or not.
>
> Maybe that is to much to ask, but IMHO super beneficial. I see this usage in word documents often and it works remarkable well in high stakes environments between untrusting partners.
>
> (this comment received 1 "+1" reaction)

**[pieterhijma](https://github.com/pieterhijma) — 2025-12-22T13:17:26Z**
> > Quick question. Seeing how the nomenclature uses "topics"... The state-of-the-art UX for discussions in documents is to have topics where you can have an original comment, replies to it, and being able to mark a topic as solved. This proposal offers something that looks more like separate annotations. Are you planning to expand the concept?
>
> Yes, that's the idea. This belongs more to [this issue](https://github.com/FreeCAD/FreeCAD/issues/25685) though. We agreed that this should be a FEP anyway, so all those things will be ironed out there more.
>
> (this comment received 1 "+1" and 1 "rocket" reaction)

**[pieterhijma](https://github.com/pieterhijma) — 2025-12-22T13:18:56Z**
> > One of the biggest parts of such a framework would IMHO need to be a way to track identity, to be sure who did comment what exactly. Without knowing Author, Date etc. of the comments, and having them somewhat verified, they become less useful.
>
> Indeed, this is part of it as well and I would like to do it such that it is generic.
>
> (this comment received 2 "heart" reactions)

### Review (diff) comments

- **github-advanced-security[bot]** (2025-12-19T21:15:14Z) flagged 4 unused-import lint warnings in `src/Mod/Collaboration/Init.py` (lines 23, 24), `src/Mod/Collaboration/InitGui.py` (line 25), and `src/Mod/Collaboration/collaboration/gui/components/task_topic_dialog.py` (`Collaboration_rc` unused) — routine CI code-scanning noise, not substantive design feedback.
- **maxwxyz** (2026-03-18T19:47:45Z), on `src/Mod/Collaboration/Gui/Command.cpp`: suggested tooltip text should use third-person present tense ("Creates a topic" not "Create a topic").
- **maxwxyz** (2026-03-18T19:47:45Z), on `src/Mod/Collaboration/Gui/Resources/ui/task_topic_dialog.ui`: suggested the UI file's placeholder string should read "Topic" to reflect the actual title, noting it's probably overwritten at runtime anyway.

## 8.6 GitHub Discussion #40 — "FEP-0011: Generic Collaboration Framework" (full thread, verbatim)

https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals/discussions/40 — category "FEP Discussion - In-Process Proposal Discussion." Opened by pieterhijma, 2026-03-03T15:02:17Z:

> Discussion on adding a new generic collaboration framework to FreeCAD:
>
> ### Important Links
>
> - 🔍 rendered proposal: [FEP-0011: Generic Collaboration Framework](https://github.com/pieterhijma/FreeCAD-Enhancement-Proposals/blob/generic-collaboration-framework/FEPs/FEP-0011-generic-collaboration-framework/README.md)
> - 💬 official discussion: https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals/discussions/40

17 top-level comments, chronological, pulled verbatim via `api.github.com/repos/FreeCAD/FreeCAD-Enhancement-Proposals/discussions/40/comments`:

**[hyarion](https://github.com/hyarion) — 2026-03-04T22:42:38Z**
> I don't understand why we need a collaboration module in the repo when it would only be used by addons. Can't that just as easily be a library/addon that Lens and others depend upon instead?
>
> Maybe a better approach would be if you guys start collaborating on this shared standardized framework first and then we move it into the main repository if needed?
>
> I agree that the existing annotation object needs some love though.

**[marcuspollio](https://github.com/marcuspollio) — 2026-03-06T13:48:04Z**
> Hi @pieterhijma nice write-up!
> What about introducing the concept of `Project` as a top-level construct? A Project could contain several Topics, could be used/shared across multiple files, help manage users and other general metadata, and would allow to follow the evolution/life-cycle of Topics resolution.
> Could this concept work or maybe is it not generic enough?
>
> Pieter answer:
> > I wouldn't be against it, but in principle, you could create a project.FCStd file where you link to various topics in the documents within the project. This is also not something that BCF supports. Do you have experience with something like that in a BIM context?
>
> This [BCF-XML](https://github.com/buildingSMART/BCF-XML/tree/release_3_0/Documentation#project-bcfp-file) link you provided in the FEP-0011 mentions a project reference topics belong to. I have not used it nor know the details but it could help as inspiration?
>
> I have limited experience of proper yet simple and well though-out collaboration tools in the BIM context. Emails (with external hosting services) and phone are still the main collaboration method among SMEs 😉 Bigger structures use specialized tools such as Speckle, Autodesk stuff and other proprietary cloud-based SaaS. I have used ArchiCAD collaborate a bit, that allows multiple users to work on the same project almost simultaneously (local network or remote server), with some revision system and annotation tools. While convenient when it works, a pretty nightmare when it does not. The Bonsai (Blender extension for BIM) project has introduced since a few years a git-based revision system developed mainly by @brunopostle that works fairly well (though I have not used it in production yet).
>
> Yes, maybe this `Project` concept could be used to extend the FreeCAD [Project Information](https://wiki.freecad.org/Std_ProjectInfo/en) instead if defining a `main` project file with children/links, and the collaboration framework could just read their metadata and construct this "graph" between this `main` file and the others linked ones?

**[pieterhijma](https://github.com/pieterhijma) — 2026-03-06T16:36:29Z**
> Because of the following reasons:
> - Currently FreeCAD has virtually no support for collaboration, whereas its open source nature makes it very convenient for exactly collaboration.  Having support in FreeCAD stengthens FreeCAD's position as a tool for collaboration.
> - It is not only used by addons, it can be used standalone as well.
> - Having a standalone collaboration framwork ensures that users with a FreeCAD document with collaboration information in it can always read the topics without having to install or download an external workbench.  The specialized links are not available if you don't have the correct addon, but the user will be notified.
> - Adding it to the FreeCAD source code ensures that we can make use of C++ and extend AnnotationLabel, for example.  Creating a separate module ensures that the core of FreeCAD is minimally touched.  Other candidates for C++ interaction is the GUI for these specialized links.
>
> An alternative is to implement collaboration in an addon.  The BCF plugin is an example of this and it has logic that I and others could make use of.  I could simply try to see if the license is compatible, port it and then there are two addons that share the code.  It could be a possibility then to extract the common parts and create an underlying collaboration addon for it.  This is only successful if my addon and BCF make use of this, so BCF would have to be ported as well.  This is all possible, but the likelihood of success for such a setup is much greater if that underlying collaboration module is part of the FreeCAD source code.  It would provide more incentive to make use of that framework if it were in the FreeCAD source code.  Additionally, the underlying framework would be limited to Python only.
>
> What is more likely to happen in the case above is that my addon and the BCF share things conceptually but probably not in code because it is takes time to come up with a generic underlying layer and to port BCF to the new framework.

**[hyarion](https://github.com/hyarion) — 2026-03-06T18:47:54Z**
> To me it sounds a lot like: [xkcd "Standards" comic image, not reproducible in text]
>
> Have you discussed this with the other projects? Are they interested in adopting their system? Maybe you could collaborate on it together to prototype what you need before trying to standardize it? Paddle stroke introduced me to YAGNI principle earlier today and as John Carmack said: it is rarely architecting for future requirements turns out net-positive.
>
> I do think it is good to include things like better annotations. That's already something that people want to use outside of collaboration, like for Measurement, DFM, and maybe even GD&T.

**[pieterhijma](https://github.com/pieterhijma) — 2026-03-07T11:09:25Z**
> To me that sounds a lot like: 🙂
>
> [xkcd-style meme image, not reproducible in text]
>
> Isn't this the discussion? Your reactions come very fast, even before [the index has been updated to include this FEP](https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals/pull/39). To have a clear direction for any discussion, I prefer to come with something concrete before the discussion starts, hence my initial PR to which the reaction (rightfully) came to set up an FEP for this.
>
> Perhaps some background: As I'm working for NLnet's [Lens/FreeCAD integration](https://nlnet.nl/project/Lens-FreeCAD-integration/) project I have a mandate to make my work as publicly available as possible. So, although I could add collaboration features as much as possible in the Lens addon, I try to make what I do available to other parties as well. Hence, this effort from me to try to think how others can benefit from what I'm doing as well. So, I'm trying to do the right thing here! Discussion with other projects assumes that all those projects are aligned in time, but you can already see that the BCF plugin is not maintained anymore; my PR to update to FreeCAD 1.0 is still sitting there.
>
> Regarding the argument of architecting for the future: I wish that the author of BCF had done what I'm doing now. So, it's 6 years apart then. In open source it's difficult to get alignment in time, which is different in companies or so. Additionally, this is not architecting for the future. The features are part of my project. This also nullifies the YAGNI, I do need them.
>
> I do understand the reservations to add another module to FreeCAD's source code. That is a very valid point because there is a risk that what I came up with is useless and only one workbench makes use of it. Then it has to be maintained forever since in FreeCAD we have no good way to deprecate things and get rid of things. However, I think this is exactly why we have the FEP process. To evaluate whether this is a good idea.
>
> But again, your reservations are noted and I fully understand them. So there are arguments in favor of adding it to the repo and there are arguments against. The task is to evaluate if what I described is contained enough and useful enough to warrant this work being included in the FreeCAD repo.
>
> Suppose the verdict is not, what would be an alternative path? So, what I currently did in C++ is essentially extending `App::AnnotationLabel` giving it a group extension for comments, making it double-click aware, and being able to attach to geometry. I didn't want to make `App::Topic`, but preferred `Collaboration::Topic` to separate it from the core. I think this separation is good, but indeed, we are adding then a new module to the FreeCAD source code.

**[hyarion](https://github.com/hyarion) — 2026-03-07T15:58:47Z**
> Just so I'm clear, I don't mind Annotations, they are great generic features which are used in various places already. There are probably other generic features which could be included as well.
>
> I also don't mind extending FreeCAD to allow easier implementations of features in addons.
>
> The issue I see is to include modules and features that isn't used by anything in FreeCAD itself (only by the ecosystem).
>
> I get that you want to make a library that others could use too, but I don't understand why it should live in the main repository.

**[pieterhijma](https://github.com/pieterhijma) — 2026-03-13T16:00:39Z**
> > Just so I'm clear, I don't mind Annotations, they are great generic features which are used in various places already. There are probably other generic features which could be included as well.
>
> Great.
>
> > I also don't mind extending FreeCAD to allow easier implementations of features in addons.
>
> Great.
>
> > The issue I see is to include modules and features that isn't used by anything in FreeCAD itself (only by the ecosystem).
>
> That is not the case. The collaboration framework is stand alone as well. It can be used by users to create topics and write comments.
>
> > I get that you want to make a library that others could use too, but I don't understand why it should live in the main repository.
>
> It's not a library alone. It contains annotations. It lives in the main repository because of the C++.

**[pieterhijma](https://github.com/pieterhijma) — 2026-03-13T16:32:56Z**
> > This [BCF-XML](https://github.com/buildingSMART/BCF-XML/tree/release_3_0/Documentation#project-bcfp-file) link you provided in the FEP-0011 mentions a project reference topics belong to. I have not used it nor know the details but it could help as inspiration?
>
> You're right, indeed, there is a project defined in the BCF standard. However, a project is nothing more than an ID and a name and .bcf files record that they are part of that project. So, what I said earlier is still true. You can easily make a "parent" freecad that links into the files to group topics.
>
> > I have limited experience of proper yet simple and well though-out collaboration tools in the BIM context. Emails (with external hosting services) and phone are still the main collaboration method among SMEs 😉 Bigger structures use specialized tools such as Speckle, Autodesk stuff and other proprietary cloud-based SaaS. I have used ArchiCAD collaborate a bit, that allows multiple users to work on the same project almost simultaneously (local network or remote server), with some revision system and annotation tools. While convenient when it works, a pretty nightmare when it does not. The Bonsai (Blender extension for BIM) project has introduced since a few years a git-based revision system developed mainly by @brunopostle that works fairly well (though I have not used it in production yet).
>
> Cool, thanks for these insights!
>
> > Yes, maybe this `Project` concept could be used to extend the FreeCAD [Project Information](https://wiki.freecad.org/Std_ProjectInfo/en) instead if defining a `main` project file with children/links, and the collaboration framework could just read their metadata and construct this "graph" between this `main` file and the others linked ones?
>
> Currently "Project Information" is merely "document information". Indeed, it would be good to have an overview with linked documents.

**[maxwxyz](https://github.com/maxwxyz) — 2026-03-21T16:51:01Z**
> @pieterhijma for the SnapShot and ViewPoint we should make the object to be compatible with the proposed GD&T Views and idea for Saved Clipping Planes, maybe we can have one for all.
>
> What I envision for the save view / saved clipping plane tool:
> - Create Document Objects for each clipping plane in the document. This way multiple could be active and it can be activated, stored, deactivated,...,
> - Ability to store camera position/rotation and also object visibility in that object and also clip direction (similar to Draft WBs),
> - Double click to show or edit.,
> - Two optional lists: objects to cut, objects to exclude (default cuts all objects),
> - Property to reverse clipping direction.,
> - It should display a plane.,
> - Possibility to attach to objects (maybe derived from datum plane?),
> - If not attached, able to drag and rotate that clipping plane (there is already a built in coin clipping plane that offers that in FEM WB),
> - Option to create the section profile as geometry (there is already a Cross-Section tool in Part but it works only with one object today and is slow) (that would be the geometry of that clipping plane document object.,
> - Option to "close" the cut-face (default is closed) and showing the colors of the different bodies for that cut-face.
>
> Similar options already exist for the Draft Working Plane (store objects visibility and camera data).
>
> Related issues:
> - https://github.com/FreeCAD/FreeCAD/issues/16187,
> - https://github.com/FreeCAD/FreeCAD/issues/11169,
> - https://github.com/FreeCAD/FreeCAD/issues/16190,
> - https://github.com/FreeCAD/FreeCAD/issues/16193

**[marcuspollio](https://github.com/marcuspollio) — 2026-03-23T09:08:23Z**
> @maxwxyz For the Snapshots, Viewpoints, and Clipping planes, do you mean these features should be integrated into Core FreeCAD (e.g. `Gui`, not in a specific module like `Draft` currently), and the Collaboration framework would have some utilities to link/refer to these with some custom interaction?
> For example, some user posts a topic or comment via the Collaboration tools:
> e.g. "See *\[the clipping plane at the top](main_file#clipping_plane_01)*, the *\[blue screw](link_01#object_01)* should not fully pierce the *\[red base plate](link_02#object_02)*". Then other users click these links, viewpoint is set (orientation, visibility, clipping are restored), and said objects are highlighted or custom labels are attached or whatever?

**[maxwxyz](https://github.com/maxwxyz) — 2026-03-23T17:11:53Z**
> Yes in Core or Part. RT branch has the feature to save custom views. These views are also supported in STEP AP242

**[marcuspollio](https://github.com/marcuspollio) — 2026-03-24T08:01:02Z**
> I suppose the linking/referring logic implemented in the Collaboration framework should be generic enough (but probably quite complex) to support selecting/highlighting items (e.g. document objects, sub-elements or properties) also in non 3D-view workspaces, such as TechDraw, Spreadsheet, or others Workbenches and interfaces.
> For a start, only supporting objects exposed via the Tree View is enough?
>
> Also, there is perhaps some UI/UX design work to be done on the FreeCAD Core side first, that is:
> how to best integrate this select/highlight/go-to of such linked items and display some info in a nice way?
> Maybe the [Selection View](https://wiki.freecad.org/Selection_View) needs some love and could help achieve smoother workflows, and support specific features like https://github.com/FreeCAD/FreeCAD/pull/23989?

**[pieterhijma](https://github.com/pieterhijma) — 2026-04-04T11:04:31Z**
> I've created a new version in which I emphasize better that the framework is standalone as well.

**[pieterhijma](https://github.com/pieterhijma) — 2026-04-04T11:09:34Z**
> I'm reviewing this comment and I will get back to this. Specifically I have to make my mind up about these two things:
>
> A problem with clipping planes is that it defines then more than what the BCF standard supports. On the other hand, it also seems ludicrous to not support clipping planes.
>
> Regarding storing clipping planes as document objects: I'm considering how this would work in a situation in which we don't want to store the collaboration info as part of the main document, a situation that we would like to support (after a discussion with @pierreporte).

**[pierreporte](https://github.com/pierreporte) — 2026-04-06T20:58:07Z**
> @pieterhijma
>
> Like other said, I think that a container for all annotations, topics, etc. would be interesting, instead of having just a list of topics.
>
> I don't think that there should be just one annotation per topic. Maybe it would be better to have unlimited annotations per comment, and ability to attach a conversation (or topic) to an annotation. For example, someone starts an annotation about a particular hole (pointed to by an annotation), an a reply points to another feature.
>
> Great idea to have the ability to have separate annotation files using links, so that they can either stay in the file or be separate depending on what is needed.
>
> For clipping planes, there is a PR that if merged will bring them as document objects just like measures, so I don't know how it would interact with collaboration framework, especially if it needs to be both in the document and in the annotation container.

**[pierreporte](https://github.com/pierreporte) — 2026-04-07T15:46:47Z**
> I forgot to add the authentication. Is it really required to have that? It's necessary in organizations but if you take Word documents for instance you don't have to log in or register to write comments. Even when working in a company, you could send file to other organizations and they would add comments without being in your system.

**[pieterhijma](https://github.com/pieterhijma) — 2026-08-02T14:09:43Z**
> Sorry for the late notice, but I don't know well how to answer this. I've put it on the agenda of the developers meeting today.

This is the last activity in the discussion as of the fetch date (2026-09-13) — a ~4 month gap between the April flurry of comments and this August response, then confirmed still "Working on it" as of the September 6 developer meeting (see below).

## 8.7 Developer meeting mentions of FEP-0011

Source: https://github.com/FreeCAD/FreeCAD-developer-meetings (Minutes directory), fetched as raw markdown files.

**2026-04-05 meeting** (https://github.com/FreeCAD/FreeCAD-developer-meetings/blob/main/Minutes/minutes-2026-04-05.md), section "Collaboration Framework" — quoted in full in section 7 above (the "was labeled GSOC due to a past BCF plugin project" quote).

**2026-09-06 meeting** (https://github.com/FreeCAD/FreeCAD-developer-meetings/blob/main/Minutes/minutes-2026-09-06.md), the most recent status as of the fetch date:

> 9. **[FEP-0011: Collaboration Framework](https://github.com/pieterhijma/FreeCAD-Enhancement-Proposals/tree/generic-collaboration-framework/FEPs/FEP-0011-generic-collaboration-framework)** (pieterhijma)
> 	- Working on it, taking into account all comments of the [discussion](https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals/discussions/40) and last meeting's comments.

No minutes file exists between 2026-05-16 and 2026-08-15 (a gap — possibly meetings weren't held or minutes weren't published over that stretch), and the 2026-08-15 minutes do not mention the collaboration framework at all, so the immediate substance of what pieterhijma raised "at the developers meeting" on 2026-08-02 (per his discussion comment) is not captured in any minutes file found.

---

# 9. Related work: versioning and visual diffs

Not part of the #25681/FEP-0011 collaboration thread directly, but the same
author (pieterhijma) is independently working on two more FEPs that are
directly relevant to "PDM-shaped" work in FreeCAD — both surfaced via the
2026-09-06 developer meeting minutes (section 8.7 above) and worth tracking
alongside FEP-0011:

## 9.1 PR #28312 — "Core: Improve FreeCAD file format for versioning"

https://github.com/FreeCAD/FreeCAD/pull/28312 (Open, Draft: false per API — check live state before citing as merged)

> This PR is a potential solution to improve FreeCAD's file format for versioning, for example with Git.
>
> Concretely, this PR solves the following subset of versioning-related problems:
> - Redundant BRep files (that can be computed) are stored in the FreeCAD file.
> - Removing redundant BRep files comes at the cost of expensive recomputes.
> - A recompute of the FreeCAD file without any modifications to the model changes the FreeCAD file.
>
> The underlying principle is a **document cache**. With a **document cache**, each file can have its own, user-defined cache for files that can be recomputed, for example BRep files. With this PR, FreeCAD documents obtain a new path property `DocumentCacheDir` that a user can set to a specific directory. This can be a local directory on the computer, a directory within a versioned project that the versioning system ignores, or simply a temporary directory. When the document cache directory is empty (not set by the user), FreeCAD acts as always and BRep files and thumbnails are stored in the zip file.
>
> However, if the document cache directory is set, thumbnails and BRep files of document objects that can be recomputed, will be stored in the document cache directory and not in the FreeCAD zip file. A crucial aspect is whether a document object can compute its shape (and hence the BRep file).
>
> To indicate this, `Part::Feature` has a new hidden property `CanComputeShape` that is false by default. However, if we know that a document object c[...truncated in this fetch — full body not retrieved in this pass]

This is directly relevant to the "History Workbench" / git-friendly-FreeCAD-files thread already noted in `possible_freecad_collaboration.md` §5 — same underlying goal (make `.FCStd` diffable/versionable with a real VCS) approached from the file-format side rather than the tooling side.

## 9.2 FEP-0013: Visual Diffs

Branch: https://github.com/pieterhijma/FreeCAD-Enhancement-Proposals/blob/visual-diffs/FEPs/FEP-0013-visual-diffs/README.md (open PR #55, not yet merged)
Discussion: https://github.com/FreeCAD/FreeCAD-Enhancement-Proposals/discussions/56

- **Status:** Draft | **Author:** Pieter Hijma | **Version:** 0.2 | **Created:** 2026-08-19 | **Updated:** 2026-09-05

> This proposal introduces a generic workbench for visual diffs to FreeCAD.  A
> *visual diff* is a way to highlight differences between (versions of) files in
> a visual way.  Although this feature is very useful for CAD applications,
> FreeCAD has limited support for this.  In this document we propose to add a
> Diff workbench/module that allows 1) users to create visual diffs between
> versions of FreeCAD files and 2) provide external workbenches with an API
> to create versioning-based software.

Per the 2026-09-06 developer meeting minutes: "Update: discussion going well,
about to do a write a new version." Not fetched in full in this pass — flagged
here as a clear next-step document if the versioning/history side of FreeCAD
PDM work becomes the focus rather than the collaboration-annotation side.

---

## Not yet fetched (remaining leads for further digging)

- **`podestplatz/BCF-Plugin-FreeCAD` PR #10** — https://github.com/podestplatz/BCF-Plugin-FreeCAD/pull/10 — pieterhijma's still-unmerged update of the BCF plugin to FreeCAD 1.0.
- **BCF-XML standard docs** — https://github.com/buildingSMART/BCF-XML/tree/release_3_0/Documentation — the normative standard behind BCF.
- **The other "known initiatives" listed in FEP-0011's Motivation section** — CADBaseLibrary (https://cadbase.rs/en/ + https://github.com/mnnxp/cadbaselibrary-freecad), Taack FreeCAD PLM (https://taack.org/en/app/Plm + https://github.com/Taack/taack-plm-freecad), and the Ondsel-Lens-Addon (https://github.com/FreeCAD/Ondsel-Lens-Addon/) — named but not yet independently researched here. FreePDM and nanoPLM were already covered at a survey level in `possible_freecad_collaboration.md` §2.
- **PR #28312's full body/comments** — only the first ~1500 characters of the body were captured in this pass (see section 9.1); worth a full fetch if the versioning-file-format angle becomes the active research target.
- **FEP-0013 Visual Diffs full text and discussion #56** — only the front-matter and opening paragraph were captured; not yet a full fetch.
- **The exact FreeCAD developer meeting where pieterhijma raised the FEP-0011 authentication question "today" (his 2026-08-02 discussion comment)** — no minutes file was found covering early August 2026 (the archive jumps from 2026-05-16 to 2026-08-15, and the 08-15 minutes don't mention it).

No other FreeCAD-org issue numbers appear anywhere in the bodies or comments captured in section 1-6 above, beyond what's now been traced through in sections 7-9.

---

