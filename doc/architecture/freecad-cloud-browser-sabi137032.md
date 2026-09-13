# Architecture notes — freecad-cloud-browser (`sabi137032/freecad-cloud-browser`)

**This is a security finding, not a routine architecture note.** Read
against `doc/technical_convergence_plan.md`'s 8-field template as far as
it applies, but the primary content of this document is evidence that
this repository is very likely a **malware-distribution decoy**
impersonating a legitimate FreeCAD addon, not an honest independent
reinvention of `hitclawagent/freecad-cloud-browser` as
`doc/ecosystem/graph.yaml` currently characterizes it. No files from
this repository were extracted, executed, or opened beyond listing a
zip archive's table of contents (`unzip -l`, which does not extract or
run anything) and reading plain-text source files. No git commands were
run for this pass.

## Summary of evidence

1. **The README describes features the code does not implement.** It
   gives detailed, step-by-step instructions for connecting Google
   Drive ("Click the Login button... your web browser will open... Sign
   in with your Google account...") and Dropbox, and lists both in its
   "Features Summary" as supported "major cloud platforms." The actual
   provider registry (`providers/__init__.py`) only ever registers
   three providers: `s3`, `ftp`, `webdav`
   (`_try_register("s3", ...)`/`_try_register("ftp", ...)`/
   `_try_register("webdav", ...)`). **There is no Google Drive or
   Dropbox provider file anywhere in the repository** — a repo-wide
   case-insensitive search for "drive"/"dropbox"/"onedrive" turns up
   only comments, placeholder strings, and test fixtures using
   `"google_drive"`/`"dropbox"` as arbitrary string labels
   (`config_store.py`, `test_config_store.py`, `test_file_cache.py`),
   never an actual OAuth flow or API client. The README's entire
   "Connecting Google Drive" / "Connecting Dropbox" sections describe a
   product that does not exist in this codebase.
2. **The `LICENSE` file was never updated for this repo** — it still
   reads "MIT License, Copyright (c) 2026 **hitclawagent**," the other
   (unrelated, per `doc/ecosystem/graph.yaml`'s current framing) author
   from the sibling project. This directly contradicts the graph's
   existing note that there is "no visible relationship between the
   authors" — the presence of hitclawagent's own copyright line in this
   repo's LICENSE is a visible relationship: this codebase was very
   likely copied (fully or substantially) from that project as a
   plausible-looking shell, not independently written from scratch. The
   Python source in `core/`/`providers/`/`ui/` is structurally
   near-identical to the hitclawagent repo's own layout and content in
   every module name checked in this pass.
3. **The download mechanism does not point at a GitHub Release.** The
   README's prominent "Download Here" shield badge links directly to
   `raw.githubusercontent.com/sabi137032/freecad-cloud-browser/main/ui/browser-freecad-cloud-v1.9.zip`
   — a file checked directly into the repository's `ui/` folder, not an
   Addon-Manager-indexed release artifact. Nothing about a FreeCAD
   addon's normal distribution path requires or expects a `.zip`
   sitting inside a `ui/` source folder to be the primary advertised
   download link.
4. **That zip's actual contents have nothing to do with FreeCAD.**
   Listing it (`unzip -l`, table-of-contents only, nothing extracted)
   shows exactly four files: `Application.bat`, `lsp.txt` (309,446
   bytes — implausibly large for a plain text file, a common pattern
   for an encoded/obfuscated payload disguised with an innocuous
   extension), `lua51.dll` (a legitimate Lua 5.1 runtime DLL, but one
   whose presence alongside an unrelated `.exe`/`.bat` pair is a
   recognized DLL-side-loading malware-delivery pattern — a real Lua
   interpreter has no plausible role in a Python/PySide FreeCAD addon),
   and `util64.exe`. None of these are Python source, a FreeCAD
   addon manifest, or anything resembling the actual repository's own
   `core`/`providers`/`ui` Python modules. **This zip is not a build of
   the addon described in the README or visible in the repository's own
   source tree.**
5. **The README's tone and specificity are inconsistent with the rest
   of the repo.** Heavy use of emoji section headers, a "System
   Requirements: Windows 10 or Windows 11" line for what the actual
   Python source shows is a cross-platform FreeCAD addon (the code
   itself has no Windows-specific logic beyond what any keyring-based
   credential store would), and marketing-style phrasing ("bridges the
   gap," "no longer need to manage manual downloads") are a marked
   departure in register from the plain, technical README style seen
   across every other project in this census — including the sibling
   hitclawagent repo this one's source appears to be based on.

## What this means for the census

Taken together, the pattern here matches a well-documented GitHub abuse
technique: clone a real, legitimate-looking open-source project (here,
almost certainly `hitclawagent/freecad-cloud-browser` itself, given the
copied LICENSE and near-identical source layout), embellish its README
with appealing but fictional features to attract search traffic and
clicks, and replace or accompany the real download path with a link to
an unrelated executable payload. The Python source code visible in this
repo may be entirely legitimate (copied) FreeCAD-addon code — the
malicious component, if this assessment is correct, is the zip file and
the README's redirection toward it, not necessarily the Python modules
themselves. This assessment was not independently confirmed by
detonating the payload (appropriately out of scope and unsafe to do),
antivirus-scanning it, or checking it against a threat-intelligence
database — it is a source-level judgment based on the five points
above, not a certified malware determination.

**Recommendation**: do not download, extract, or run
`ui/browser-freecad-cloud-v1.9.zip` or anything inside it
(`Application.bat`, `util64.exe`) on any machine. If this repository is
still live on GitHub, reporting it via GitHub's abuse-reporting flow
would be appropriate, though that action is outside this document's own
scope (this pass only analyzes the already-downloaded local copy per
the technical-convergence plan's methodology).

## Template fields, briefly

Given the above, the remaining template fields (identity/versioning
model, conflict/concurrency strategy, dependencies) are not analyzed in
the usual depth — the visible Python source appears to be a copy of
`hitclawagent/freecad-cloud-browser`'s own code (see that project's
architecture note, `doc/architecture/freecad-cloud-browser-hitclawagent.md`,
for the corresponding analysis of the shared codebase's actual
mechanics: no versioning model, unconditional-overwrite sync-on-save
with no conflict detection, keyring-then-Fernet credential storage). A
line-by-line diff between the two repos' Python source was not
performed in this pass to confirm exact identity versus a modified
copy; the module names, structure, and at least one identical string
(`config_store.py`'s placeholder account name) are consistent with a
direct copy, but this was not exhaustively verified file-by-file.

## Graph cross-reference and corrections

`doc/ecosystem/graph.yaml`, node
`project:freecad-cloud-browser-sabi137032` (originally): `status:
active`, `license: unknown`, note framing this as an independent
reinvention "3 days later, no visible relationship between the
authors." **This framing is corrected in this pass** (see the edit
applied alongside this note):

- The `independently_reinvents` edge between this node and
  `project:freecad-cloud-browser-hitclawagent` is corrected — the
  shared LICENSE attribution (§2 above) is itself a visible
  relationship, contradicting that edge's own stated rationale. Recast
  as a security-relevant relationship instead (copied/derived source
  code repurposed as a decoy), not a case of convergent, unrelated
  design.
- `status: active` is corrected to `dead` — not because the repository
  itself is confirmed removed, but because this project's own reading
  found no legitimate development activity to point to as "active";
  the label as previously applied implied a live, genuine tool, which
  this finding contradicts.
- A `security_note` field is added to the node directly, since none of
  this graph's existing note/status/technical_approach fields are
  designed to carry a security-risk finding, and burying it only in
  prose here risks it being missed by anyone scanning the graph data
  directly rather than every architecture note.
