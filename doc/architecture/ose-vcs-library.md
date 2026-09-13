# Architecture notes — OpenSourceEcology/vcs-library

Read against `doc/technical_convergence_plan.md`'s 8-field template.
Source read: `README.md`, `LIBRARY_ONTOLOGY.md`, `GOVERNANCE.md`,
`LICENSE.md`, one full entry (`library/parts/sloped_rafter_raked/{meta.yaml,
schema.py,compiler.py,expect.yaml}`), `collections/gvcs/README.md`,
`pyproject.toml`, `.github/workflows/*.yml` (names only). No git commands
were run for this pass.

This project is architecturally unlike every other repo read so far in
this series (EasyPDM, the Omniverse connector, Ondsel-Server) — it has
**no server, no database, and no binary CAD file checked into version
control at all**. It's a plain git repo of Python + YAML text, where the
CAD geometry itself is a *build artifact* regenerated on demand. Worth
treating as a structurally distinct fourth pattern in this census, not a
minor variant of the other three.

## 1. Repo & basic facts

- **Language(s)**: Python (~11,400 lines across `libtools/`, `tests/`,
  and library entries, excluding the raw `upstream/` source dump) +
  YAML (`meta.yaml`/`expect.yaml` per entry) — no JavaScript, no
  database driver, no web framework anywhere in the stack.
- **Size**: small but structurally dense — 12 active/wip library entries
  (per the README's own contents table: 2 parts, 6 modules, 4 assemblies,
  0 structures) plus a separate `collections/gvcs/` sub-library (8
  entries: sourced/reverse-engineered OSE machine geometry — Universal
  Axis, Power Cube, CEB Press — see §7).
- **Maturity/activity**: `doc/ecosystem/graph.yaml` records
  `first_seen: 2026-09-07`, `status_as_of: 2026-09-07` — this is the
  **newest** project read in this series by a wide margin (weeks old,
  not the months-to-years of the others), seeded directly from a named
  individual's (Catarina's) OSE wiki design log (`Catarina_Log`,
  2026-07-08 entry) rather than growing organically as a repo.
- **License**: `doc/ecosystem/graph.yaml` currently records `license:
  unknown` — **this is stale**; `LICENSE.md` states content is
  CC-BY-SA-4.0 "unless otherwise noted," under the umbrella of the OSE
  License for open-source hardware documentation, with per-entry
  authorship/provenance recorded in each entry's own `meta.yaml`. See §6.
- **CI**: four GitHub Actions workflows
  (`validate.yml`/`gvcs-validate.yml`/`output-validate.yml`/`pages.yml`)
  — code validation (Tier 1, no FreeCAD needed) runs on every push/PR;
  output/geometry validation (Tier 2, needs `freecadcmd`) runs when
  library/toolkit files change. A real, enforced two-tier CI gate, not
  just a build check.
- **Governance is explicitly programmatic, not just descriptive**:
  `GOVERNANCE.md` frames this as "one library in a larger pattern: many
  small, owned libraries rather than one monolith" and explicitly invites
  forking into sibling libraries reusing the same `libtools` package for
  entirely different domains (tractors, 3D printers, vehicles) — the
  toolkit is deliberately domain-generic, only the seeded content is
  housing-specific.

## 2. Identity/versioning model

**Identity is the directory name, and versioning is git itself — nothing
custom.** This is the most radical simplification of the "identity"
question in this whole series:

- An entry's ID *is* its directory path segment under
  `library/{parts,modules,assemblies,structures}/<id>/`, and `meta.yaml`
  must restate that same `id` (checked, per `LIBRARY_ONTOLOGY.md`
  §"Entry Contract") — a consistency check, not a separate identity
  layer. **"Renames are breaking"** is stated outright: ids are baked
  into command-line selection, report filenames, and generated artifact
  names, so there is no rename-safe indirection (no UUID, no numeric
  sequence) anywhere in the design.
