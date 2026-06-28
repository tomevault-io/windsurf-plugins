---
trigger: always_on
description: This file is written for AI coding agents. It assumes no prior knowledge of the project.
---

# Agent Guide: browser-bridge-mcp

This file is written for AI coding agents. It assumes no prior knowledge of the project.

## Shell convention

- Use `fish` as the user's shell for this project. Prefer fish-compatible commands and syntax when shell behavior matters.

## Project overview

`browser-bridge-mcp` is a local demo / integration harness for bidirectional Browser ↔ AI communication using Playwright and a Chrome extension. It provides:

- A simple Vite-served demo web page (`index.html`) with randomized system-status widgets, buttons that deliberately trigger console errors and 404 network requests, and an intentionally broken layout section. This is designed to produce interesting Playwright/MCP capture output.
- A Chrome extension that lets the user click any page element and send its selector and context to a local Node.js receiver.
- A Node.js selection receiver server (`scripts/selection-server.mjs`) that persists the picked element to `<outputDir>/<hostname>/last-selected-element.json` and archives past selections under `<outputDir>/<hostname>/history/`.
- A Browser Bridge MCP server (`scripts/bridge-mcp-server.mjs`) that exposes the latest selection and history to any MCP-compatible AI.
- A Playwright capture script (`scripts/capture-context.mjs`) that can screenshot and inspect either a URL or the last selected element.
- An MCP server configuration (`.vscode/mcp.json`) that wires Playwright and Browser Bridge MCP tools to the IDE/chat client.

The project is intentionally minimal: plain HTML/CSS/JS, no frontend framework, no test runner, no production build, and no CI/CD.

## Technology stack

- **Runtime / package manager:** Node.js, npm, ES Modules.
- **Web app dev server:** Vite 6 (default config; no `vite.config` file exists).
- **Frontend:** HTML5, CSS3, vanilla JavaScript.
- **Browser automation:** Playwright (Chromium).
- **MCP server:** `@executeautomation/playwright-mcp-server`.
- **Browser extension:** Chrome Manifest V3 with service worker, content script, and popup.
- **Local receiver:** Plain Node.js `http` server.

## Directory structure

```
/home/arme/Projects/browser-bridge-mcp
├── AGENTS.md                 # Agent instructions (this file)
├── BROWSER_BRIDGE.md         # Human-readable architecture & setup guide
├── docs/
│   └── BROWSER_BRIDGE_EXTENSION_USAGE.md  # How to use the extension with any MCP-compatible AI
├── package.json              # npm scripts and dependencies
├── package-lock.json
├── index.html                # Vite-served demo web app
├── main.js                   # Demo app logic
├── style.css                 # Demo styles
├── start-local.sh            # Start the Vite demo web app
├── start-bridge.sh           # Start the MCP + selection receiver servers
├── stop-bridge.sh            # Stop the bridge servers started by start-bridge.sh
├── .vscode/
│   ├── mcp.json              # MCP server definition (Playwright + Browser Bridge, stdio)
│   └── settings.json         # MCP access / discovery settings
├── extension/                # Unpacked Chrome extension
│   ├── manifest.json         # Manifest V3
│   ├── background.js         # Service worker icon-click handler
│   ├── content.js            # Element picker, selector generation, sender
│   ├── popup.html            # Extension popup markup
│   ├── popup.js              # Popup toggle/status logic
│   └── icons/                # icon16/32/48/128.png
├── scripts/
│   ├── selection-server.mjs  # HTTP receiver on port 8932
│   └── capture-context.mjs   # Playwright capture/reporting script
├── output/                   # Generated artifacts
│   ├── last-selected-element.json
│   ├── *-context.json
│   ├── *-screenshot.png
│   └── mcp-screenshots/
├── .pids/                    # PID files for background bridge servers
└── .logs/                    # Logs for background bridge servers
```

## Runtime architecture

The project is meant to run as several co-located processes plus a browser extension:

1. **Vite dev server** — `http://127.0.0.1:5173` — serves the demo web app.
2. **Playwright MCP server** — `BROWSER_BRIDGE.md` shows `http://localhost:8931/mcp` when started with `--port 8931`; `.vscode/mcp.json` uses stdio transport instead.
3. **Browser Bridge MCP server** — stdio MCP server (`scripts/bridge-mcp-server.mjs`) that exposes `bridge://latest`, `bridge://history`, and related tools.
4. **Selection receiver server** — `http://127.0.0.1:8932` — receives element selections from the extension.
5. **Chrome extension** — loaded unpacked from `extension/`, injects a content-script picker into any page.

Data flow:

- **AI → Browser:** An MCP client uses Playwright tools to navigate, click, screenshot, and inspect pages.
- **Browser → AI:** The extension captures a clicked element, generates a selector, and POSTs JSON to the selection server. The server writes `output/last-selected-element.json`, and `npm run capture:last-selected` can read it back with Playwright.

### Key runtime files

- **`index.html` / `main.js` / `style.css`** — Demo single-page app.
  - Random CPU/memory values update every 5 seconds.
  - `btn-fetch` shows mocked JSON in `#result-area`.
  - `btn-error` deliberately logs errors/warnings and throws a caught exception.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arayaphong/browser-bridge-mcp](https://github.com/arayaphong/browser-bridge-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
