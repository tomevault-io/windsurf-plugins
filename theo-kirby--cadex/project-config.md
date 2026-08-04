---
trigger: always_on
description: Verified against source: 2026-07-25. This file replaces the retired
---

# CLAUDE.md — Agent Entry Point

Verified against source: 2026-07-25. This file replaces the retired
`AGENTS.md` (see `docs/DECISIONS.md` ADR-005).

Cadex is an AI-native CAD app. **This repository is the whole product**
(Phase 13a, ADR-030): clone it, `pixi run setup && pixi run app`, and you
have a running application.

Two halves, one repo, separated by a process boundary rather than a
repository boundary:

- **the engine**, at the repo root — a FreeCAD fork. The AI authors
  declarative **xscript** Python programs, and `cadexd`, a per-project
  headless service speaking NDJSON over stdio, runs them in sandboxed
  `FreeCADCmd` workers that produce detached BREP, publish into an
  ephemeral document, and stream tessellation back. Five domains:
  partdesign, sketcher, part, mesh, assembly.
- **the shell**, under `shell/` — a Blender fork carrying the
  `mesh_agent` add-on. It is the product UI, it speaks the protocol in
  `docs/INTEGRATION.md`, and it ships the engine inside its own bundle.

There is no Qt shell, no provider stack, and no API-key model loop — the AI
runs as the Claude Code CLI inside the shell. `pixi run build-engine`
produces `FreeCADCmd` and `CadexGeometryWorker` and no application; the
application is what `pixi run build-shell` installs, with the engine inside
it.

**Where this is going (ADR-025, ADR-030).** The product becomes **one
application we own** — a derivative of but not dependent on either FreeCAD
or Blender. **OCCT stays** as the geometry kernel; the FreeCAD application
layer is to be replaced by our own pybind11 binding (Phase 11, engine stays
Python) and the Blender shell by our own Rust + wgpu + egui shell
(Phase 12), both behind the *unchanged* cadexd protocol. Neither is
scheduled and neither blocks anything: merging the repos moved the deadline
pressure off them, and the test-pinned protocol is what keeps them
available. What *is* live is Phase 13b — deleting from both inherited trees,
in place, under the normal removal protocol. **Do not start writing a
replacement engine or shell in this tree ahead of its phase.**

Read `docs/VISION.md` before designing anything.

## Read this first (doc index, in order)

| Doc | What it answers |
|---|---|
| `docs/VISION.md` | What the product is; principles; non-goals. **Authoritative.** |
| `docs/ARCHITECTURE.md` | What exists today: pipeline, file map, project store, substrate. |
| `docs/XSCRIPT.md` | The scripting model — today (per-domain programs) vs target (one project script). |
| `docs/ROADMAP.md` | Phases 0–13, status checkboxes, exit criteria. Living status lives here. |
| `docs/DECISIONS.md` | ADR log. Append an entry for every removal or direction change. |
| `docs/PROVENANCE.md` | Which code came from FreeCAD, from Blender, and from VibeCAD; licences, credit, and how two licences share one repo. |
| `docs/FREECAD.md` | Inherited-tree ledger for the **engine**: kept / disabled / already-deleted. |
| `docs/BLENDER-TREE.md` | The same ledger for **`shell/`**, plus the seven-file diff against upstream Blender. |
| `docs/INTEGRATION.md` | **The process contract**: the cadexd protocol (test-enforced on both requests and responses) and the engine payload. |
| `docs/BLENDER.md` | The shell: `mesh_agent`'s file map, its tools, and how to run its suites. |
| `docs/IDEAS.md` | Parking lot for uncommitted ideas. |
| `docs/cadex-release-packaging.md` | One bundle: what ships, how it is gated. |
| `docs/history/` | Superseded VibeCAD-era docs. Historical context only — never cite as current. |

Doc conventions: each doc carries a `Verified against source:` date;
provenance tags `[FreeCAD-inherited]` / `[Blender-inherited]` /
`[VibeCAD-era]` / `[Cadex-new]`; *exists today* is kept separate from
*target*. When you change behavior, update the doc and its date in the same
PR.

## Repo map

```
src/Mod/cadex/            the engine (start here; file map in docs/ARCHITECTURE.md)
src/Mod/cadex/cadex_tests/  pytest suite (headless; FreeCAD stubbed in conftest.py)
src/Mod/{Part,PartDesign,Sketcher,Assembly}   the four capability workbenches
src/Mod/{Mesh,MeshPart}   the mesh domain substrate
src/{App,Base,Main}       inherited FreeCAD core (conservative zone)
src/Gui                   present but NOT BUILT (BUILD_GUI=OFF, ADR-022);
                          deletion is Phase 8 — docs/FREECAD.md §3
shell/                    the shell — a Blender fork (conservative zone;
                          ledger and upstream diff in docs/BLENDER-TREE.md)
shell/scripts/addons_core/mesh_agent/   the add-on: ours, subtractive
                          changes encouraged (docs/BLENDER.md)
shell/lib/<platform>      submodules, NEVER content (1.3 GB prebuilt each)
                          NOTE: shell/ also carries ~790 MB in git-LFS
                          (binary assets, per shell/.gitattributes)
package/engine/           the engine payload build (ADR-023)
package/app/build_app.sh  the shell build, with the conda env scrubbed off
                          PATH — read its header before touching the build
docs/                     the documentation set above
build/release/bin/        FreeCADCmd, CadexGeometryWorker  (no FreeCAD binary)
build/engine/             the staged engine payload

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theo-kirby/cadex](https://github.com/theo-kirby/cadex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
