---
trigger: always_on
description: Library for building node-based UIs in the terminal — from a static diagram to a fully interactive editor. Built on [ratatui](https://github.com/ratatui/ratatui), inspired by [xyflow](https://github.com/xyflow/xyflow) (React Flow).
---

# rataflow

Library for building node-based UIs in the terminal — from a static diagram to a fully interactive editor. Built on [ratatui](https://github.com/ratatui/ratatui), inspired by [xyflow](https://github.com/xyflow/xyflow) (React Flow).

See `docs/ARCHITECTURE.md` and `docs/INTERNALS.md` for design rationale and implementation details. See `README.md` for performance characteristics and feature flags.

## Documentation Principles

**Parallel abstractions get parallel wording.** When two types share the same design philosophy (e.g., `EdgeStyle`/`HandleStyle`, `StepEdge`/`StraightEdge`, `FlowAction`/`ControlsAction`), their doc comments must use the same sentence structure, same level of specificity, and same categories of description.

Examples of parallel wording:
- `EdgeStyle`: "Visual configuration for edge rendering." / `EdgePreviewStyle`: "Visual configuration for edge preview rendering." / `HandleStyle`: "Visual configuration for handle rendering."
- `StepEdge`: "Built-in edge type that routes with..." / `StraightEdge`: "Built-in edge type that draws..."
- `FlowAction`: "Semantic actions for flow graph interaction." / `ControlsAction`: "Semantic actions for Controls widget interaction."
- Companion widgets: all open with "Companion widget for [purpose]." then describe what they render.

When adding new abstractions, find the existing parallel and match its structure.

**This governs naming as well as wording — but only where the things named actually correspond.** `EdgeStyle::stroke_style` and `HandleStyle::char_style` look like they should match and deliberately don't, because a stroke is not the parallel of a char: an edge has rasterization modes and `EdgeStroke::Braille` has no character to name, while a handle is a single character. Matching the names would assert a parallel that isn't there. See `docs/INTERNALS.md` § Braille Strokes.

## Rendering Model

Three distinct rendering approaches — this asymmetry is intentional, do not "fix" it:

| Element | Custom rendering? | Library primitive |
|---------|------------------|------------------|
| **Nodes** | Yes (`NodeContent`) | None — use ratatui directly |
| **Edges** | Yes (`EdgeContent`) | `EdgeStyle` + `EdgeRenderContext::render_path()` |
| **Handles** | No — library-rendered | `HandleStyle` on `Handle` instances |

All style structs (`EdgeStyle`, `HandleStyle`, `EdgePreviewStyle`, `ControlsStyle`, `MiniMapStyle`, `BackgroundStyle`) share the same design: private fields, `Default` + builders, `Option<Style>`/`Option<Color>` for theme-derived values. `None` means "use theme" — resolved at render time via `resolved_style(palette)`. Structural fields (characters, markers, booleans) are concrete with sensible defaults since they don't vary by theme.

### Canvas Rendering Pipeline

1. Render edges + edge preview to separate buffer (symbol merging at intersections)
2. Composite edge buffer onto main buffer (non-space cells only)
3. Render nodes + handles in z-order (body then handles per node, into per-node scratch buffers)

**Node scratch buffers:** Each visible node is rendered into a per-node buffer at local `(0, 0)` coordinates with full dimensions, then only the visible portion is composited onto the main buffer. This ensures `NodeContent::render()` always receives the complete area (correct partial rendering) and sidesteps ratatui's u16 coordinate space (nodes off the left/top edge have negative terminal positions). The overhead is negligible even at scale: most nodes are culled by the visibility check before any buffer is allocated, each buffer is node-sized (typically ~10x3 cells), and frame time is dominated by the canvas-sized edge buffer and edge path computation — not node rendering. Benchmarked at 28k+ nodes (WASM) and 37k+ nodes (native) with no measurable impact on frame time or memory.

**Node opacity:** Nodes are opaque by default (`node.opaque = true`) — the entire node area blocks content behind it, even cells not written by `NodeContent::render()`. Set `opaque: false` on parent nodes in hierarchical graphs so edges and children remain visible inside the parent.

## Module Structure

```
src/
├── types/      # Primitives: Position, Dimensions, Node, Edge, Handle, Viewport
├── state/      # Flow, split semantically: graph ops, selection, viewport, mouse, edge preview, auto-pan, event handling, FlowOps trait
├── ui/         # Widgets and rendering, split semantically: edges, handles, controls, minimap, background
├── actions.rs  # Semantic actions (FlowAction) and events (FlowEvent)
├── content.rs  # Extension point: NodeContent, EdgeContent traits + render contexts
├── input.rs    # Backend-agnostic input types (KeyEvent, MouseEvent)
├── error.rs    # Structured error types with thiserror
└── layout.rs   # Layout algorithms (feature-gated: `sugiyama`)
```

`types`, `state`, `ui` are the three layers. The standalone modules (`actions`, `content`, `input`, `error`, `layout`) are library-wide concerns — their names speak for themselves.

## Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [furkankly/rataflow](https://github.com/furkankly/rataflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
