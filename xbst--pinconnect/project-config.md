---
trigger: always_on
description: Guidance for AI coding agents working in the PinConnect repository. Human-facing documentation lives in [`docs/`](docs/) and [`README.md`](README.md); this file covers how the pieces fit together and the conventions to follow when changing them.
---

# AGENTS.md

Guidance for AI coding agents working in the PinConnect repository. Human-facing documentation lives in [`docs/`](docs/) and [`README.md`](README.md); this file covers how the pieces fit together and the conventions to follow when changing them.

## What this project is

PinConnect turns a photo of a PCB into an interactive pinout diagram. One generator, two front ends, and an optional publishing step:

1. **pinout-design** (`pinout_design/`) — a browser app that writes a board TOML config and renders the pinout. It does not reimplement the renderer: it loads `pinout_gen` into the page under Pyodide and calls it.
2. **pinout-gen** (`pinout_gen/`) — a Python CLI that reads that TOML and renders a single interactive HTML file.
3. **pinout-embed** (`pinout_embed/`) — a Python-Markdown extension that embeds the generated HTML into MkDocs / Zensical sites.

The board TOML config is the interface between stage 1 and stage 2; the generated HTML is the interface between stage 2 and stage 3.

The tools are named with hyphens in prose (they are the command names); the underscored forms are directories and Python packages.

## Repository layout

- `pinout_gen/` — the `pinout-gen` CLI package.
  - `pinout_gen/pinout_gen/` — source: `cli.py` (argparse entry point), `config.py` (TOML dataclasses + loaders), `renderer.py` (SVG/HTML generation), `symbols.py` (bundled symbol icons + name aliases).
  - `pinout_gen/pinout_gen/connectors/` — connector type definitions (`*.toml`). **Source of truth** for connector geometry. Bundled with the package so they are found regardless of where the tool is run.
  - `pinout_gen/pinout_gen/themes/` — theme definitions (`*.toml`). **Source of truth** for themes, bundled the same way.
  - `pinout_gen/example.toml` — example board config.
- `pinout_design/` — the visual designer (vanilla ES modules, no framework).
  - `js/` — `board-model.js`, `state.js`, `*-panel.js`, `toml-io.js`, `dialogs.js`, `runtime.js` (boots Pyodide), `main.js`.
  - `py/bridge.py` — the only Python the designer adds: a thin shim over `pinout_gen` for the catalogs, the connector preview, and Generate.
  - `pinout_gen.zip` — the payload the browser fetches at startup. **Generated and git-ignored**; built by `pinout_gen.designer.build_payload`.
  - `tools/build-payload.py` — builds the payload for anyone serving this folder without `--serve`.
- `pinout_embed/` — single-module Markdown extension (`pinout_embed.py`).
- `docs/` — human documentation.
- `tools/` — repo-level doc-asset generators, each its own folder with a `generate.py` and a `README.md` (see "Regenerating doc images" below). Distinct from `pinout_design/tools/`, which holds the designer's own build scripts.

## Key facts and gotchas

- **The designer must be served over HTTP.** It fetches `pinout_gen.zip` at startup, so opening `index.html` via `file://` fails. Use `pinout-gen --serve`, which builds the payload, picks a free port and opens a browser. Serving the folder by hand needs `python pinout_design/tools/build-payload.py` first, or the designer boots into "Renderer unavailable".
- **One representation of connector types, themes, and symbols.** `pinout_gen/pinout_gen/connectors/*.toml`, `themes/*.toml` and `symbols.py` are read by the designer through the bridge, so there is nothing to mirror and nothing to regenerate. (There used to be JSON copies and a `convert-connectors.py`; both are gone. Do not reintroduce them.)
- **The designer only sees what is bundled.** A board naming its own `connector_dir` or `theme_dir` has files on disk the browser cannot read, so it renders with the bundled definition of any name it shares. `bridge.generate` returns warnings for this and the Generate dialog shows them. Keep that reporting intact: without it the designer silently disagrees with the CLI.
- **Adding a connector `style` requires code, not config.** The eleven styles (`box`, `latch`, `grid`, `header-male`, `screw-terminal`, `barrier`, `button`, `xt30`, `sherlock`, `slide-switch`, `none` — `_BODY_STYLES` in `config.py`) are drawn by custom code in `pinout_gen/pinout_gen/renderer.py`. Write it once: the designer calls the same function. `none` is the odd one out: both renderers return an empty string for it, and the page then leaves out the drawing slot and the type/pin-count line, so such a connector is a hotspot carrying only a name and a description.
- **A new `style` must be drawn in detail and match the real connector.** Study the existing `xt30` and `grid` styles as the quality bar: the render should reproduce the actual connector's housing shape as closely as possible, not a generic rectangle. Critically, the polarizing/keying features (latches, chamfers, cavities, the flat/keyed side) must be clearly visible in the render so a human can read the connector's orientation at a glance and not miswire it — this is a safety property, not a cosmetic one. A front cross-section DXF of the connector is the ideal reference to model the geometry from.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xbst/PinConnect](https://github.com/xbst/PinConnect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
