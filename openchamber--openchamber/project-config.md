---
trigger: always_on
description: This skill contains all color tokens, semantic logic, decision tree, and usage patterns. All UI colors must use theme tokens - never hardcoded values or Tailwind color classes.
---

# OpenChamber - AI Agent Reference (verified)

## Core purpose

OpenChamber provides UI runtimes (web/desktop/VS Code) for interacting with an OpenCode server (local auto-start or remote URL). UI uses HTTP + SSE via `@opencode-ai/sdk`.

## Runtime architecture (IMPORTANT)

- `Desktop` (Electron) boots the web server **in the same Node process** as the Electron main, then loads the web UI from `http://127.0.0.1:<port>`. No sidecar subprocess.
- `Desktop` (Tauri, legacy) still spawns `openchamber-server` as a bun-compiled sidecar binary. Kept only for auto-update compatibility with existing Tauri installs.
- All backend logic lives in `packages/web/server/*` (and `packages/vscode/*` for the VS Code runtime). The native shell is not a feature backend.
- The shell is used only for stable native integrations: menu, dialog (open folder), notifications, updater, deep-links, quit confirmation.

### Desktop shell: Electron is the target, Tauri is legacy

- **New desktop work goes into `packages/electron/`.** This is the forward path.
- `packages/desktop/` (Tauri) is kept running in parallel only to preserve auto-update for existing installs until the cutover. Do **not** add features to it; do **not** port bug fixes back unless they actually affect currently-released Tauri users.
- Desktop-side changes (IPC handlers, native integrations, window/quit/notification behavior) land in `packages/electron/main.mjs` + `packages/electron/preload.mjs`. The `__TAURI__` shim exposed by the preload keeps the shared UI working against both shells, so renderer-side code should not branch on shell type.
- Electron imports the server via `@openchamber/web/server/index.js` (workspace dep) and calls `startWebUiServer({...})`. The returned handle has `getPort()` / `stop()`. Notifications flow via an `onDesktopNotification` callback injected at startup — no stdout-parsing IPC.
- Build/release: both shells ship in the same GitHub release today (`.github/workflows/release.yml`). The one-shot Tauri → Electron auto-update migration is documented in `docs/TAURI_TO_ELECTRON_CUTOVER.md`; run that when the user decides to flip.
- After the cutover ships and stabilises, `packages/desktop/` is deleted; this note collapses back to "Desktop is Electron".

## Tech stack (source of truth: `package.json`, resolved: `bun.lock`)

- Runtime/tooling: Bun (`package.json` `packageManager`), Node >=20 (`package.json` `engines`)
- UI: React, TypeScript, Vite, Tailwind v4
- State: Zustand (`packages/ui/src/stores/`)
- UI primitives: Base UI (`@base-ui/react`, primary source for dropdown/select/dialog/menu/tooltip/etc. — wrappers live in `packages/ui/src/components/ui/`), Radix UI (`package.json` deps, legacy usages being migrated), HeroUI (`package.json` deps), Remixicon (`package.json` deps)
- Server: Express (`packages/web/server/index.js`)
- Desktop (forward): Electron 41 (`packages/electron/`)
- Desktop (legacy, maintenance-only): Tauri v2 (`packages/desktop/src-tauri/`)
- VS Code: extension + webview (`packages/vscode/`)

## Monorepo layout

Workspaces are `packages/*` (see `package.json`).

- Shared UI: `packages/ui`
- Web app + server + CLI: `packages/web`
- Desktop shell (Electron — forward): `packages/electron`
- Desktop shell (Tauri — legacy, maintenance-only): `packages/desktop`
- VS Code extension: `packages/vscode`

## Documentation map

Before changing any mapped module, read its module documentation first.

### web

Web runtime and server implementation for OpenChamber.

#### lib

Server-side integration modules used by API routes and runtime services.

##### quota

Quota provider registry, dispatch, and provider integrations for usage endpoints.

- Module docs: `packages/web/server/lib/quota/DOCUMENTATION.md`

##### git

Git repository operations for the web server runtime.

- Module docs: `packages/web/server/lib/git/DOCUMENTATION.md`

##### github

GitHub authentication, OAuth device flow, Octokit client factory, and repository URL parsing.

- Module docs: `packages/web/server/lib/github/DOCUMENTATION.md`

##### opencode

OpenCode server integration utilities including config management, provider authentication, and UI authentication.

- Module docs: `packages/web/server/lib/opencode/DOCUMENTATION.md`

##### notifications

Notification message preparation utilities for system notifications, including text truncation and optional summarization.

- Module docs: `packages/web/server/lib/notifications/DOCUMENTATION.md`

##### terminal

WebSocket protocol utilities for terminal input handling including message normalization, control frame parsing, and rate limiting.

- Module docs: `packages/web/server/lib/terminal/DOCUMENTATION.md`

##### tts

Server-side text-to-speech services and summarization helpers for `/api/tts/*` endpoints.

- Module docs: `packages/web/server/lib/tts/DOCUMENTATION.md`

##### skills-catalog

Skills catalog management including discovery, installation, and configuration of agent skill packages.

- Module docs: `packages/web/server/lib/skills-catalog/DOCUMENTATION.md`

## Build / dev commands (verified)

All scripts are in `package.json`.

- Validate: `bun run type-check`, `bun run lint`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openchamber/openchamber](https://github.com/openchamber/openchamber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
