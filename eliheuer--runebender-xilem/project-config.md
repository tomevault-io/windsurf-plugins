---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Runebender Xilem is a font editor built with Xilem, a Rust reactive UI framework from the Linebender ecosystem. It edits UFO (Unified Font Object) font sources and designspace (variable font) files. Status: very alpha.

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
│   ├── kerning.rs        # Kerning lookup algorithm
│   ├── entity_id.rs      # Unique entity identifiers
│   └── glyph_renderer.rs # Glyph contour → BezPath conversion
│
├── data/                 # AppState — central hub
│   ├── mod.rs            # AppState struct, Tab enum, new(), Default
│   ├── file_io.rs        # Font loading/saving (open, load, save)
│   ├── grid.rs           # Glyph grid operations (columns, scroll, filter)
│   ├── editor.rs         # Editor session management
│   └── kerning.rs        # Kerning & glyph property updates
│
├── components/           # UI components & widgets
│   ├── editor_canvas/    # Main glyph editor canvas
│   │   ├── mod.rs        # EditorWidget struct, Widget impl
│   │   ├── paint.rs      # Paint helpers (background, glyph modes)
│   │   ├── text_buffer.rs # Text buffer rendering (multi-sort layout)
│   │   ├── pointer.rs    # Pointer event handlers
│   │   ├── keyboard.rs   # Keyboard shortcut handlers
│   │   ├── drawing.rs    # Standalone drawing functions (points, metrics)
│   │   └── view.rs       # Xilem View wrapper (EditorView)
│   └── ...               # Other components (toolbars, panels)
│
├── views/                # Top-level views (welcome, grid, editor)
│   ├── editor.rs         # Glyph editing tab
│   ├── welcome.rs        # Welcome screen
│   └── glyph_grid/       # Glyph grid tab
│       ├── mod.rs        # Grid tab view, toolbar panels, grid building
│       └── glyph_cell.rs # GlyphCellWidget, GlyphCellView wrapper
├── tools/                # Editing tools (Select, Pen, Knife, etc.)
├── shaping/              # Text shaping (Arabic joining, etc.)
└── sort/                 # Multi-glyph text buffer
```

### UI Layer


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliheuer/runebender-xilem](https://github.com/eliheuer/runebender-xilem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
