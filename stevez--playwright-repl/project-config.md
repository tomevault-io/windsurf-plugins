---
trigger: always_on
description: **playwright-repl** is an interactive REPL (Read-Eval-Print Loop) for browser automation that runs Playwright's `BrowserServerBackend` in-process via an `Engine` class. No daemon, no socket — commands execute directly.
---

# CLAUDE.md — Context for Claude Code

## Project Overview

**playwright-repl** is an interactive REPL (Read-Eval-Print Loop) for browser automation that runs Playwright's `BrowserServerBackend` in-process via an `Engine` class. No daemon, no socket — commands execute directly.

Think of it as a **keyword-driven test runner** (like Robot Framework) backed by Playwright instead of Selenium.

## Monorepo Structure

```
playwright-repl/
├── package.json                    # Root workspace config (npm workspaces)
├── tsconfig.base.json              # Shared TypeScript compiler options
├── packages/
│   ├── core/                       # Shared engine + utilities (TypeScript, tsc)
│   │   ├── src/
│   │   │   ├── engine.ts           # Wraps BrowserServerBackend in-process
│   │   │   ├── parser.ts           # Command parsing + alias resolution
│   │   │   ├── page-scripts.ts     # Text locators + assertion helpers
│   │   │   ├── completion-data.ts  # Ghost completion items
│   │   │   ├── extension-server.ts # HTTP server for extension commands
│   │   │   ├── colors.ts           # ANSI color helpers
│   │   │   └── resolve.ts          # COMMANDS map, minimist re-export
│   │   ├── dist/                   # Compiled output (gitignored)
│   │   └── test/
│   │
│   ├── cli/                        # Terminal REPL (published as "playwright-repl", TypeScript, tsc)
│   │   ├── src/
│   │   │   ├── playwright-repl.ts  # CLI entry point (compiles to dist/)
│   │   │   ├── repl.ts             # Interactive readline loop
│   │   │   ├── recorder.ts         # Session recording/replay
│   │   │   └── index.ts            # Public API exports
│   │   ├── dist/                   # Compiled output (gitignored)
│   │   ├── test/
│   │   └── examples/               # .pw session files
│   │
│   └── extension/                  # Chrome side panel extension (React, Vite, Tailwind)
│       ├── public/
│       │   └── manifest.json       # Manifest V3 config (copied to dist/ by Vite)
│       ├── src/
│       │   ├── background.ts       # Side panel behavior + recording handlers
│       │   ├── panel/              # Side panel UI (React)
│       │   │   ├── panel.html
│       │   │   ├── panel.tsx       # React entry point
│       │   │   ├── panel.css       # Theme variables + residual styles
│       │   │   ├── App.tsx         # Root component
│       │   │   ├── reducer.ts      # useReducer state management
│       │   │   ├── components/     # Toolbar, EditorPane, ConsolePane, etc.
│       │   │   ├── hooks/          # useCommandHistory
│       │   │   └── lib/            # server, run, autocomplete, filter, etc.
│       │   └── content/
│       │       └── recorder.ts     # Event recorder injected into pages
│       ├── dist/                   # Vite build output (gitignored, loaded by Chrome)
│       ├── vite.config.ts          # Vite build config (3 entry points)
│       ├── test/                   # Vitest browser component tests
│       └── e2e/                    # Playwright E2E tests
```

## Architecture

### The Three Layers

```
Layer 1: CLI keyword → MCP tool name (commands.js from Playwright)
  "click"     → "browser_click"
  "goto"      → "browser_navigate"
  "press"     → "browser_press_key"
  "fill"      → "browser_type"
  "snapshot"  → "browser_snapshot"

Layer 2: MCP tool name → handle() function (tools/*.js in Playwright)
  backend.callTool("browser_click", { ref: "e5" })
  → looks up tool in registry → calls handle()

Layer 3: handle() → Playwright API
  browser_click  → locator.click()
  browser_navigate → page.goto()
  browser_press_key → page.keyboard.press()
  browser_snapshot → accessibility tree walk via CDP
```

### Full Command Flow

```
User types:  "click e5"
  ↓ alias resolution (REPL layer)
tokens:      ["click", "e5"]
  ↓ minimist parsing
args:        { _: ["click", "e5"] }
  ↓ Engine.run(args)
  ↓ parseCommand(command, args)
toolName:    "browser_click"
toolParams:  { ref: "e5" }
  ↓ backend.callTool(toolName, toolParams)
  ↓ Playwright API
browser:     locator.click()
  ↓ CDP WebSocket
Chrome:      actual DOM click event
```

### EvaluateConnection (packages/core/src/evaluate-connection.ts)

The primary execution mode. Launches Chromium with the Dramaturg extension and executes commands via `serviceWorker.evaluate()`:

```js
const conn = new EvaluateConnection();
await conn.start(extensionPath, { headed: true, chromium });
const result = await conn.run('click e5');
// result = { text: 'Clicked', isError: false }
await conn.close();
```

No WebSocket bridge, no port management. Playwright talks directly to the extension's service worker.

### Engine (packages/cli/src/engine.ts)

Fallback mode when the extension is not available. Wraps Playwright's `BrowserServerBackend` in-process. Only supports keyword commands (no JavaScript).

### BridgeServer (packages/core/src/bridge-server.ts)

WebSocket server for `--bridge` mode — connects to the user's existing Chrome with Dramaturg installed. Used when the user wants to automate their real browser session with cookies/auth intact.

### Element Refs (e1, e5, etc.)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stevez/playwright-repl](https://github.com/stevez/playwright-repl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
