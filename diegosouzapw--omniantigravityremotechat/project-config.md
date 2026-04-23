---
trigger: always_on
description: > This file provides context for AI coding assistants (Antigravity, Copilot, Cursor, etc.) working on this project.
---

# AGENTS.md — AI Coding Assistant Instructions

> This file provides context for AI coding assistants (Antigravity, Copilot, Cursor, etc.) working on this project.

## Project Overview

**OmniAntigravity Remote Chat** (v1.3.0) is a premium mobile command center for Antigravity AI sessions.  
Architecture: Node.js server connects to Antigravity via Chrome DevTools Protocol (CDP), mirrors the chat UI to a mobile browser via WebSocket, and adds remote workspace tools, supervisor intelligence, quota monitoring, and tunnel-based remote access.

## Tech Stack

- **Runtime**: Node.js 22+ (ESM modules, `"type": "module"`)
- **Server**: Express.js 4.22, WebSocket (ws 8.x), Cookie-based auth, compression
- **Frontend**: Vanilla HTML/CSS/JS (no framework), 5 themes, PWA support
- **Protocol**: Chrome DevTools Protocol (CDP) for Antigravity communication
- **Testing**: Vitest 4.x (unit), custom smoke suite (integration)
- **Tunneling**: Cloudflare Quick Tunnels, Pinggy (SSH-based), ngrok (optional)
- **Notifications**: Telegram Bot (optional, lazy-loaded `node-telegram-bot-api`)
- **AI Supervisor**: OmniRoute OpenAI-compatible API (optional)
- **Ports**: CDP on 7800-7803 (configurable via `CDP_PORTS`), web server on 4747 (configurable via `PORT`)
- **Docker**: `node:22-alpine` with production-only deps

## Project Structure

```
src/
├── server.js               — Main server: Express + WebSocket + CDP actions (3400+ lines)
├── config.js               — Constants, env vars, feature flags, version
├── env.js                  — dotenv bootstrap (resolves .env from project root)
├── state.js                — Shared mutable state + JSDoc type definitions
├── supervisor.js           — AI supervisor (OmniRoute), suggest queue, heuristic safety
├── quota-service.js        — Model quota polling via language server HTTPS API
├── session-stats.js        — In-memory session analytics and metrics
├── screenshot-timeline.js  — Persistent screenshot capture with disk storage
├── ui_inspector.js         — UI DOM inspection utilities
├── cdp/
│   └── connection.js       — CDP discovery, WebSocket connect, context management
└── utils/
    ├── network.js          — getLocalIP, isLocalRequest, getJson
    ├── process.js          — killPortProcess, isPortFree, launchAntigravity
    ├── hash.js             — djb2 hash for snapshot diff detection
    ├── telegram.js         — Telegram bot: commands, inline keyboards, rate-limiting, threading
    └── workspace.js        — File browser, terminal manager, Git actions, quick commands, uploads

public/
├── index.html              — Main mobile chat interface
├── login.html              — Authentication page
├── admin.html              — Admin panel (metrics, logs, tunnel control)
├── minimal.html            — Lite mode for unstable connections
├── manifest.json           — PWA manifest
├── sw.js                   — Service worker
├── css/                    — Modular CSS: variables, themes (5), layout, components, workspace, assist, chat
├── js/
│   ├── app.js              — Main client logic (WebSocket, rendering, UI state)
│   ├── admin.js            — Admin panel logic
│   ├── login.js            — Login page logic
│   ├── minimal.js          — Lite mode logic
│   ├── theme-bootstrap.js  — Theme initialization (CSP-compliant external script)
│   └── components/         — Modular workspace panels (assist, files, git, stats, terminal, timeline)
└── icons/                  — SVG app icons (standard + maskable)

scripts/
├── cloudflare-tunnel.js    — Cloudflare Quick Tunnel manager
├── pinggy-tunnel.js        — Pinggy SSH tunnel manager
├── generate_ssl.js         — SSL certificate generator (OpenSSL + Node.js crypto fallback)
├── setup-ssl.js            — Interactive SSL setup with mkcert
├── install_context_menu.*  — OS context menu installers (Windows/Linux)
├── start.sh / start.bat    — Shell launchers (local)
├── start_web.sh / start_web.bat — Shell launchers (remote)
├── sync_features.py        — Feature sync utility
└── windows-wsl-remote/     — PowerShell scripts for WSL2 integration

test/
├── test.js                 — Integration/smoke test suite
└── unit/                   — Vitest unit tests (9 test files)
    ├── config.test.js
    ├── hash.test.js
    ├── network.test.js
    ├── quota-service.test.js
    ├── screenshot-timeline.test.js
    ├── session-stats.test.js
    ├── supervisor.test.js
    └── telegram.test.js

launcher.js                 — Node.js entry point (QR code, tunnel orchestration)
data/                       — Runtime data (quick-commands.json, screenshots/, uploads/)
docs/                       — Extended documentation (CODE_DOCUMENTATION, plans, competitive analysis)
```

## Key Patterns

- **ESM imports** — All `.js` files use `import/export`, not `require()`. Exceptions: `fix_ts.cjs` and `sync_wsl.cjs` (CommonJS utilities)
- **PROJECT_ROOT** — `src/config.js` uses `join(__dirname, '..')` as `PROJECT_ROOT` since it lives in `src/`
- **CDP connection** — `discoverCDP()` scans configurable ports (default 7800-7803) for Antigravity targets
- **Multi-window** — `discoverAllCDP()` returns all available CDP workbench targets across ports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diegosouzapw/OmniAntigravityRemoteChat](https://github.com/diegosouzapw/OmniAntigravityRemoteChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
