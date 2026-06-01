---
trigger: always_on
description: StarBreaker is a Rust workspace for inspecting and exporting Star Citizen data
---

# StarBreaker Copilot instructions

StarBreaker is a Rust workspace for inspecting and exporting Star Citizen data
from `Data.p4k`. The workspace includes the CLI (`cli/`, binary
`starbreaker`), reusable format crates (`crates/starbreaker-*`), an MCP server
(`mcp/`), a Tauri/React app (`app/`), an Astro docs site (`website/`), and a
Blender addon (`blender_addon/`). Read `AGENTS.md` first for the full project
rules; read `blender_addon/AGENTS.md` before editing addon code.

## Required first reads

Before planning or editing, read the relevant instruction files in this order:

1. `AGENTS.md` in the StarBreaker root.
2. `blender_addon/AGENTS.md` before any Blender addon work.
3. This file (`StarBreaker/.github/copilot-instructions.md`).

Do not rely on memory when these files are available. Re-read them when changing
component, starting a new phase, or resuming after a long interruption.

## Context: This repo

- StarBreaker is the main git repository; work inside this folder.
- The workspace root is at `../../` and contains workspace-level guidance, research
  documentation, and generated/exported packages. Follow workspace root `AGENTS.md`
  for broader context, but this file takes precedence for StarBreaker-specific work.

## Build, test, and lint commands

Run commands from the StarBreaker git root, not the parent `scorg_tools/`
workspace.

```bash
# Rust iteration build; dev profile has opt-level=1 and optimized deps.
cargo build

# Release CLI build, used for final exporter runs and CI artifacts.
cargo build --release -p starbreaker

# Full Rust workspace tests.
cargo test --workspace

# CI-equivalent Rust checks.
cargo build --release --workspace
cargo test --release --workspace

# Target one crate or one test by substring.
cargo test -p starbreaker-3d --lib
cargo test -p starbreaker-3d validate_lights_extraction -- --nocapture

# Lint/format Rust when the components are installed.
cargo clippy --workspace --all-targets
cargo fmt --all
```

```bash
# Blender addon tests use stubbed bpy and run on system Python.
cd blender_addon
python3 -m unittest discover -s tests -q
python3 -m unittest discover -s tests -p 'test_manifest.py' -q
```

```bash
# Tauri app frontend.
cd app
npm ci
npm run build
npm run lint
npm run tauri build

# Documentation website.
cd website
npm ci
npm run build
```

For native decomposed `.blend` export performance/regression work, use the
checked-in profiler:

```bash
tools/profile_aurora_blend.sh /tmp/starbreaker_aurora_profile 0
```

The CLI auto-detects common Star Citizen install paths. Set `SC_DATA_P4K` only
when targeting a non-default install. The canonical Aurora decomposed export is:

```bash
SC_DATA_P4K="$HOME/Games/star-citizen/drive_c/Program Files/Roberts Space Industries/StarCitizen/LIVE/Data.p4k" \
  target/release/starbreaker entity export "aurora_mk2" \
  "../ships" --kind decomposed --format blend --lod 0 --mip 0 --materials all
```

## Architecture

- `crates/starbreaker-3d` owns entity export. The pipeline is split under
  `src/pipeline/`: `loadout.rs` resolves DataCore loadouts and item ports,
  `interiors.rs` handles interior placements, `textures.rs` and `palette.rs`
  build shared package assets, `glb_assembly.rs` assembles GLB output, and
  `blend_assembly.rs` writes native `.blend` package scenes and decomposed
  asset libraries. `decomposed.rs` coordinates package output and shared
  `Data/...` asset reuse.
- Lower-level format crates are deliberately separate: `starbreaker-p4k` reads
  archives, `starbreaker-datacore` parses the database, `starbreaker-cryxml`
  decodes binary XML, `starbreaker-chunks` handles CryEngine chunk files,
  `starbreaker-dds` decodes textures, and Wwise/WEM/CHF crates cover audio and
  character formats.
- The decomposed export contract is shared between Rust and Blender. Rust emits
  `Packages/<package>/scene.json`, `palettes.json`, `liveries.json`, material
  sidecars, textures, and reusable assets under `Data/...`; the Blender addon
  imports those packages and reconstructs materials, lights, palette/livery
  state, and animation controls.
- `blender_addon/starbreaker_addon/runtime/importer/` composes
  `PackageImporter` from mixins: palette, decals, layers, materials, builders,
  groups, and orchestration. Keep new per-entity import behavior in focused
  helpers or mixins rather than growing orchestration.
- The Tauri app wraps the Rust crates from `app/src-tauri` and uses the React
  frontend in `app/src`. The website is independent Astro/Starlight content in
  `website/`.

## MCP workflows

- StarBreaker MCP is the preferred way to inspect game data while researching
  exporter bugs. Use it for fast DataCore, P4k, material, texture, chunk, and
  animation lookups instead of writing one-off CLI probes.
- Use `search_entities` for EntityClassDefinition lookup, `search_records` for
  all DataCore record types, `datacore_record` for full JSON records, and
  `datacore_query` for a specific property path. For raw loadout data, query
  `Components[SEntityComponentDefaultLoadoutParams]`; `entity_loadout` returns
  StarBreaker's processed/resolved tree.
- Use `p4k_search`, `p4k_list`, and `p4k_read` to find and inspect archive
  paths. Use `mtl_summary` for material XML summaries, `image_preview` for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diogotr7/StarBreaker](https://github.com/diogotr7/StarBreaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
