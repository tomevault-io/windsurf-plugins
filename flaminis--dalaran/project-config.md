---
trigger: always_on
description: Guidance for LLMs working in this repo.
---

# CLAUDE.md

Guidance for LLMs working in this repo.

## Project overview

Dalaran: time-aware multimodal data stack + visualization for robotics, spatial AI, computer vision. SDKs (Python, Rust, C++) log rich data (images, point clouds, tensors, etc.). Viewer for visualization.

## Build system

`pixi` for task management + deps. See `pixi.toml` for full task list.

### Essential commands

**Building:**
- `pixi run py-build` - Build Python SDK into local .venv (uses uv)
- `pixi run dalaran-build` - Build native viewer (without web viewer)
- `pixi run dalaran-build-web` - Build web viewer (wasm)
- `pixi run cpp-build-all` - Build all C++ artifacts

**Running:**
- `pixi run dalaran` - Run viewer
- `pixi run uvpy script.py` - Run Python scripts with dalaran SDK
- `cargo run -p <package_name>` - Run specific Rust example (e.g., `cargo run -p dna`)

**Code generation:**
- `pixi run codegen` - Generate Rust/Python/C++ code from .fbs type definitions

**Formatting:**
- `pixi run rs-fmt` - Format Rust files. **Always run after editing Rust files, before committing.**
- `pixi run py-fmt` - Format Python files
- `pixi run cpp-fmt` - Format C++ files
- `pixi run toml-fmt` - Format TOML files

**Testing:**
- `cargo clippy -p <crate_name>` - Run rust checks before building
- `cargo nextest run --all-features --no-fail-fast -p <crate_name>` - Run tests for specific crate
  - Example: `cargo nextest run --all-features --no-fail-fast -p dl_view_spatial`
- Use `cargo nextest` (not `cargo test`) for better output + parallelism
- Always use `--all-features` unless specific reason not to
- Use `--no-fail-fast` to gather all failures in single run

**Snapshots:**
- **`insta` snapshots**: Text-based, run with regular Rust tests. On failure: `cargo insta review` (install: `cargo install cargo-insta`)
- **Image comparison tests**: Render image vs checked-in reference. Uses `egui_kittest`'s `Harness::snapshot` + `TestContext` for mocking viewer.
  - Results saved to `tests/snapshots/`, failures produce `diff.png`
  - Update refs: `UPDATE_SNAPSHOTS=1`
  - There is no CI here, so snapshots are always updated locally
  - Best practices: see [egui_kittest README](https://github.com/emilk/egui/tree/master/crates/egui_kittest#snapshot-testing)

## Code generation system

**Critical: Never edit generated files directly.** All generated files marked "DO NOT EDIT" at top.

### Type definition flow

```
.fbs files (definitions/) → pixi run codegen → Generated code (Rust/Python/C++) + docs (docs/content/reference/types/)
```

- Type definitions in `crates/store/dl_sdk_types/definitions/dalaran/`
  - `datatypes/*.fbs` - Low-level types (Vec3D, Mat4x4, etc.)
  - `components/*.fbs` - Component types (Position3D, Color, etc.)
  - `archetypes/*.fbs` - Archetypes (Points3D, Image, etc.)
  - `blueprint/*.fbs` - Blueprint system types
- Codegen implementation in `crates/build/dl_types_builder/`
- After modifying .fbs files, run `pixi run codegen` to regenerate

### Extension pattern

Add custom functionality to generated types via `_ext` files:
- Rust: `filename_ext.rs` (auto-imported by codegen)
- Python: `filename_ext.py` (mixed into generated class)
- C++: `filename_ext.cpp` (compiled + included auto, parts may be marked for copy into header by codegen)

## Code conventions

### General

- use `…` instead of `...` <!-- NOLINT -->
- Validate conventions via `pixi run lint-dalaran <file>` (no file = check everything)
- Prose style (em vs en dash, sentence endings, casing) — see [`DESIGN.md`](DESIGN.md). In short: spaced em dash ` — `, never unspaced `word—word`, and don't use `–` as a sentence dash (it's for numeric ranges only) <!-- NOLINT -->
- In error and log messages, put the error first and any file path at the end (e.g. `Failed to import: {err}\nFile path: {path}`), never in the middle.
  Paths can be long or sensitive, so trailing placement makes them easy to strip when copy-pasting.
- One sentence per line in markdown files.
  Markdown joins consecutive lines into a paragraph, so rendering is unchanged — but diffs become much easier to review.

## Architecture overview

### Crate organization

```
crates/
├── build/     # Code generation (dl_types_builder)
├── store/     # Data types, storage, querying
├── top/       # User-facing SDKs and CLI
└── viewer/    # Viewer UI and rendering
```

More details in `ARCHITECTURE.md`.

**When adding, removing, or renaming a crate**, update `ARCHITECTURE.md`:
add the crate to the appropriate crate table, and flag for the author that the crate-organization diagram (FigJam) needs a manual update — see the HTML comment next to the diagram in `ARCHITECTURE.md` for instructions.

### Type system hierarchy

Three levels (generated from .fbs files):

1. **Datatypes** (`dalaran.datatypes.*`) - Basic types like Vec3D, Color
2. **Components** (`dalaran.components.*`) - Named semantic wrappers (Position3D, Radius)
3. **Archetypes** (`dalaran.archetypes.*`) - Collections of components (Points3D, Image)

Each archetype specifies:
- Required components (must provide)
- Recommended components (good defaults)
- Optional components

Example: `Points3D` requires `positions`, recommends `colors` and `radii`, optional `labels`.

### Data flow

```
SDK (log archetype)
    ↓ encode to Apache Arrow
LogMsg (encoded data)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Flaminis/Dalaran](https://github.com/Flaminis/Dalaran) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
