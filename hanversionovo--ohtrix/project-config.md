---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Motrix is a full-featured download manager (HTTP/FTP/BitTorrent/Magnet) built on **Electron + Vue 2 + Aria2**. This repo contains **two independent codebases**:

1. **The desktop app** (`src/`, `.electron-vue/`) — the actual Motrix application, built with Webpack/electron-builder. This is what `package.json` describes.
2. **A HarmonyOS (OpenHarmony) port** (`platform/`) — a separate DevEco Studio / `hvigor` project that runs the Electron app on HarmonyOS. It has its own build system, dependencies (`oh_modules/`), and language (ArkTS/ETS + C++). It does **not** share tooling with the desktop app.

Treat these as two projects. Changes to `src/` rarely touch `platform/` and vice-versa.

## Desktop App (`src/`)

### Commands

```bash
yarn                      # install deps (postinstall also runs electron-builder install-app-deps + lint:fix)
yarn run dev              # dev mode: webpack-dev-server (renderer, :9080) + main watch + electron --inspect=5858
yarn run build            # build main+renderer to dist/electron, then electron-builder → release/
yarn run build:applesilicon  # build + package arm64 mac
yarn run build:dir        # build unpacked (no installer) — fastest way to test a packaged build
yarn run build:web        # BUILD_TARGET=web, builds the web-only renderer bundle
yarn run lint             # eslint .js + .vue under src/
yarn run lint:fix         # eslint --fix
```

Node `>=16`. There is **no unit-test runner** for the desktop app — `lint` is the only automated check. Do not invent test commands.

### Process Model

Three top-level layers under `src/`:

- `src/main/` — Electron **main process** (Node). Webpack alias `@` → `src/main`.
- `src/renderer/` — **renderer** (Vue 2 UI). Webpack alias `@` → `src/renderer`.
- `src/shared/` — code imported by **both**. Alias `@shared` → `src/shared`.

> **Gotcha:** `@` resolves to a *different* directory in main vs. renderer code. When moving code between processes, fix the `@` imports. `@shared` is stable across both.

### Boot Sequence

`src/main/index.js` → `new Launcher()` → `new Application()`:

- **`Launcher.js`** enforces the single-instance lock and parses `argv` for a URL/file to download (deep-link / file-association entry points), then constructs `Application`.
- **`Application.js`** is the heart of the app. Its `init()` instantiates every manager in order and wires up IPC (`handleIpcMessages` for `ipcMain.on`, `handleIpcInvokes` for `ipcMain.handle`). When adding main-process features, you almost always touch `Application.js`.

### Manager Pattern

Main-process responsibilities are split into single-purpose manager classes:

- `src/main/core/` — non-UI managers: `ConfigManager`, `Engine`, `EngineClient`, `UPnPManager`, `UpdateManager`, `EnergyManager`, `ProtocolManager`, `AutoLaunchManager`, `Context`, `Logger`, `ExceptionHandler`.
- `src/main/ui/` — UI managers: `WindowManager`, `MenuManager`, `TrayManager`, `ThemeManager`, `DockManager`, `TouchBarManager`, `Locale`.

Follow this pattern for new main-process features rather than adding logic directly to `Application.js`.

### Aria2 Engine — the core dependency

Downloads are not handled in JS; they run in the bundled **`aria2c`** binary, controlled over JSON-RPC:

- **`core/Engine.js`** spawns `aria2c` as a child process (binary + conf resolved per platform/arch via `src/main/utils`), writing a PID file and loading/saving the session file. Engine config is assembled from `systemConfig` + `userConfig`.
- **`core/EngineClient.js`** is the *main-process* RPC client, built on the `Aria2` client from `@shared/aria2`.
- **`src/renderer/api/Api.js`** — the **renderer talks to aria2 RPC directly** (over WebSocket via `@shared/aria2`), not only through IPC. It fetches the RPC port/secret from main once via `ipcRenderer.invoke('get-app-config')`, then opens its own client. So download state flows renderer↔aria2 directly; IPC is mainly for app/window/config concerns.

**`src/shared/aria2/`** is the JSON-RPC WebSocket client used by both processes — the central communication primitive.

### Renderer

Vue 2 + VueX + Vue Router + Element UI. Entry: `src/renderer/pages/index/main.js`.

- `src/renderer/store/modules/` — VueX modules: `app`, `task`, `preference`.
- `src/renderer/api/` — the aria2 API wrapper (`api` singleton, used as `@/api`).
- `src/renderer/components/` — feature-grouped Vue components.
- SCSS auto-injects `@/components/Theme/Variables.scss` into every stylesheet (see `webpack.renderer.config.js` `additionalData`).

### i18n

Two parallel systems (see `CONTRIBUTING.md`):

- **Element UI** locales registered in `src/shared/locales/all.js`.
- **i18next** for menus + main interface, under `src/shared/locales/<locale>/`, split per business module (`app.js`, `task.js`, `menu.js`, …). Menu translations live with the locale files, not in `src/main/menus`.

### Build Toolchain (`.electron-vue/`)

Custom electron-vue setup (not Vue CLI / Vite):

- `dev-runner.js` — orchestrates dev mode (renderer dev-server + main watch/recompile + electron restart).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HanversionOvO/OHtrix](https://github.com/HanversionOvO/OHtrix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
