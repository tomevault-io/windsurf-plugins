---
trigger: always_on
description: Handles dynamic bundling of scope extensions:
---

# Browser Canvas Development Guide

## Project Structure

```
browser-canvas/
├── src/
│   ├── client.ts                # CanvasClient API for operations
│   ├── server/
│   │   ├── index.ts             # Bun + Hono server entry
│   │   ├── watcher.ts           # File system watcher (chokidar)
│   │   ├── websocket.ts         # WebSocket handler
│   │   ├── canvas-manager.ts    # Canvas instance state
│   │   ├── vanilla.ts           # Vanilla mode: bridge injection, detection
│   │   ├── component-loader.ts  # Reusable component loading
│   │   ├── scope-builder.ts     # Dynamic scope bundling
│   │   ├── tailwind-builder.ts  # Dynamic CSS building
│   │   ├── validators.ts        # ESLint, scope, Tailwind validation
│   │   ├── validation-status.ts # Validation state for status API
│   │   └── log.ts               # Unified log system
│   └── browser/
│       ├── index.html           # React canvas shell
│       ├── app.tsx              # React app with react-runner
│       ├── scope.ts             # Pre-bundled dependencies
│       ├── event-bridge.ts      # window.canvasEmit implementation
│       ├── use-canvas-state.ts  # Two-way state sync hook
│       ├── linter.ts            # axe-core, overflow, image validation
│       └── components/ui/       # shadcn components
├── vanilla/
│   └── base.css                 # CSS variables and utilities for vanilla mode
├── skills/
│   └── canvas/
│       ├── SKILL.md             # Claude instructions
│       ├── components/          # Reusable components (auto-loaded)
│       └── references/          # Component documentation
├── hooks/
│   ├── hooks.json               # PostToolUse hook config
│   └── canvas-validation.sh     # Validation feedback injection
├── dist/                        # Built output (gitignored)
├── server.sh                    # Start server script
└── package.json
```

## Key Architecture

### Dual-Mode Support

The server auto-detects canvas mode by filename:

| File | Mode | Stack |
|------|------|-------|
| `App.jsx` | React | shadcn/ui + Tailwind + React hooks |
| `index.html` | Vanilla | Pure HTML/CSS/JS, CSS variables |

Both modes share the same file protocol, WebSocket bridge, and API.

### File-Based Protocol

Claude interacts via filesystem for data:

| Claude Action | File Operation |
|---------------|----------------|
| Create React canvas | Write `App.jsx` to new folder |
| Create Vanilla canvas | Write `index.html` to new folder |
| Update canvas | Edit `App.jsx` or `index.html` |
| Read log | Read `_log.jsonl` (grep for type/severity/category) |
| Read/write state | Read/write `_state.json` |

### TypeScript API (Operations)

Use `CanvasClient` for operations:

```typescript
import { CanvasClient } from "browser-canvas"
const client = await CanvasClient.fromServerJson()

await client.screenshot("my-app")  // Take screenshot
await client.close("my-app")       // Close canvas
await client.getState("my-app")    // Get state (alternative to file)
await client.setState("my-app", { step: 2 })  // Set state
await client.getStatus("my-app")   // Get validation status
await client.health()              // Check server health
```

### Server ↔ Browser Protocol (WebSocket)

```typescript
// Server → Browser
{ type: "reload", code: string }  // React: includes code; Vanilla: code omitted
{ type: "screenshot-request" }

// Browser → Server
{ type: "event", event: string, data: unknown }
{ type: "screenshot", dataUrl: string }
{ type: "error", message: string }
{ type: "set-state", state: Record<string, unknown> }
```

### Vanilla Mode Architecture

Vanilla canvases serve `index.html` with an injected bridge script:

```
src/server/vanilla.ts
├── detectCanvasMode()      # Check for App.jsx vs index.html
├── readVanillaCanvas()     # Read index.html from disk
└── injectVanillaBridge()   # Inject WebSocket bridge into HTML
```

The bridge provides:
- `window.canvasEmit(event, data)` - Send events to log
- `window.canvasState(newState?)` - Get/set two-way state
- Auto-reconnecting WebSocket
- Hot-reload on file change (full page refresh)

Styling via `/base.css` route serving `vanilla/base.css` with CSS variables.

### PostToolUse Hook (Validation Feedback)

The plugin includes a PostToolUse hook that automatically injects validation errors after Write/Edit operations on `App.jsx` files:

```
hooks/
├── hooks.json               # Hook configuration
└── canvas-validation.sh     # Fetches validation status, formats output
```

Flow:
1. Claude writes/edits `App.jsx`
2. Server validates (ESLint, scope, Tailwind)
3. Hook calls `/api/canvas/:id/status?wait=true`
4. Validation errors injected via `additionalContext`

The status API supports waiting for pending validation:
```
GET /api/canvas/:id/status?wait=true&timeout=2000
```

## Development Commands

```bash
bun install          # Install dependencies
bun run dev          # Start dev server with watch
bun run build        # Build browser bundle
bun run typecheck    # Type check
```

## Adding shadcn Components

1. Copy component to `src/browser/components/ui/`
2. Export from `src/browser/scope.ts`
3. Document in `skills/canvas/references/components.md`
4. Rebuild: `bun run build`

## Adding Reusable Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parkerhancock/browser-canvas](https://github.com/parkerhancock/browser-canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
