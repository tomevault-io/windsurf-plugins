---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) and other LLMs when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) and other LLMs when working with code in this repository.

## Project overview

Rerun is a time-aware multimodal data stack and visualizations tool used in robotics, spatial AI, computer vision, and similar domains. It provides SDKs (Python, Rust, C++) for logging rich data (images, point clouds, tensors, etc.) and a Viewer for visualization.

## Build system

We use `pixi` for task management and dependency installation. Check `pixi.toml` for a full list of tasks.

### Essential commands

**Building:**
- `pixi run py-build` - Build Python SDK into local .venv (uses uv)
- `pixi run rerun-build` - Build native viewer (without web viewer)
- `pixi run rerun-build-web` - Build web viewer (wasm)
- `pixi run cpp-build-all` - Build all C++ artifacts

**Running:**
- `pixi run rerun` - Run the viewer
- `pixi run uvpy script.py` - Run Python scripts with rerun SDK
- `cargo run -p <package_name>` - Run specific Rust example (e.g., `cargo run -p dna`)

**Code generation:**
- `pixi run codegen` - Generate Rust/Python/C++ code from .fbs type definitions

**Formatting:**
- `pixi run rs-fmt` - Format all Rust files. Always run this after making changes.
- `pixi run py-fmt` - Format Python files
- `pixi run cpp-fmt` - Format C++ files
- `pixi run toml-fmt` - Format TOML files

**Testing:**
- Use `cargo clippy -p <crate_name>` to run general rust checks before building things
- `cargo nextest run --all-features --no-fail-fast -p <crate_name>` - Run tests for a specific crate
  - Example: `cargo nextest run --all-features --no-fail-fast -p re_view_spatial`
- Use `cargo nextest` (not `cargo test`) for better output and parallelism
- Always use `--all-features` unless you have a specific reason not to
- Use `--no-fail-fast` to gather all test failures in a single run

## Code generation system

**Critical: Never edit generated files directly.** All generated files are marked "DO NOT EDIT" at the top.

### Type definition flow

```
.fbs files (definitions/) → pixi run codegen → Generated code (Rust/Python/C++) + docs (docs/content/reference/types/)
```

- Type definitions live in `crates/store/re_sdk_types/definitions/rerun/`
  - `datatypes/*.fbs` - Low-level types (Vec3D, Mat4x4, etc.)
  - `components/*.fbs` - Component types (Position3D, Color, etc.)
  - `archetypes/*.fbs` - Archetypes (Points3D, Image, etc.)
  - `blueprint/*.fbs` - Blueprint system types
- Codegen implementation is in `crates/build/re_types_builder/`
- After modifying .fbs files, run `pixi run codegen` to regenerate code

### Extension pattern

To add custom functionality to generated types, create `_ext` files:
- Rust: `filename_ext.rs` (automatically imported by codegen)
- Python: `filename_ext.py` (mixed in with generated class)
- C++: `filename_ext.cpp` (compiled and included automatically, parts of it may be marked for copy into the header by codegen)

## Code conventions

### General

- use `…` instead of `...` <!-- NOLINT -->
- validate various custom conventions via `pixi run lint-rerun <file>` (not passing any file will check everything)
- Use `format!("{x}")` over `format!("{}, x)` (same in log calls etc)
- Don't write trivial comments that add nothing new
- In error and log messages, put the error first and any file path at the end (e.g. `Failed to import: {err}\nFile path: {path}`), never in the middle.
  Paths can be long or sensitive, so trailing placement makes them easy to strip when copy-pasting.
- Prose style (em vs en dash, sentence endings, casing) — see [`DESIGN.md`](DESIGN.md). In short: spaced em dash ` — `, never unspaced `word—word`, and don't use `–` as a sentence dash (it's for numeric ranges only) <!-- NOLINT -->
- One sentence per line in markdown files.
  Markdown joins consecutive lines into a paragraph, so rendering is unchanged — but diffs become much easier to review.

## Architecture overview

### Crate organization

```
crates/
├── build/     # Code generation (re_types_builder)
├── store/     # Data types, storage, querying
├── top/       # User-facing SDKs and CLI
└── viewer/    # Viewer UI and rendering
```

For more details about the architecture see `ARCHITECTURE.md`.

**When adding, removing, or renaming a crate**, update `ARCHITECTURE.md`:
add the crate to the appropriate crate table, and flag for the author that the crate-organization diagram (FigJam) needs a manual update — see the HTML comment next to the diagram in `ARCHITECTURE.md` for instructions.

### Type system hierarchy

The type system has three levels (generated from .fbs files):

1. **Datatypes** (`rerun.datatypes.*`) - Basic types like Vec3D, Color
2. **Components** (`rerun.components.*`) - Named semantic wrappers (Position3D, Radius)
3. **Archetypes** (`rerun.archetypes.*`) - Collections of components (Points3D, Image)

Each archetype specifies:
- Required components (must be provided)
- Recommended components (have good defaults)
- Optional components (purely optional)

Example: `Points3D` archetype requires `positions`, recommends `colors` and `radii`, allows optional `labels`.

### Data flow

```
SDK (log archetype)
    ↓ encode to Apache Arrow
LogMsg (encoded data)
    ↓ transport (gRPC/file/memory)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rerun-io/rerun](https://github.com/rerun-io/rerun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
