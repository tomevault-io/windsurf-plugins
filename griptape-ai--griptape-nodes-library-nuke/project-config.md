---
trigger: always_on
description: A Griptape Nodes library for Foundry Nuke integration. Two integration directions:
---

# griptape-nodes-library-nuke

A Griptape Nodes library for Foundry Nuke integration. Two integration directions:

1. **Griptape → Nuke** (shipped, v0.2.0): Publish a Griptape workflow as a versioned `.gizmo` installable in Nuke. See `publish_gizmo/`.
2. **Nuke → Griptape** (shipped): A `NukeScriptNode` that accepts a `.nk` script, runs it headlessly via `nuke -t`, and surfaces annotated inputs/outputs as typed ports in the Griptape canvas.

---

## Key Files

```
nuke_nodes/              Griptape node classes (NukeStartFlow, NukeEndFlow, NukeScriptNode)
  nuke_script_node.py      NukeScriptNode — dynamic ports, expose-knob, Open in Nuke, render
  artifacts.py             CameraMatrixArtifact for 4×4 camera-to-world matrix outputs
publish_gizmo/           Gizmo publisher — packages workflows as versioned .gizmo files
  nuke_discovery.py        Nuke installation auto-discovery (macOS/Windows/Linux)
  nuke_gizmo_builder.py    Generates .gizmo TCL from a workflow shape
  run_button.py            Executes inside Nuke's Python; PySide6/PySide2 progress dialog
nuke_runner/             Headless runner — scripts that run inside nuke -t (stdlib only)
  runner.py                Entry point; reads manifest, runs Write nodes, collects outputs
  introspect.py            Knob schema introspection; writes JSON to temp file
  baker.py                 Bakes knob override values into a copy of the .nk before render
  exporter.py              Extracts camera matrices and converts meshes to GLB for 3D outputs
  manifest.py              JobManifest dataclass and schema
execution/               Execution layer — launches and polls Nuke subprocesses
  provider.py              JobStatus and JobResult types
  direct.py                DirectSubprocessProvider
  installations.py         NukeInstallation, per-install launch modes and env overrides
script_parser/           Pure-Python .nk parser (no Nuke dependency)
  parser.py                Parses .nk into node/knob tree
  annotation.py            Reads gt_* knobs; GriptapeAnnotation and ExposedKnob dataclasses
  sidecar.py               Reads/writes .gt.json sidecar files
  knob_filter.py           Drops Obsolete_Knob and internal panel knobs from sidecar schema
  knob_type_map.py         Maps Nuke knob types to Griptape parameter types
nuke_plugin/             Griptape Annotator — dockable panel that runs inside Nuke GUI
  panel.py                 PySide2 panel: Annotate tab, Overview tab, expose-knob UI
  installer.py             get_plugin_path() — resolves versioned plugin directory
  nuke14/ … nuke17/        Version-specific menu.py registrations
presets/                 (planned) Bundled preset .nk scripts
griptape-nodes-library.json   Library manifest — register nodes here
pyproject.toml           Dependencies and tool config
Makefile                 All dev workflows (check, fix, version, deps)
```

---

## Coding Rules

**Language and runtime**
- Griptape-side code: Python 3.12 (matches `pyproject.toml`).
- Nuke-runner scripts (`nuke_runner/runner.py`, `nuke_runner/introspect.py`): Python 3.10+ (must run inside Nuke 15+'s bundled interpreter). These files must be **stdlib only** — no third-party imports. `nuke` itself is available.
- PySide6 first, PySide2 fallback. Match the pattern in `publish_gizmo/run_button.py`.

**Dependencies**
- Use `uv` only — never `pip install` directly.
- Adding a new dependency: edit `pyproject.toml`, then run `make deps/sync` to sync `pip_dependencies` into the library JSON.

**Style**
- Ruff (format + lint) and pyright for type checking. Run `make check` before every commit.
- `make fix` auto-corrects format and lint issues.
- No comments unless the WHY is non-obvious (a hidden constraint, a workaround, a subtle invariant). Do not describe what the code does.
- One-sentence docstrings max on public classes. No multi-line docstrings.
- `from __future__ import annotations` first in every file. Stdlib → third-party → local import order.

---

## Hard Constraints

**Never write `.nk` file content by hand.**
Nuke's format is a TCL dialect with delta-from-defaults encoding and many undocumented edge cases. Always generate scripts via `nuke -t <script.py>` using Nuke's own Python API. The preset scripts in `presets/` must be created and validated by actually running them through Nuke.

**`nuke_runner/` scripts must stay stdlib-only.**
These run inside Nuke's bundled Python where only the standard library and the `nuke` module are available. Never add a third-party import to `runner.py` or `introspect.py`.

**Forward slashes for all paths passed to or from Nuke.**
Nuke's TCL layer treats backslashes as escape characters when saving `.nk` files. Always call `.replace("\\", "/")` on paths before passing them into Nuke knobs or writing them to manifests.

**Never hardcode license server addresses.**
The `foundry_LICENSE` environment variable is set by the Griptape secrets panel and injected into the subprocess at launch. Never embed a license server string in code.

**Never call `subprocess.Popen()` from node code directly.**
All Nuke process launch goes through `DirectSubprocessProvider` (`execution/direct.py`). Node code calls `provider.submit(manifest)` only.

**Never modify a `.nk` file by string manipulation.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [griptape-ai/griptape-nodes-library-nuke](https://github.com/griptape-ai/griptape-nodes-library-nuke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