- **There is no separate "file version" concept at all** — because there
  is no binary file to version. `meta.yaml.version` is a bare
  human-assigned semver-like string (`0.1.0` in the one entry read),
  advisory only; nothing in the validator contract (`LIBRARY_ONTOLOGY.md`
  §"Validator Contract") reads or enforces it. The actual version history
  of an entry is just **git's own commit history on that directory** —
  the project deliberately doesn't reinvent anything on top of git,
  unlike EasyPDM (filename convention) or Ondsel-Server (embedded
  versions array), because git already does this job for plain text.
- `meta.yaml.status` (`active` | `wip`) is the closest thing to a
  lifecycle/status field (cf. EasyPDM's `w_pracy`/`sprawdzany`/`wydany`
  state machine) — but it's a single flat flag with exactly one
  consequence: a `wip` entry's validation failures are report-only and
  don't fail CI (`LIBRARY_ONTOLOGY.md` §"Validator Contract", exit-code
  rules), whereas an `active` entry's failure is a hard CI failure. No
  released/cancelled/review states, no owner-lock, no revision-letter
  convention.
- **Provenance is a first-class, structured field**
  (`meta.yaml.provenance`: `author`, `source`, `original_filename`,
  `drive_file_id`), explicitly required by the `meta_complete` check —
  every other project in this census treats "who created this and where
  did it come from" as an afterthought (EasyPDM: an audit-trail history
  list; the others: implicit via commit authorship); this project
  requires it as a *validated schema field on every entry*.

## 3. Conflict/concurrency strategy

**None beyond git itself, and this is explicit, not accidental.**
`GOVERNANCE.md` §"Ownership": "The author of an entry is its owner and
maintainer... Design changes to an existing entry go through that
owner" — this is a **social/process convention** (route changes through
the named owner) with **zero technical enforcement** anywhere in the
validator contract or CI. No lock field, no check-out state, no
merge-conflict-avoidance mechanism of any kind — concurrent edits to the
same entry are handled exactly the way concurrent edits to any git repo
are handled: a merge conflict at the text level, resolved by whoever
does the merge.

This is a meaningful data point for the plan's Cluster 2 (concurrency)
question: it demonstrates that **when the underlying artifact is plain
text (Python schema + compiler code) rather than an opaque binary CAD
file, the entire "lock, don't merge" problem this cluster keeps
reinventing simply does not arise** — ordinary git merge (even if
occasionally requiring manual conflict resolution) is sufficient,
because there's no binary-diff problem to route around. This is the
strongest naturally-occurring counter-example in the census so far to
the assumption (implicit in EasyPDM/Anchorpoint/the Omniverse connector)
that CAD collaboration inherently needs a locking layer — it doesn't, if
the source representation is kept in a git-mergeable form and the binary
is regenerated, not stored.

## 4. File format / serialization touchpoints

- **The FCStd (or STEP/etc.) file is never committed** — it's a build
  product of running `compiler.py`'s `compile(schema, doc)` against
  `schema.py`'s `SCHEMA` dict through FreeCAD (`freecadcmd`), and
  `.gitignore` explicitly excludes `*.FCStd`/`*.FCStd1`/`out/`/`reports/`/
  `exported/`/`slots/`. This is the inverse of every other project in
  this census, which all treat the CAD file as the thing being versioned
  — here, the *parametric recipe* (schema + compiler, both plain Python)
  is the versioned artifact, and the CAD file is disposable, regenerable
  output, like a compiled binary in a normal software repo.
- **A deliberately restricted "data-only" schema format**: `schema.py`
  must contain exactly one `SCHEMA = {...}` dict-literal assignment and
  nothing else executable (`schema_data_only` check, `libtools/schema_check.py`)
  — no imports, functions, loops, conditionals, comprehensions, f-strings,
  etc. This is enforced by actually parsing the AST and loading the
  module with an **empty `__builtins__` namespace** (`LIBRARY_ONTOLOGY.md`
  §"Schema Discipline") — a real sandboxing measure against
  arbitrary-code-execution-as-schema, notable given schema.py files are
  contributed content from potentially many small library owners.
