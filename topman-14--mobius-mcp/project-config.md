---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Setup

```bash
npm install
```

Root-only. This is an npm workspaces monorepo (`apps/*`, `packages/*`) — installing inside an individual workspace folder fights the root's symlinked `node_modules` and is never necessary.

### Build

```bash
npm run build
```

Builds every workspace in dependency order: `packages/capture-core` → `apps/mcp-server` / `apps/npm-client` / `apps/browser-extension`. The apps consume `packages/capture-core`'s built `dist/` output, not its TypeScript source — after editing it, rebuild before an app will see the change (or use watch mode).

Build one workspace:

```bash
npm run build -w packages/capture-core
npm run build -w apps/mcp-server
```

### Watch mode (active development)

```bash
npm run watch
```

Does a one-time build of `packages/capture-core`, then runs three labeled watchers in parallel: `[packages]` (`tsc -b --watch`), the extension's Vite dev server + esbuild watcher for the standalone content-script/injected bundles, and `apps/mcp-server`'s `tsup --watch`. Load the extension once via `chrome://extensions` → Developer Mode → Load unpacked → `apps/browser-extension/dist`; background/popup/options changes hot-reload via crxjs, but `content-script.ts`/`injected.ts` changes only take effect the next time they're injected (reload the target tab or toggle capture off/on).

If only the shared packages need rebuilding (e.g. working on `apps/npm-client` or `apps/mcp-server` without the extension watchers):

```bash
npm run watch -w packages/capture-core
```

### Run the MCP server from source

```bash
npm run start --workspace=apps/mcp-server
```

### Test / lint

`npm run test` and `npm run lint` exist at the root (`--workspaces --if-present`) and run in CI, but no workspace currently defines a `test` or `lint` script — there is no test suite or lint config in this repo yet.

## Architecture

One shared package implements one pipeline: a browser client captures runtime events and streams them over a local WebSocket to a Node server, which exposes them to AI agents as MCP tools over stdio.

```
apps/browser-extension/   Chrome MV3 extension — captures events in the page, streams over WS
apps/npm-client/          Lightweight alternative to the extension for direct app integration (paused, no CDP)
apps/mcp-server/          WS hub + MCP tool server (stdio)
packages/capture-core/    Shared capture logic (console/network/error/navigation patching + redaction)
                           and the wire-protocol types (ClientMessage/ServerMessage/ControlMessage, event
                           schema) every workspace above compiles against
skills/                   Scenario-focused agent skills (dead clicks, silent API failures, contract drift, etc.)
```

### Capture pipeline (extension)

Three separate JS execution contexts are involved per tab, because an isolated-world content script cannot patch the page's real `window.fetch`/`console`:

1. **`apps/browser-extension/injected.ts`** runs in the page's MAIN world. It calls `startCapture()` (`packages/capture-core/src/index.ts`), which monkey-patches `console.*`, `window.fetch`/`XMLHttpRequest`, `history.pushState`/`replaceState`, and (opt-in) a `MutationObserver`. Captured events are relayed out via `window.postMessage`.
2. **`apps/browser-extension/src/content-script.ts`** runs in the isolated world. It relays `postMessage`d events to the background worker via `chrome.runtime.sendMessage`, and answers the injected script's `"ready"` handshake with capture/privacy settings read from `chrome.storage` (the MAIN-world script has no storage access).
3. **`apps/browser-extension/src/background.ts`** (MV3 service worker) is the hub: `enableTab()` injects both scripts (`chrome.scripting.executeScript`, one in `world: "MAIN"`), owns the WebSocket connection to `apps/mcp-server`, and relays received events onto it. It also handles inbound `"command"` messages from the server (screenshots, `evaluate_js`, CDP calls via `src/lib/cdp.ts`) for tools that need live browser control rather than just stored events.

Auto-enable per site is rule-based (`src/lib/rules.ts` + `src/lib/host-permissions.ts`), evaluated on `chrome.webNavigation.onCommitted` — capture is opt-in per tab otherwise (popup toggle).

### mcp-server

- **`src/wsServer.ts`** — the WebSocket hub browser clients connect to (`ws://localhost:7331` by default, `CONSOLE_STREAM_PORT` env override).
- **`src/store.ts`** (`EventStore`/`TabBuffer`) — per-tab in-memory ring buffer (capped, oldest evicted), with a single `seq` counter shared across all tabs so `get_logs_since` cursors stay globally orderable.
- **`src/registry.ts`** (`ClientRegistry`) — tracks connected clients; keeps a disconnected client around for a grace period (`PURGE_DELAY_MS`) so a page refresh doesn't instantly wipe its history.
- **`src/commandDispatcher.ts`** — request/reply RPC over the same WS connection (`sendCommand` + matching `ack`), used for anything that isn't passive event capture (navigation, screenshots, `evaluate_js`, profiling).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Topman-14/mobius-mcp](https://github.com/Topman-14/mobius-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
