---
trigger: always_on
description: Vue 3 + CanvasKit (Skia WASM) + Yoga WASM design editor. Tauri v2 desktop, also runs in browser.
---

# OpenPencil

Vue 3 + CanvasKit (Skia WASM) + Yoga WASM design editor. Tauri v2 desktop, also runs in browser.

**Roadmap:** `plan.md` — phases, tech stack, CLI architecture, test strategy, keyboard shortcuts.

## Monorepo

Bun workspace with three packages:

- `packages/core` — `@open-pencil/core`: scene graph, renderer, layout, codec, kiwi, clipboard, vector, snap, undo. Zero DOM deps, runs headless in Bun.
- `packages/cli` — `@open-pencil/cli`: headless CLI for .fig inspection, export, linting. Uses `citty` + `agentfmt`.
- `packages/docs` — `@open-pencil/docs`: VitePress documentation site. Run with `cd packages/docs && bun run dev`.
- `packages/mcp` — `@open-pencil/mcp`: MCP server for AI coding tools. Stdio + HTTP (Hono). Reuses `createServer()` factory with all core tools.

- `packages/vue` — `@open-pencil/vue`: headless Vue 3 SDK (Reka UI-style) for building custom OpenPencil-powered editor shells and embedded editing surfaces. Renderless components and composables. The app is one consumer of the SDK.

The root app (`src/`) is the Tauri/Vite desktop editor. Its `src/engine/` files are thin re-export shims from `@open-pencil/core`. `src/composables/use-canvas.ts` re-exports from `@open-pencil/vue`.

### Core subpath exports

`@open-pencil/core` exposes domain-specific subpath exports for targeted imports. The main `"."` entry re-exports everything for backward compatibility.

| Subpath | What | Heavy dep isolated |
|---|---|---|
| `@open-pencil/core` | everything (barrel) | all |
| `@open-pencil/core/scene-graph` | SceneGraph, node types, hit-test, copy, snap, undo | — |
| `@open-pencil/core/color` | parseColor, colorToHex, color management, OkHCL | culori |
| `@open-pencil/core/text` | fonts, text editor, style runs, direction | — |
| `@open-pencil/core/vector` | vector network encode/decode, bezier math | — |
| `@open-pencil/core/figma-api` | FigmaAPI, FigmaNodeProxy | — |
| `@open-pencil/core/icons` | Iconify API client, icon rendering | @iconify/utils |
| `@open-pencil/core/canvas` | SkiaRenderer (Skia/CanvasKit painting engine) | — |
| `@open-pencil/core/design-jsx` | JSX-to-design renderer | sucrase |
| `@open-pencil/core/editor` | createEditor, Editor, EditorState | — |
| `@open-pencil/core/tools` | ToolDef, ALL_TOOLS, AI adapter | diff |
| `@open-pencil/core/kiwi` | .fig parse/serialize, codec, protocol | fflate, fzstd |
| `@open-pencil/core/rpc` | RPC commands for CLI | — |
| `@open-pencil/core/lint` | design linter rules and presets | — |
| `@open-pencil/core/profiler` | render profiling | — |
| `@open-pencil/core/canvaskit` | getCanvasKit loader | canvaskit-wasm |
| `@open-pencil/core/layout` | computeLayout | yoga-layout |

Runtime `canvaskit-wasm` import exists only in `canvaskit.ts` — all other files use `import type`. CanvasKit instance is passed as a parameter everywhere.

### Editor architecture

`packages/core/src/editor/` is the framework-agnostic editor core — 13 modules sharing an `EditorContext` interface:

| Module | What |
|---|---|
| `types.ts` | EditorState, EditorOptions, Tool, EditorToolDef, EditorContext |
| `create.ts` | `createEditor()` assembler — wires context + all modules |
| `viewport.ts` | screenToCanvas, applyZoom, pan, zoomToFit/100/Selection |
| `selection.ts` | select, clearSelection, marquee, snap, hover, entered container |
| `pages.ts` | switchPage, addPage, deletePage, renamePage |
| `shapes.ts` | createShape, pen tool, adoptNodesIntoSection |
| `structure.ts` | group, ungroup, wrapInAutoLayout, reorder, reparent, z-order |
| `components.ts` | component/instance/detach/componentSet |
| `clipboard.ts` | duplicate, copy, paste, delete, storeImage |
| `undo.ts` | commitMove/Resize/Rotation, snapshot/restore |
| `text.ts` | startTextEditing, commitTextEdit |
| `nodes.ts` | updateNode, updateNodeWithUndo, setLayoutMode |

Each module exports a factory: `createXxxActions(ctx: EditorContext) => { ... }`.
`create.ts` assembles context + all modules, spreads into a flat return object.
`Editor` type = `ReturnType<typeof createEditor>`.

The app store (`src/stores/editor.ts`) is a thin Vue wrapper: creates `shallowReactive` state, calls `createEditor()`, adds Vue-specific concerns (computed refs, file I/O, autosave, export, image placement, mobile clipboard).

## Commands

- `bun run check` — type-aware lint + typecheck via oxlint + tsgo (run before committing)
- `bun run check:vue` — vue-tsc type-check for .vue files (has pre-existing errors, fix progressively)
- `bun run test:dupes` — jscpd copy-paste detection across all TS sources
- `bun run format` — oxfmt with import sorting
- `bun test ./tests/engine` — unit tests
- `bun run test` — Playwright visual regression
- `bun run tauri dev` — desktop app with hot reload
- `bun open-pencil info <file>` — document stats
- `bun open-pencil tree <file>` — node tree
- `bun open-pencil find <file>` — search nodes
- `bun open-pencil node <file> --id <id>` — detailed node properties
- `bun open-pencil pages <file>` — list pages
- `bun open-pencil variables <file>` — list design variables
- `bun open-pencil export <file>` — headless render to PNG/JPG/WEBP
- `bun open-pencil analyze colors <file>` — color palette usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-pencil/open-pencil](https://github.com/open-pencil/open-pencil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
