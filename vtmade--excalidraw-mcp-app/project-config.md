---
trigger: always_on
description: Standalone MCP server that streams Excalidraw diagrams as SVG with hand-drawn animations.
---

# Excalidraw MCP App Server

Standalone MCP server that streams Excalidraw diagrams as SVG with hand-drawn animations.

## Architecture

```
server.ts        → 2 tools (read_me, create_view) + resource + cheat sheet
main.ts          → HTTP (Streamable) + stdio transports
src/mcp-app.tsx  → React widget: SVG-only rendering via exportToSvg + morphdom
src/global.css   → Animations (stroke draw-on, fade-in) + auto-resize
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

# Dev mode (watch + serve)
npm run dev
```

## Claude Desktop config

```json
{
  "excalidraw": {
    "command": "node",
    "args": ["<path>/dist/index.js", "--stdio"]
  }
}
```

## Rendering Pipeline

### Streaming (`ontoolinputpartial`)
1. `parsePartialElements` tries `JSON.parse`, falls back to closing array after last `}`
2. `excludeIncompleteLastItem` drops the last element (may be incomplete)
3. Only re-renders when element **count** changes (not on every partial update)
4. Seeds are **randomized** per render — hand-drawn style animates naturally
5. `exportToSvg` generates SVG → **morphdom** diffs against existing DOM
6. morphdom preserves existing elements (no re-animation), only new elements trigger CSS animations

### Final render (`ontoolinput`)
1. Parses complete JSON, renders with **original seeds** (stable final look)
2. Same `exportToSvg` + morphdom path — seamless transition, no blink
3. Sends PNG screenshot to model context (debounced 1.5s)

### CSS Animations (3 layers)
- **Shapes** (`g, rect, circle, ellipse, text, image`): opacity fade-in 0.5s
- **Lines** (`path, line, polyline, polygon`): stroke-dashoffset draw-on effect 0.6s
- **Existing elements**: smooth `transition` on fill/stroke/opacity changes

### Key Libraries
- **morphdom**: DOM diffing for SVG — preserves existing nodes, only new nodes get animations
- **exportToSvg**: Excalidraw's SVG export (with fonts inlined by default)

## Cheat Sheet: Progressive Element Ordering

The `server.ts` cheat sheet instructs the model to emit elements progressively:
- BAD: all rectangles → all texts → all arrows (blank boxes stream, then labels appear late)
- GOOD: background shapes first, then per node: shape → label → arrows → next node
- This way each node appears complete with its label during streaming

## Debugging

### Dev workflow
1. Edit source files
2. `npm run build` (or `npm run dev` for watch mode)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vtmade) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
