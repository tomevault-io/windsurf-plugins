---
trigger: always_on
description: Context for AI coding agents working on `runebender-xilem`. Evergreen
---

# AGENTS.md

Context for AI coding agents working on `runebender-xilem`. Evergreen
info only — architecture, build, conventions, load-bearing gotchas.
Task-specific plans live under `.agents/`. New agents: read this
top-to-bottom before touching code, then check `.agents/` for active
plans and `.agents/active/` for in-flight claims by other agents.

Agent-name-agnostic: Codex (native `AGENTS.md` convention),
Claude Code (via `CLAUDE.md` → here), and any other human-driven
agent that reads this file get the same instructions.

## What this is

Runebender Xilem is a native Rust font editor built with Xilem, the
Linebender reactive UI framework. It edits UFO (Unified Font Object)
font sources and designspace (variable-font) files. Status: alpha.

This is the canonical Runebender UI/UX reference. The
`runebender-comfy` port (WASM/Vue, ComfyUI custom node) mirrors this
repo's `src/components/*.rs` 1:1 in Vue. When you change a component
here, expect a comfy follow-up.

## Sister repos

All assumed to be siblings under `~/GH/repos/`:

| Repo | License | Role |
|---|---|---|
| `runebender-xilem` | Apache-2.0 | **This repo.** Canonical native editor + UI/UX reference. |
| `runebender-core` | Apache-2.0 | Shared editing/model crate. Local `path = "../runebender-core"` dep. |
| `runebender-comfy` | GPL-3.0 | WASM/Vue port for ComfyUI. Mirrors this repo's UI/UX. |

Fresh-clone needs `runebender-core` checked out as a sibling (or
switch to a git dep before public publish).

## ⚠ Load-bearing gotcha: the kurbo version split

- `runebender-xilem` is pinned to **kurbo 0.12** (via masonry 0.4).
- `runebender-comfy` is on **kurbo 0.13** (forced by peniko 0.5 /
  vello 0.8).
- The `spline` crate uses kurbo 0.9 internally; conversion happens
  at the boundary.

Sharing kurbo-using modules between xilem and comfy currently
produces ~289 errors of `masonry::kurbo::X is not kurbo::X`.
Switching xilem to masonry-2 is a multi-week project.

**Do not naively bump this repo's kurbo.** Only modules with NO kurbo
types in their public API can move into `runebender-core`. Today
that's selection, undo, edit_types, entity_id, kerning, category.

## Build and Development Commands

```bash
cargo build                          # Debug build
cargo run                            # Run (opens file picker)
cargo run -- assets/untitled.ufo     # Open a specific UFO file
cargo run -- --verbose               # Run with verbose logging
cargo check                          # Type-check without building
cargo clippy                         # Lint (uses .clippy.toml with Linebender canonical lints)
cargo fmt                            # Format (uses .rustfmt.toml)
cargo build --release                # Release build
```

There is no test suite. No CI/CD pipeline.

## Architecture

### Data Flow

Xilem reactive architecture with single-direction data flow:
```
AppState → app_logic() → View Tree → Masonry Widgets → Vello Rendering
```
The entire UI is rebuilt from `AppState` on each update. State mutations happen in button/event callbacks.

### Key State Types

- **`AppState`** (`src/data.rs`) — Central app state: loaded workspace, selected glyph, active edit session, current tab, window metadata
- **`Workspace`** (`src/model/workspace.rs`) — Font data model wrapping `norad` UFO types. Thread-safe via `Arc<RwLock<Workspace>>`. Glyphs sorted by Unicode codepoint
- **`EditSession`** (`src/editing/session.rs`) — Per-glyph editing state: editable paths, selection, current tool, viewport, undo/redo history, text buffer for multi-glyph editing

### Module Layout

```
src/
├── lib.rs                # Root app_logic(), window setup
├── main.rs               # Entry point
├── theme.rs              # Color constants
├── settings.rs           # Config constants
│
├── path/                 # Path representation & geometry
│   ├── mod.rs            # Path enum, re-exports
│   ├── cubic.rs          # Cubic bezier paths (UFO default)
│   ├── quadratic.rs      # Quadratic/TrueType paths
│   ├── hyper.rs          # Hyperbezier paths (spline solver)
│   ├── point.rs          # PathPoint, PointType
│   ├── point_list.rs     # PathPoints collection
│   ├── segment.rs        # Segment types for hit-testing
│   └── quadrant.rs       # Quadrant geometry utility
│
├── editing/              # Editing model & interaction
│   ├── mod.rs            # Re-exports
│   ├── session/          # Per-glyph editing state (EditSession)
│   │   ├── mod.rs        # Struct, constructors, component methods, tests
│   │   ├── text_buffer.rs # Sort creation, Arabic shaping, buffer management
│   │   ├── hit_testing.rs # Point/segment/component hit tests
│   │   └── path_editing.rs # Point movement, deletion, contour operations
│   ├── selection.rs      # Entity selection set
│   ├── edit_types.rs     # Undo grouping types
│   ├── undo.rs           # Undo/redo system
│   ├── hit_test.rs       # Cursor hit-testing
│   ├── mouse.rs          # Mouse event state machine
│   └── viewport.rs       # Design↔screen coordinate transform
│
├── model/                # Font data model
│   ├── mod.rs            # Re-exports EntityId
│   ├── workspace.rs      # UFO font data (Workspace, Glyph, etc.)
│   ├── designspace.rs    # Variable font designspace support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliheuer/runebender-xilem](https://github.com/eliheuer/runebender-xilem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