- **Units discipline is schema-enforced, not just documented**: every
  dimensional numeric key must carry an `_in`/`_in3` suffix
  (`unit_suffix_discipline` check, with a fixed list of dimension-word
  substrings and a short exemption list) — compilers then convert
  inches→mm internally (`IN = 25.4`) before creating FreeCAD geometry, and
  reports convert back. A small but concrete convention directly
  addressing a real, cross-project unit-ambiguity failure mode.
- **Output geometry validation is itself substantial**: Tier 2 checks
  (`compiles`, `completeness` — valid/closed/positive-volume solids plus
  role/count rules via `fnmatch` patterns, `overlap` — common-volume
  intersection detection with an explicit allow-list of expected
  contacts, `fit` — bounding-box envelope checks with tolerance) amount
  to a real automated geometry QA pipeline running in CI against actual
  FreeCAD-computed shapes, not just schema shape-checking. None of the
  other three projects in this census have anything comparable — EasyPDM
  and the others manage metadata about files, this project checks the
  actual geometry the parametric code produces.
- **Web export path** (`bake-web`) tessellates compiled solids (0.5 mm
  deflection) into flat vertex/triangle JSON plus normalized BREP, keyed
  into a `catalog.json` that is deleted before compilation and rewritten
  only on full success — "a failed rebuild cannot leave a stale catalog"
  (README) — a small but real atomicity guarantee for a generated,
  publishable artifact.

## 5. Dependencies & integration points

- **FreeCAD via `freecadcmd`** (the CLI/headless FreeCAD binary), invoked
  as an external process, not a Python package import inside a
  long-running service — Tier 1 validation and `schema_check`/registry
  loading work with *no* FreeCAD dependency at all; only Tier 2 output
  validation, `compile`, and `bake-web` need it, and its absence degrades
  gracefully (checked via `FREECADCMD` env var / `PATH`, exit code 2 if
  missing) rather than crashing opaquely.
