---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**FoxPilot** — AI browser automation via MCP. npm package `foxpilot-mcp`; repo `balakumardev/foxpilot`.

## Commands

```bash
npm install                          # all deps (postinstall installs every subproject)
npm run build                        # build all via nx
cd mcp-server && npm run build        # individual builds (also: firefox-extension, chrome-extension)
cd firefox-extension && npx jest      # tests (chrome-extension also has a jest suite)
cd chrome-extension && npx jest
cd mcp-server && npm start            # start MCP server (auto-starts the broker)
cd mcp-server && npm run pack-dxt     # package the .dxt
npm run package --prefix chrome-extension   # build + zip -> chrome-extension/web-ext-artifacts/foxpilot-chrome-<v>.zip
```

## Architecture

Monorepo, five projects:
1. **mcp-server** — Node MCP server (stdio to the client) plus the **broker** (`broker-main.ts` / `broker.ts` / `broker-protocol.ts`).
2. **firefox-extension** — MV2 (background script + `browser.tabs.executeScript`).
3. **chrome-extension** — MV3 (service worker + offscreen document + content-script messaging).
4. **common** — shared message interfaces (`@foxpilot/common`).
5. **input-sidecar** — native-input helper.

### Communication flow
- Client ↔ MCP server: MCP over stdio.
- MCP server ↔ extension: through a **broker** on port 8089 (WebSocket). The broker is a separate, persistent process that holds the extension connection; multiple browsers (Chrome/Firefox) can connect and one is the active driver (`list-browsers` / `select-browser`).
- Auth: zero-config by default. The broker admits an extension based on the connection being loopback with a `chrome-extension://`/`moz-extension://` Origin — no user-typed secret. The control leg (MCP server ↔ broker) is signed with an auto-managed secret the mcp-server persists at **`~/.foxpilot/control-secret`** (mode 0600, created on first run, all server instances converge on it). A manual `EXTENSION_SECRET` is only set for remote/CONTAINERIZED deployments (where loopback+Origin gating doesn't apply); the same secret must then be set in the extension's Advanced settings. The broker binds both IPv4 and IPv6 loopback. `EXTENSION_PORT` defaults to 8089.
- Wire-protocol frames on `/extension` (post-zero-config): broker sends an explicit **`welcome`** ack on accept (typed **`rejected`** reason on refusal); the connection status pill flips "Connected" only on this ack, not on socket-open (the old socket-open-flip was the "Connected ↔ Disconnected flicker on wrong secret" failure mode). A **`healthcheck`** frame round-trips the broker's `/health` snapshot (server up, `extensionConnected`, browser count, active or standby) and powers the **Test Connection** button.

### Key files
- `mcp-server/server.ts` — tool definitions; `mcp-server/broker.ts` — broker.
- `firefox-extension/message-handler.ts`, `chrome-extension/message-handler.ts` — execute browser actions. Both have a `switch(req.cmd)` whose `default:` uses a `const _exhaustiveCheck: never = req` tripwire — adding a `cmd` to the `ServerMessage` union forces a matching case or compile fails.
- `firefox-extension/injected/*` — functions stringified and run in the page (`action-script`, `snapshot-script`, `page-world`, `upload-script`, `point-action-script`, `screenshot-script`). Mirrored in `chrome-extension/injected/*`. **For new modules, the function bodies (or trailing code past `export function`) must be byte-identical between the two extensions** — header / doc-comment lines are allowed to diverge. Mirror manually. `self-containment.test.ts` only checks the FIREFOX copies are stringify-safe — `mirror.test.ts` is what actually compares the two extensions (`page-world.ts` excluded; see the gotcha below).
- `chrome-extension/cdp-input.ts` — refcounted `chrome.debugger` attach by `purpose: "network" | "input"` (Phase 3, v1.0.13). Detaches only when the last purpose releases; `capture-response-bodies` defaults purpose to `"network"` so its tests/exports are unchanged. Coexist with `network-capture.ts`.
- `mcp-server/point-format.ts`, `mcp-server/network-format.ts`, `mcp-server/snapshot-format.ts` — server-side formatters for the new tools (unit-tested in isolation).
- `firefox-extension/browser-http.ts`, `chrome-extension/browser-http.ts` — background-context HTTP/cookie/stream logic for the privileged tools (auto-bundled via `background.ts` import chain).
- `common/server-messages.ts`, `common/extension-messages.ts` — message types.

## Gotchas (these cost real time)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [balakumardev/foxpilot](https://github.com/balakumardev/foxpilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
