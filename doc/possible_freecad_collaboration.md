# FreeCAD Collaboration Efforts: What's Live, What's Stalled, and Where This Project Could Plug In

Purpose: a tangent off the core research

not a scoped plan to build mechanical-CAD support,
but a survey of the real-world state of collaborative/multi-party FreeCAD work,
dug down to the issue-tracker level,
so that if this project ever grows a mechanical-CAD module
(or simply wants to contribute upstream while validating its own architecture
against a live, messy, real domain), it knows exactly where the unclaimed work is.

**Headline finding:** real-time simultaneous editing of a FreeCAD document is a dead end nobody is actively pursuing.
But *asynchronous, federated, multi-party PDM/PLM collaboration*
the same problem `possible_agile_manufacturing.md` frames as "party breadth" —
is a **live, actively debated, currently unowned problem** inside the FreeCAD ecosystem right now,
with a core-team-flagged GSoC-sized gap that went unclaimed in 2026,
three independent and non-interoperating third-party attempts to fill it,
and a governance conversation (models vs. items vs. documents, versioned vs. unversioned annotations) that is,
point for point, the conversation this project's core-vocabulary thesis was written to resolve.

## 1. Real-time editing: confirmed dead, not just quiet

- **CollaborativeFC / Open Collaboration Platform (OCP)** —
the one serious real-time attempt (Stefan Tröger, longtime FreeCAD core contributor),
a custom Go P2P framework with its own "Datastructure Markup Language" rather than an established CRDT library.
Never left alpha; concurrent editing was documented as unreliable by its own author.
**Archived May 2026**, no successor project claims its scope.
[github.com/OpenCollaborationPlatform/CollaborativeFC](https://github.com/OpenCollaborationPlatform/CollaborativeFC)

- **FreeCAD's own 2026 roadmap** (OpenCascade partnership, Wayland/Qt, possible Vulkan rendering)
contains no real-time collaboration line item.
**OCCT**, the geometry kernel, has no shared-document concept at all — confirming (as this project's own architecture already assumes)
that collaboration has to be built entirely above the kernel, never inherited from it.

- **JupyterCAD** (QuantStack) is the only project that achieved real-time multi-user CRDT sync for CAD-like data —
but it does this by *not* touching FreeCAD's live document at all.
It defines its own document format (`.jcad`) as Yjs shared types, and treats `.FCStd` as an import/export target via a separate adapter plugin.
This is real-time collaboration on a parallel, purpose-built representation, not on FreeCAD itself —
structurally identical to this project's own "adapter canonicalizes into the shared representation" pattern.

**Conclusion:** if this project ever wants live simultaneous mechanical-CAD editing,
JupyterCAD's pattern (a CRDT-native canonical doc + a FreeCAD import/export adapter, never editing `.FCStd` in place) is the only proven approach,
and even that hasn't attempted parametric-feature-level conflict resolution —
only whole-shape/tree-level sync. Nobody has solved "two people edit the same sketch's constraints simultaneously." That remains genuinely open.

## 2. What's actually a live work in progress

### JupyterCAD core — active, but its FreeCAD bridge is the weak link
- Main repo: commits almost daily through September 2026 (e.g. `Publish 3.4.3`, annotation/profile-picture UI work, `jupyter-collaboration` version bumps) — this is a healthy, funded (QuantStack), actively maintained project.
- The **`jupytercad-freecad` adapter plugin is comparatively neglected**: last release March 2026, only 9 stars, 13 open issues, several unresolved for a year+:
  - [#22 "Update to support FreeCAD v1"](https://github.com/jupytercad/jupytercad-freecad/issues/22) — open since Nov 2024, meaning the bridge may still target pre-1.0 FreeCAD semantics.
  - [#26 "Reduce the amount of data we're reading from FCstd files"](https://github.com/jupytercad/jupytercad-freecad/issues/26) — the loader currently reads *all* FreeCAD object properties even though the JupyterCAD client only understands a subset; a schema-driven fix was proposed but not done.
  - [#34 "Cannot make suggestions on freecad objects"](https://github.com/jupytercad/jupytercad-freecad/issues/34) and [#35 "Annotations break when exporting FCStd to jcad"](https://github.com/jupytercad/jupytercad-freecad/issues/35) — the collaboration features (comments/annotations) that work fine on native `.jcad` objects **do not survive the FreeCAD round-trip**, i.e., collaboration and FreeCAD-interop are currently second-class citizens relative to each other.
- **Read:** the core CRDT/collaboration engineering is solid and moving fast; the FreeCAD-specific half of the bridge is under-resourced and stalled on exactly the kind of schema-mapping problem this project's semantic-layer thinking is built for.

### FreeCAD core: collaboration is an acknowledged, unclaimed gap
Three linked, still-open core issues, all opened by the same design-discussion thread (Nov–Dec 2025, ongoing into 2026):
- [**#25681 "Core: Collaboration features"**](https://github.com/FreeCAD/FreeCAD/issues/25681) — the overarching issue. Explicitly proposed as a **GSoC project description** ("Researching currently available PLM/PDM solutions... Define what would be missing features and how they could be implemented... Define a plan of an ideal PLM/PDM system for FreeCAD"), scoped as 350h/medium-hard. **Checked against the [GSoC 2026 announced-projects list](https://blog.freecad.org/2026/05/02/gsoc2026-projects-announced/) — it was not selected.** The two 2026 slots went to multibody-dynamics visualization and TechDraw annotation workflow. This PDM/collaboration design work remains **explicitly identified by the core team as needed, and currently has no owner.**
- [**#25682 "Core: Improve annotations as a collaboration feature"**](https://github.com/FreeCAD/FreeCAD/issues/25682) — active discussion (last comment Jan 2026) about whether `App::AnnotationLabel` should become the collaboration-comment primitive, explicitly framed against the Lens server.
- [**#25685 "Core: Add conversations to annotations"**](https://github.com/FreeCAD/FreeCAD/issues/25685) — extends #25682 to threaded comments.

### Ondsel Lens / `FreeCAD/Ondsel-Server` — alive under community stewardship, mid-identity-crisis
After Ondsel (the company) shut down in November 2024, the FreeCAD Project Association absorbed the codebase. It is genuinely active: **pushed August 2026, 70 stars, 20 open issues**, Node.js/MongoDB/Vue stack, AGPL-3.0-or-later, now self-hostable via docker-compose with OIDC/Keycloak support (a real improvement over the old AWS-locked deployment). Two open issues matter a lot here:
- [**#48 "Extent to which Lens is a PDM"**](https://github.com/FreeCAD/Ondsel-Server/issues/48) — a live design debate (pierreporte, Creymore, pieterhijma) about the exact problem `possible_agile_manufacturing.md` already has an answer for: Lens currently only versions *files*, but a real PDM needs a **model / part / assembly / item / document** distinction with **links** between them — i.e., they are independently re-deriving a part-of graph with typed relationships, in real time, right now, without a stated schema philosophy.
- [**#47 "Feature: Synchronize between Servers / Sync tool Library"**](https://github.com/FreeCAD/Ondsel-Server/issues/47) — a from-scratch proposal for **peer-to-peer sync between independent Lens server instances** ("Migration of a Makerspace Hosted or FPA Server to a Personal or the other way around," couple/one-way/one-time sync modes). This is the "federation of graphs, not one graph" problem from `possible_agile_manufacturing.md` §Proposed architecture principles, being proposed **without any CRDT, content-addressing, or consensus-model vocabulary** — it's currently scoped as an ad hoc bespoke sync protocol.

### Two more independent, non-interoperating PDM attempts
Neither talks to Lens, JupyterCAD, or each other:
- **[FreePDM](https://github.com/grd/FreePDM)** (Go, MIT) — pushed Oct 2025, 81 stars, only 3 open issues (small, quiet, but not abandoned).
- **[nanoPLM](https://github.com/alekssadowski95/nanoPLM)** (MIT) — pushed July 2025, 72 stars, 48 open issues (larger scope, more churn, more unresolved).

**Read on this cluster:** the FreeCAD ecosystem currently has **at least three separately-invented, non-interoperating attempts at "PDM for FreeCAD"** (Lens, FreePDM, nanoPLM), plus a core-team design conversation independently re-deriving graph/provenance concepts, plus a GSoC-flagged research gap nobody claimed. This is a textbook instance of the "no single authority should own the schema" fragmentation problem this project's README already names as its founding rationale — just discovered natively, from the outside, in a domain this project hasn't touched yet.

## 3. High-value targets for contribution

Ranked by leverage (impact on the ecosystem vs. effort), not by ease:

1. **Bring the core-vocabulary thinking to Ondsel-Server issue #48.** This is the single highest-leverage, lowest-cost move available: a comment on a live, unresolved, multi-stakeholder design thread proposing exactly the model/part-of/link ontology this project's core vocabulary already specifies (part-of, connected-to, satisfies, verified-by, supersedes), and pointing at IEC 81346 for the identity scheme they're implicitly missing. Costs an afternoon, could shape a real system's schema before it calcifies into MongoDB collections that are expensive to change later.
2. **Pick up the unclaimed #25681 GSoC-scoped research work** ("survey existing PLM/PDM solutions, define missing features, define an ideal PLM/PDM plan for FreeCAD") as an actual deliverable — this project's `possible_agile_manufacturing.md` and `possible_data_architecture.md` are most of this write-up already done from first principles; reshaping them as a direct answer to #25681, informed by what Lens/FreePDM/nanoPLM actually got wrong, would be a substantive, citable contribution with an already-identified, receptive audience (pieterhijma and the design-working-group are visibly asking for exactly this).
3. **Fix the jupytercad-freecad schema-mapping issues (#26, #34, #35).** Concrete, scoped, technically approachable (Python, reading FreeCAD's `App::Document` object model against jupytercad-core's schema), and directly exercises this project's central claim — that legacy tool formats become schema-driven adapters into a canonical representation — against a real, already-mostly-working system instead of a hypothetical one. Good way to pressure-test the semantic-layer design cheaply before committing to it for a from-scratch module.
4. **Respond to #47 (server-to-server sync) with the durable-history-layer research.** Their proposed design (bespoke "synchronization link," couple/one-way/one-time modes) is solving, without knowing it, the exact problem `possible_data_architecture.md` researched in depth (content-addressed history, Automerge/Fluree/Dolt tradeoffs, CAP positions). A well-aimed comment linking that research could save them from building an ad hoc protocol that has already been shown not to generalize (this is literally why CollaborativeFC's bespoke P2P layer failed).
5. **Weigh in on FreeCAD core #25682/#25685 (annotations-as-collaboration).** Lower leverage than #48/#25681 — it's UI-scoped, not architecture-scoped — but worth noting because JupyterCAD *already has a working, real-time, multi-user annotation/commenting system* (right down to profile pictures in the annotation panel, shipped September 2026) that the FreeCAD core discussion doesn't appear to reference. Flagging that existing, working implementation to the core-issue participants is a nearly-free way to prevent duplicated design effort across two projects in the same ecosystem.

## 4. Most likely integration paths into this project

If a mechanical-CAD module is ever added to the federation this project is building:

- **FreeCAD is a submodel/module producer, never the core.** Consistent with `possible_agile_manufacturing.md`'s architecture principles: a FreeCAD-authored geometry model becomes one AAS-style submodel (or OML description) that plugs into the thin core vocabulary via IEC-81346-style stable identifiers — exactly the role already sketched for the QET electrical-schematic work, just for mechanical geometry instead.
- **The canonicalization adapter is the actual integration point, and JupyterCAD's `jupytercad-freecad` plugin is the closest existing reference implementation** for what that adapter needs to do (read `.FCStd`'s `App::Document` object graph, map its properties into a schema-constrained canonical form, round-trip cleanly) — including, usefully, a working list of what's still hard about it (issues #22/#26/#34/#35 above).
- **Ondsel-Server / Lens is the most plausible near-term collaboration partner, not competitor.** It's community-owned (under the `FreeCAD` GitHub org, not a company), AGPL (compatible license family with this project's own Fluree-adjacent licensing thinking), self-hostable, and — per issue #48 — actively looking for exactly the modeling philosophy this project already has. A realistic path is: this project's core ontology becomes the schema Lens's "item/model/link" redesign adopts, rather than two projects inventing separate, incompatible answers to the same question.
- **FreeCAD core's annotation work (#25682/#25685) is a plausible place to prototype the "statements carry provenance" idea from Wikidata's data model** (per `possible_agile_manufacturing.md` §Solving party breadth) at small scale — a collaboration annotation with an author, timestamp, and versioning status is a miniature, already-motivated test case for the statement/qualifier/reference/rank model before attempting it at full graph scale.

## 5. Entry points that don't require real-time editing at all

Since real-time simultaneous editing is confirmed dead-end territory here, the actually-live collaboration surface in this ecosystem is asynchronous and PDM-shaped — which maps cleanly onto this project's own "continuous, not phase-gated" and "git-like verifiable history" goals rather than its CRDT-real-time goal:

- **Async, git-like versioning is already where FreeCAD's own community is converging** (History Workbench, shipped Sept 2026, wraps git with CAD-friendly terms and 3D/tree diffing) — this is a much closer near-term fit to this project's durable-history-layer research (content-addressed, immutable, diffable states) than to its live-CRDT-layer research, and a natural place to demonstrate the "every accepted state is content-addressed and immutable" principle concretely, in a tool people already use, without needing to solve live concurrent geometry edits at all.
- **PDM federation (issue #47) is a direct instance of this project's "federation of graphs, not one graph" principle**, playable today: Lens instances syncing peer-to-peer is precisely the "each party owns and publishes their own subgraph" pattern, just currently missing the theory (consensus model, conflict semantics, CAP position) that would make it robust rather than ad hoc.
- **Standards-compliance-as-continuous-check (this project's "Continuous, not phase-gated workflow" goal) has a ready-made proving ground in the annotation/conversation work** (#25682/#25685): a comment thread with a "resolved/unresolved" status attached to a specific geometry feature is a small, concrete instance of "acceptance cascades to invalidate exactly the downstream sign-offs that depended on what changed" — worth prototyping there before attempting it at whole-factory scale.

## Sources

- [CollaborativeFC — GitHub (archived May 2026)](https://github.com/OpenCollaborationPlatform/CollaborativeFC)
- [OCP (Open Collaboration Platform) core — GitHub](https://github.com/OpenCollaborationPlatform/OCP)
- [Lens Platform Now Available — FreeCAD News](https://blog.freecad.org/2025/05/19/lens-platform-now-available-collaborate-with-freecad-online/)
- [The End Of Ondsel And Reflecting On The Commercial Prospects For FreeCAD — Hackaday](https://hackaday.com/2024/11/12/the-end-of-ondsel-and-reflecting-on-the-commercial-prospects-for-freecad/)
- [FreeCAD/Ondsel-Server — GitHub](https://github.com/FreeCAD/Ondsel-Server)
- [Ondsel-Server issue #47 — server-to-server sync proposal](https://github.com/FreeCAD/Ondsel-Server/issues/47)
- [Ondsel-Server issue #48 — extent to which Lens is a PDM](https://github.com/FreeCAD/Ondsel-Server/issues/48)
- [JupyterCAD — GitHub](https://github.com/jupytercad/JupyterCAD)
- [jupytercad/jupytercad-freecad — GitHub](https://github.com/jupytercad/jupytercad-freecad)
- [jupytercad-freecad issue #22 — FreeCAD v1 support](https://github.com/jupytercad/jupytercad-freecad/issues/22)
- [jupytercad-freecad issue #26 — reduce FCStd data read](https://github.com/jupytercad/jupytercad-freecad/issues/26)
- [jupytercad-freecad issue #34 — suggestions on FreeCAD objects](https://github.com/jupytercad/jupytercad-freecad/issues/34)
- [jupytercad-freecad issue #35 — annotations break on export](https://github.com/jupytercad/jupytercad-freecad/issues/35)
- [FreeCAD/FreeCAD issue #25681 — Core: Collaboration features (GSoC-scoped, unclaimed in 2026)](https://github.com/FreeCAD/FreeCAD/issues/25681)
- [FreeCAD/FreeCAD issue #25682 — Core: Improve annotations as a collaboration feature](https://github.com/FreeCAD/FreeCAD/issues/25682)
- [FreeCAD/FreeCAD issue #25685 — Core: Add conversations to annotations](https://github.com/FreeCAD/FreeCAD/issues/25685)
- [GSoC2026 projects announced — FreeCAD News](https://blog.freecad.org/2026/05/02/gsoc2026-projects-announced/)
- [History Workbench brings Git powered file versioning to FreeCAD — Adafruit Blog](https://blog.adafruit.com/2026/09/01/history-workbench-brings-git-powered-file-versioning-to-freecad/)
- [FreePDM — GitHub](https://github.com/grd/FreePDM)
- [nanoPLM — GitHub](https://github.com/alekssadowski95/nanoPLM)
- [2025 annual report and plans for 2026 — FreeCAD News](https://blog.freecad.org/2026/04/24/2025-annual-report-and-plans-for-2026/)