- **No server, no database, no auth, no network service of any kind.**
  The entire "integration surface" is a CLI (`libtools/cli.py`) and a
  static, publishable output directory (`bake-web`'s `web-dist`) — a
  radically smaller dependency footprint than any other project in this
  census.
- **Packaging**: a normal Python package (`pyproject.toml`, `pip install
  -e '.[dev]'`), tested with `pytest` (10 test files under `tests/`
  covering the CLI, registry, each validator, BOM/fab-drawing/web-bake
  generation).
- **`collections/` is an explicit extension point** for independent,
  differently-scoped design families sharing the same entry contract but
  outside the default `--root .` discovery scope — `collections/gvcs/`
  (see §7) is the first instance, itself declaring its own `toolchain.json`
  and running its own CI job (`gvcs-validate.yml`) separate from the
  housing library's.

## 6. Graph cross-reference

`doc/ecosystem/graph.yaml`, node `project:ose-vcs-library` (line 1147):
category `[pdm, bom]`, `technical_approach: [other]`, `scope:
multi-cad-platform`, origin US/EN, `license: unknown`, `first_seen:
2026-09-07`. Existing note already correctly identifies the
parts→modules→assemblies→structures hierarchy and its
`independently_reinvents` edge against Ondsel-Server's own
part/assembly/sub-assembly vocabulary (issue #48) — confirmed accurate
by this reading (§7 below adds detail rather than revising it).

**Correction applied**: `license: unknown` → `CC-BY-SA-4.0` (content) per
`LICENSE.md`, which cites the OSE License for hardware documentation with
CC-BY-SA-4.0 as the default for unmarked content; code license (`libtools/`)
is not separately marked and appears to fall under the same umbrella —
worth a follow-up check of `pyproject.toml` classifiers if a
code-specific license distinction ever matters. Applied directly to
`graph.yaml` in this pass (see below).

## 7. Friction points observed firsthand

- The existing `independently_reinvents` edge (parts→modules→assemblies
  hierarchy vs. Ondsel-Server's part/assembly vocabulary) is confirmed,
  but this reading surfaces a **more interesting, unflagged
  independent-reinvention**: this project's git-native,
  compile-from-source-not-store-the-binary approach is structurally the
  same idea FreeCAD's own community periodically proposes (storing a
  parametric "recipe" instead of a binary FCStd for version-control
  friendliness — the exact problem space `doc/possible_freecad_collaboration.md`
  and `doc/optimistic_locking_research.md` survey from the discussion/
  academic-literature side). This project is a **working, shipped
  instance** of that idea, not just a proposal — worth flagging as
  directly relevant prior art for any future `possible_freecad_collaboration.md`-style
  synthesis, and as a candidate new node/edge in `graph.yaml` (a
  `cites_as_prior_art`-style connection from this project to that
  discussion thread, if one doesn't already exist — not confirmed either
  way in this pass).
- The `collections/gvcs/` sub-library is a genuinely different friction
  case worth separating from the main library's: it holds **"fixed
  source geometry"** — real, but *reverse-engineered/reconstructed*
  OSE machine designs (Universal Axis, Power Cube, CEB Press) where the
  original CAD is not available to convert into the schema/compiler
  pattern, so most entries are geometry captured as-is rather than
  regenerated from a parametric recipe (only one of eight,
  `axis_idler_spacer`, is truly parametric). This is the same
  "how do you fold in someone else's undocumented legacy design"
  problem every PDM-adjacent project in this census eventually hits
  (cf. EasyPDM's "attach an existing file," Ondsel-Server's plain file
  upload) — but here it's handled by explicitly labeling the entry
  `Kind: Fixed source part/assembly` rather than pretending it's
  parametric, a small but honest convention worth citing.
- The `schema_data_only` sandboxing (empty `__builtins__`, AST-level
  rejection of executable constructs) is a real, load-bearing security
  measure for a project explicitly designed to accept contributions from
  many small, independently-owned "sibling libraries" (per
  `GOVERNANCE.md`) — none of the other three projects in this census
  need anything like it, because none of them accept executable
  contributor content as a first-class citizen of the data model the way
  this one does.

## 8. Minimal-patch hypothesis

- **Not a candidate for the "lock, don't merge" shim at all** (see §3) —
  this project's entire architecture is evidence *against* needing one,
  not a participant in that friction point. Any patch proposal targeting
  Cluster 2 should probably cite this project as a counter-example/
  existence-proof rather than a target for the locking convention itself.
- **A genuinely portable idea worth extracting for the plan's broader
  synthesis** (not a tiny patch, but flagged since field 8 is meant to
  catch this): the **schema/compiler split with a sandboxed, data-only
  schema format and a signature-checked compiler contract**
  (`compile(schema, doc)`) is a small, well-specified pattern that any
  other FreeCAD-adjacent project storing parametric definitions could
  adopt independently of this project's specific OSE housing content —
  cost: **small to moderate** per adopter (define an equivalent
  AST-restriction check and a fixed compiler function signature), but
  the payoff (git-mergeable source of truth, disposable binary output)
  is exactly the thing Cluster 2's other members are missing.
- **Provenance-as-required-schema-field** (`provenance.author`,
  `provenance.source`, checked by `meta_complete`) is a trivial-cost,
  directly transferable convention: any project in this census could add
  a required "where did this come from" block to its own metadata schema
  with no architecture change — cheaper than either the locking or the
  versioning conventions proposed in the other architecture notes, and
  addresses a real, separate friction point (informal/undocumented
  provenance) rather than concurrency or identity.
