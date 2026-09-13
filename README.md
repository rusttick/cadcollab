# cadcollab

Research into open source collaborative CAD possibilities — a tangent off [`co-edit-stream`](../co-edit-stream), which is building a standards-based framework for live, collaborative, cryptographically-verifiable digital twins. This repo asks: where does that framework's thinking already apply to the existing FreeCAD ecosystem, and where's the highest-leverage place to contribute?

# Goals

- **Map the real state of collaborative FreeCAD work**, down to the issue-tracker level, rather than relying on surface-level claims about what tools support "collaboration."
- **Distinguish live efforts from stalled/dead ones**, so effort isn't spent re-treading abandoned ground (e.g. real-time simultaneous editing of a FreeCAD document is a confirmed dead end — CollaborativeFC/OCP never left alpha and archived in 2026).
- **Find unclaimed, maintainer-endorsed gaps** — such as FreeCAD core issue #25681, a GSoC-scoped "Collaboration features" research project that went unassigned in the 2026 GSoC round — where a contribution would land on receptive ground instead of duplicating work already underway.
- **Identify where this ecosystem is independently re-deriving problems `co-edit-stream` already has answers for** — e.g. Ondsel Lens's live debate over a model/part/assembly/item/link ontology (issue #48) mirrors the core vocabulary (part-of, connected-to, satisfies, verified-by, supersedes) already proposed there, and its from-scratch server-to-server sync proposal (issue #47) mirrors the "federation of graphs, not one graph" and content-addressed history research already done there.
- **Favor async, federated, PDM-shaped collaboration over real-time editing** as the actually-live surface in this ecosystem — closer to `co-edit-stream`'s git-like verifiable history and federation goals than to its CRDT-real-time goal.
- **Turn findings into ranked, concrete contribution targets** (from a single well-aimed issue comment up to picking up an entire unclaimed research scope), not just a survey for its own sake.

# Research

- [`possible_freecad_collaboration.md`](possible_freecad_collaboration.md) — survey of FreeCAD collaboration efforts (CollaborativeFC, Ondsel Lens/Ondsel-Server, JupyterCAD, FreePDM, nanoPLM, FreeCAD core annotation/PDM discussions), what's live vs. stalled, ranked high-value contribution targets, and the most likely integration paths back into `co-edit-stream`.
