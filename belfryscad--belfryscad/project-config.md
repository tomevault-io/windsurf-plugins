---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BelfrySCAD is a hybrid procedural CAD application combining OpenSCAD-style script-based modeling with live WYSIWYG 3D interaction. Its defining feature is **bidirectional synchronization** between source code and 3D geometry — editing code or dragging geometry keeps both views in sync.

**Status**: In active development. Core pipeline, rendering, editor, and several WYSIWYG features are implemented. Full design in `PRD.md`.

## Technology Stack

- **UI Framework**: PySide6 (Qt)
- **Code Editor**: `QPlainTextEdit` + `QSyntaxHighlighter` (PySide6 built-ins; text layer only — not semantically aware)
- **Parser**: openscad_cpp_parser (C++, Bison `lalr1.cc`; generates an AST with file/line/col/span metadata; parses full OpenSCAD syntax but has no knowledge of built-in functions/modules — the evaluator implements all built-ins). Not a dependency of this project directly: it is vendored at `external/openscad_cpp_parser` inside openscad_cpp_evaluator and built with it.
- **Evaluator**: openscad_cpp_evaluator ≥0.30.0 (C++ with nanobind bindings; walks the parser's AST and produces Manifold geometry — the two-pass resolve/generate pipeline, built-ins, `ManifoldCache`, profiling; GUI-agnostic, callback-injection API). The only OpenSCAD-side dependency in `pyproject.toml`, fetched from PyPI as a wheel; see its own `CLAUDE.md` for the full architecture reference.
- **CSG Kernel**: Manifold (union, difference, intersection, boolean ops)
- **Renderer**: ModernGL (GPU mesh rendering, camera controls)
- **Language**: Python

## Core Architecture

The pipeline flows strictly one direction during normal operation:

```
Source Code → Code Editor → openscad_cpp_parser (AST) → Evaluator → Manifold (CSG/mesh) → ModernGL → PySide6 UI
```

**The AST is the single source of truth** — not the rendered geometry, not the editor text.

### Critical Constraint: Strict Parser

The parser produces **no partial AST** — it either succeeds fully or fails entirely. Handle the no-AST state gracefully:
- Cache the last valid AST
- Display last valid geometry while code is invalid
- Never block the UI or break the viewport

### Bidirectional Loop (future-critical, v1 groundwork required)

Dragging geometry in the viewport:
```
Drag event → ray cast → pick geometry ID → map ID to AST node (via span) → modify AST parameter → regenerate code + model
```

Requires every AST node to carry both its **source span** (file/line/col) and its **geometry ID(s)** from Manifold output. This mapping is the hardest design problem in the project. See `docs/wysiwyg.md` for the full interaction design and openscad_cpp_evaluator's own `CLAUDE.md` for the AST ↔ geometry ID mapping pattern.

## Key Design Requirements

- **Code ↔ Geometry mapping**: every geometry-producing AST node owns an `originalID`; the `originalID → AST node` table rebuilds on each render trigger.
- **Stability under invalid code**: UI must never crash or go blank.
- **Deterministic regeneration**: AST → geometry must be reproducible with no hidden rendering state. Every render trigger walks the whole tree, but unchanged subtrees skip actual Manifold work via a content-hash cache (`ManifoldCache`, see openscad_cpp_evaluator's `CLAUDE.md`) — a fresh AST/CSG tree is still built every render (no incremental *parsing*), but a node whose resolved content matches a previous render/debug pause reuses that prior result instead of recomputing it.
- **Performance**: <200ms model regeneration for small/medium models; 60 FPS viewport.

## File Format & Export

- **File format**: `.scad` (OpenSCAD-compatible plain text)
- **Language**: Full OpenSCAD language (variables, functions, modules, loops, conditionals, all built-in primitives and transforms)
- **Language extension — `$export_name`**: seeded with the input file's basename before
  the script runs, assignable by the script, and used (sanitised to `[A-Za-z0-9_+.-]`,
  everything else becoming one underscore each) as the Export dialog's default filename.
  Needed **no evaluator change**: `viewport_params` seeds arbitrary `$`-names and
  `Evaluator.dyn` returns them all. Seeded in the CLI and debugger too, so a script
  reading it never finds it undefined. See `belfryscad/export_name.py` and
  `docs/rendering.md`. Not part of upstream OpenSCAD.
- **Language extension — `render()` in expression position**: `obj = render() { cube(1); };`
  builds its children's geometry, measures it, and returns an `object()` with `vertices`,
  `faces`, `volume`, `area`, `genus`, `boundingbox` and `dim` — then **discards the geometry**
  (nothing is drawn). This is the only way a script can inspect its own geometry.
  `polyhedron()` and `polygon()` accept the object directly, and `polyhedron()` also takes
  BOSL2's `[vertices, faces]` 2-list, so the mesh round-trips in one call. Two consequences
  worth knowing: **`render` is a reserved keyword** (it can no longer
  be a variable/module/function/argument/member name — LALR(1) leaves no alternative), and
  **`obj = render() cube(1);` does not parse** — a bare call's `child_statement` swallows the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BelfrySCAD/BelfrySCAD](https://github.com/BelfrySCAD/BelfrySCAD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
