---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

nf-metro generates metro-map-style SVG diagrams from Mermaid graph definitions augmented with `%%metro` directives. It is designed for visualizing bioinformatics pipeline workflows (e.g., nf-core pipelines) as transit-style maps where each analysis route is a colored "metro line."

## Build & Development

```bash
# Install in development mode (uses hatchling build system)
pip install -e ".[dev]"

# Run CLI
nf-metro render examples/rnaseq_sections.mmd -o output.svg
nf-metro validate examples/rnaseq_sections.mmd
nf-metro info examples/rnaseq_sections.mmd

# Run via module
python -m nf_metro

# Run all tests
pytest

# Run a single test
pytest tests/test_parser.py::test_parse_title

# Lint
ruff check src/ tests/
```

Dependencies: click, drawsvg, networkx, pillow. Dev: pytest, ruff.

## Architecture

The pipeline is: **Parse** -> **Layout** -> **Render**

### Parser (`src/nf_metro/parser/`)
- `mermaid.py` - Line-by-line regex parser. Parses Mermaid `graph LR` syntax plus custom `%%metro` directives.
- `model.py` - Core data model: `MetroGraph`, `Station`, `Edge`, `MetroLine`, `Section`, `Port`. The `MetroGraph` dataclass is the central data structure passed through all stages.
- Sections are defined as Mermaid `subgraph` blocks with `%%metro entry:/exit:` port directives.
- Post-parse `_resolve_sections()` rewrites inter-section edges into 3-part chains: source -> exit_port -> entry_port -> target, inserting junction stations for fan-outs. Internally split into `_build_entry_side_mapping()`, `_classify_edges()`, and `_create_ports_and_junctions()`.

### Layout (`src/nf_metro/layout/`)
- `auto_layout.py` - Runs before `_resolve_sections()`. Infers missing grid positions, section directions, and port sides from the section DAG. Preserves any values explicitly set by `%%metro` directives. Handles fold thresholds (wrapping long chains into serpentine rows).
- `engine.py` - Orchestrator. Section-first layout runs 9 phases: (2) internal section layout, (3) section placement, (4) global coordinate mapping, (5) port positioning, (6) junction positioning, (7) entry port alignment, (8) exit port alignment on fold sections, (9) junction re-positioning after exit alignment. Port alignment is split into focused helpers (`_align_lr_entry_port`, `_align_tb_entry_port`, etc.).
- `layers.py` - Longest-path layering via networkx topological sort (X-axis assignment).
- `ordering.py` - Track-per-line vertical ordering (Y-axis). Each metro line gets a dedicated base track. Handles diamond (fork-join) detection for compact layout of alternative paths (e.g., FastP/TrimGalore).
- `section_placement.py` - Meta-graph layout: places sections on a grid via topological layering of section dependencies. Supports `%%metro grid:` overrides. Also handles port positioning on section boundaries.
- `constants.py` - All layout magic numbers (spacing, padding, routing tolerances, etc.). Imported by engine, routing, ordering, labels, and section_placement modules.
- `labels.py` - Station label placement.

#### Routing subpackage (`src/nf_metro/layout/routing/`)
Edge routing with horizontal runs and 45-degree diagonal transitions. Inter-section edges use L-shaped (horizontal + vertical) routing. Junction stations get horizontal offset for visual line separation in bundles.

- `core.py` - Main `route_edges()` dispatcher. Pre-computes all shared state into a `_RoutingCtx` dataclass, then dispatches each edge through handler functions in priority order: `_route_inter_section` -> `_route_tb_internal` -> `_route_tb_lr_exit` -> `_route_tb_lr_entry` -> `_route_perp_entry` -> `_route_intra_section`. First handler that returns a result wins.
- `common.py` - Shared types (`RoutedPath`) and helper functions (`compute_bundle_info`, `inter_column_channel_x`, etc.).
- `corners.py` - Corner radius computation and curve smoothing.
- `inter_section.py` - Simplified inter-section routing (used by tests).
- `offsets.py` - Per-station Y offset computation for parallel lines.
- `reversal.py` - Fold/reversal edge routing (serpentine row transitions).

### Render (`src/nf_metro/render/`)
- `svg.py` - SVG generation using the `drawsvg` library. Renders section boxes, edges (with quadratic Bezier curves at corners), pill-shaped station markers, labels, and legend.
- `animate.py` - Animated SVG balls traveling along routed metro line paths (enabled via `--animate` CLI flag).
- `style.py` - `Theme` dataclass defining all visual properties.
- `legend.py` - Legend rendering.
- `icons.py` - Icon support.
- `constants.py` - All render magic numbers (canvas padding, legend sizing, animation params, debug overlay, etc.). Theme-dependent values remain in `style.py`.

### Themes (`src/nf_metro/themes/`)
- `nfcore.py` - Dark theme (default), matching nf-core visual style.
- `light.py` - Light theme variant.
- New themes: create a `Theme` instance and register in `themes/__init__.py` `THEMES` dict.

## Input Format

`.mmd` files use a subset of Mermaid `graph LR` syntax with `%%metro` directive extensions:

```
%%metro title: Pipeline Name
%%metro style: dark

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pinin4fjords/nf-metro](https://github.com/pinin4fjords/nf-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
