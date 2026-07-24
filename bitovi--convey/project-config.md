---
trigger: always_on
description: A browser overlay + inspector panel + MCP server for visually editing Tailwind CSS classes on a running React or Angular app. The user clicks elements in the page, and the panel lets them scrub/select new Tailwind values which are queued as changes for an AI agent to apply.
---

# Convey MCP — Copilot Instructions

## Project Overview

A browser overlay + inspector panel + MCP server for visually editing Tailwind CSS classes on a running React or Angular app. The user clicks elements in the page, and the panel lets them scrub/select new Tailwind values which are queued as changes for an AI agent to apply.

**Stack:** TypeScript, React 18, Angular 21, Vite 5, Tailwind v4 (CSS-only `@theme`), Express + WebSocket, MCP SDK, Vitest, Storybook 8 + 10, esbuild.

## Key Design Tokens (Bitovi brand)

Used throughout panel UI — do not substitute generic colors:

```
bit-teal:      #00848B
bit-orange:    #F5532D
bit-surface:   panel background
bit-surface-hi: elevated surface
bit-bg:        page background
bit-border:    border color
bit-text:      primary text
bit-text-mid:  secondary text
bit-muted:     muted/disabled text
```

## Code Conventions

- **TypeScript everywhere** — no plain JS files in `overlay/`, `panel/`, or `server/`
- **No `@/` path alias** — use relative imports (`../../overlay/src/class-parser`)
- **No `cn()` utility** — use template literals for conditional classes
- **No default exports for components** — use named exports
- **React function components only** — no class components

## Running Everything

### Preferred: VS Code Tasks (run individually for full visibility)

Start each process as a separate task via **Terminal → Run Task** so you can see each terminal independently:

1. **Watch: Overlay** — esbuild `--watch`, rebuilds `overlay/dist/overlay.js` on every save
2. **Watch: Panel** — `vite build --watch`, rebuilds `panel/dist/` on every save
3. **Mock MCP Client** — spawns **its own MCP server** (port 3333) via stdio, so no separate server task is needed. Loops: `implement_next_change` → 2s simulated work → `mark_change_implemented` → repeat. Stage + commit patches in the panel and watch the mock agent pick them up.
4. **Test App (port 5173)** — Vite dev server for the test app

> **Tip:** The Mock MCP Client prints exactly what an AI agent receives from `implement_next_change`, making it easy to observe the full loop end-to-end.

### ⚠️ Mock MCP Client owns the server lifecycle

The Mock MCP Client spawns `server/index.ts` via stdio from `test-app/`, which is why the server resolves the correct `tailwindcss` package. Do not start the **Server (port 3333)** task separately when using the Mock MCP Client — only start it if you need a standalone server without the mock agent.

```bash
# If running manually (not via VS Code tasks):
# 1. Build once first
npm run build

# 2. Start the watch processes
npx esbuild overlay/src/index.ts --bundle --format=iife --outfile=overlay/dist/overlay.js --platform=browser --watch &
cd panel && npx vite build --watch &

# 3. Start the Mock MCP Client (spawns the server automatically, from test-app/)
cd test-app && npx tsx mock-mcp-client.ts
# → Server at http://localhost:3333/panel/

# 4. Start the test app
cd test-app && npx vite
# → http://localhost:5173 (page to inspect)

# 5. Storybook (optional)
cd panel && npm run storybook          # → http://localhost:6006 (panel components)
cd storybook-test/v8 && npm run storybook   # → http://localhost:6007 (test-app stories, SB8)
cd storybook-test/v10 && npm run storybook  # → http://localhost:6008 (test-app stories, SB10)
```

## Running Tests

```bash
# All panel tests (from panel/ or root)
cd panel && npm test
# or from root:
npm test

# Watch mode
cd panel && npm run test:watch

# Run specific test file
cd panel && npm test -- ScaleScrubber
```

## Package Structure

```
/                        ← root package (server + build scripts)
  server/index.ts        ← Express + WebSocket + MCP server (port 3333)
  server/tailwind.ts     ← Tailwind v4 compiler (uses target project's tailwindcss)
  server/queue.ts        ← Change queue for MCP tools
  overlay/src/           ← Browser overlay (esbuild IIFE bundle)
    class-parser.ts      ← Parses Tailwind classes → ParsedClass (shared with panel)
    index.ts             ← Shadow DOM, toggle button, click handler
    picker.ts            ← Old picker UI (being replaced by panel)
    ws.ts                ← WebSocket client
  panel/                 ← React inspector panel (separate package)
    src/App.tsx          ← Root component
    src/Picker.tsx       ← Main inspector UI (property chips, scrubbers)
    src/ws.ts            ← WebSocket client + message bus
    src/components/      ← Modlet-style components
  storybook-addon/       ← Storybook addon (SB8 + SB10 compatible)
    manager.tsx          ← Addon panel for SB8 (embeds inspector via iframe)
    manager-v10.tsx      ← Addon panel for SB10 (storybook/* imports)
    preview.ts           ← Decorator for SB8 (injects overlay.js into stories)
    preview-v10.ts       ← Decorator for SB10 (storybook/* imports)
    preset.js            ← Storybook preset hooks (auto-detects SB version)
  storybook-test/          ← Storybook integration test environments
    v8/                  ← Storybook 8 (port 6007) — stories from test-app

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitovi/convey](https://github.com/bitovi/convey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
