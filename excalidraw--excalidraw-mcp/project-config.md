---
trigger: always_on
description: Standalone MCP server that streams Excalidraw diagrams as SVG with hand-drawn animations.
---

# Excalidraw MCP App Server

Standalone MCP server that streams Excalidraw diagrams as SVG with hand-drawn animations.

## Architecture

```
server.ts          → 2 tools (read_me, create_view) + resource + cheat sheet
main.ts            → HTTP (Streamable) + stdio transports
src/mcp-app.tsx    → ExcalidrawAppCore (widget logic) + ExcalidrawApp (useApp wrapper)
src/mcp-entry.tsx  → Production entry point: createRoot + ExcalidrawApp
src/global.css     → Animations (stroke draw-on, fade-in) + auto-resize
src/dev.tsx        → Dev entry point: mock app + sample elements + control panel
src/dev-mock.ts    → Mock MCP App with event simulation (sendToolInput, streamElements, etc.)
index-dev.html     → Dev HTML entry (served by vite dev server)
vite.config.dev.ts → Dev-only vite config (resolves from node_modules, no esm.sh externals)
```

## Tools

### `read_me` (text tool, no UI)
Returns a cheat sheet with element format, color palettes, coordinate tips, and examples. The model should call this before `create_view`.

### `create_view` (UI tool)
Takes `elements` — a JSON string of standard Excalidraw elements. The widget parses partial JSON during streaming and renders via `exportToSvg` + morphdom diffing. No Excalidraw React canvas component — pure SVG rendering.

**Screenshot as model context:** After final render, the SVG is captured as a 512px-max PNG and sent via `app.updateModelContext()` so the model can see the diagram and iterate on user feedback.

## Key Design Decisions

### Standard Excalidraw JSON — no extensions
The input is standard Excalidraw element JSON. No `label` on containers, no `start`/`end` on arrows. These are Excalidraw's internal "skeleton" API (`convertToExcalidrawElements`) — not the standard format.

**Why:** Standard format means any `.excalidraw` file's elements array works as input.

**Trade-off:** Labels require separate text elements with manually computed centered coordinates. The cheat sheet teaches the formula: `x = shape.x + (shape.width - text.width) / 2`.

### No `convertToExcalidrawElements`
We tried Excalidraw's skeleton API. Problems:
1. Needs font metrics at conversion time (canvas `measureText`)
2. Non-standard format
3. Added complexity for marginal benefit

### SVG-only rendering (no Excalidraw React canvas)
The widget uses `exportToSvg` for ALL rendering — no `<Excalidraw>` React component.

**Why:**
- Eliminates blink on final render (no component swap from SVG preview to canvas)
- Loads Virgil hand-drawn font from the start (no `skipInliningFonts`)
- morphdom works on SVG DOM — smooth diffing between streaming updates

### Auto-sizing
The container has no fixed height. SVG gets `width: 100%` + `height: auto` with the `width` attribute removed. The SVG's `viewBox` preserves aspect ratio, so height scales proportionally to content.

### CSP: `esm.sh` allowed
Excalidraw loads the Virgil font from `esm.sh` at runtime. The resource's `_meta.ui.csp.resourceDomains` includes `https://esm.sh`.

### `prefersBorder: true`
Set on the resource content's `_meta.ui` so the host renders a border/background around the widget.

### Fullscreen mode
Supports `app.requestDisplayMode({ mode: "fullscreen" })`. Button appears on hover (top-right), hidden in fullscreen (host provides exit UI). Escape key exits fullscreen.

## Checkpoint System

Two-tier storage for diagram state persistence:

### Architecture
1. **Server-side store** (primary): `CheckpointStore` interface with 3 implementations:
   - `FileCheckpointStore` — local dev, writes JSON to `$TMPDIR/excalidraw-mcp-checkpoints/`
   - `MemoryCheckpointStore` — Vercel fallback (in-memory Map, lost on cold start)
   - `RedisCheckpointStore` — Vercel with Upstash KV (persistent, 30-day TTL)
   - Factory: `createVercelStore()` picks Redis if env vars exist, else Memory

2. **localStorage** (widget-side cache): Fast local cache keyed by `excalidraw:<checkpointId>` for persisting user edits across page reloads within the same session.

### Flow
- `create_view` resolves `restoreCheckpoint` references server-side, saves fully resolved state, returns `checkpointId`
- Widget reads checkpoints via `read_checkpoint` server tool (private, app-only visibility)
- User edits in fullscreen sync back to server via `save_checkpoint` server tool (debounced)
- `cameraUpdate` elements are stored as part of checkpoint data (not a separate viewport field)

### Key Design Decisions
- Server resolves checkpoints so the model never needs to re-send full element arrays
- `containerId` filtering ensures bound text elements are deleted with their containers
- Camera aspect ratio check nudges model toward 4:3 ratios
- `checkpointId` uses `crypto.randomUUID()` truncated to 18 chars (collision-resistant, URL-safe)

## Build

```bash
npm install
npm run build
```

Build pipeline: `tsc --noEmit` → `vite build` (singlefile HTML) → `tsc -p tsconfig.server.json` → `bun build` (server + index).

## Running

```bash
# HTTP (Streamable) — default, stateless per-request
npm run serve          # or: bun --watch main.ts
# Starts on http://localhost:3001/mcp

# stdio — for Claude Desktop
node dist/index.js --stdio

# Dev mode (watch + serve) — full MCP flow
npm run dev

# Dev mode (standalone UI) — no MCP server needed
npm run dev:ui

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [excalidraw/excalidraw-mcp](https://github.com/excalidraw/excalidraw-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
