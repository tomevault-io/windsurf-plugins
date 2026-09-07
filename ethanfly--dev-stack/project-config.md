---
trigger: always_on
description: Windows-only Electron + Vue 3 + TypeScript desktop app (**DevStack**) that installs and switches local PHP / MySQL / Nginx / Redis / Node.js / Go / Python / Rust / Git, plus sites, hosts, and logs. Product UI and most comments are Chinese. User-visible name lives in `src/brand.ts` (`APP_NAME`).
---

# AGENTS.md

Windows-only Electron + Vue 3 + TypeScript desktop app (**DevStack**) that installs and switches local PHP / MySQL / Nginx / Redis / Node.js / Go / Python / Rust / Git, plus sites, hosts, and logs. Product UI and most comments are Chinese. User-visible name lives in `src/brand.ts` (`APP_NAME`).

Do not treat this as a web app. There is no public HTTP UI to verify in a browser.

## Commands

```bash
npm install
npm run electron:dev    # Vite + Electron (same as npm run dev)
npm run typecheck       # vue-tsc --noEmit
npm run build:nobump    # pack without bumping package.json
```

- `npm run build` / `build:patch|minor|major` **rewrite `package.json` version** and `public/version.json`. Do not run them unless the user asked to release.
- Output installer/portable builds land in `release/` (gitignored).
- There is no test runner, ESLint, or Prettier config. Typecheck is the automated gate.
- Dev services and downloads live under `./service/` (gitignored). Packaged installs use a sibling `service/` next to the app directory.

## Architecture

| Layer | Path | Role |
| --- | --- | --- |
| Main | `electron/main.ts` | Window, tray, single-instance lock, `ipcMain.handle` |
| Managers | `electron/services/*Manager.ts` | Downloads, install/uninstall, PATH, process control |
| Config | `electron/services/ConfigStore.ts` | `electron-store` schema + `basePath` helpers |
| Preload | `electron/preload.ts` | `contextBridge` API **and** `Window['electronAPI']` types |
| Renderer | `src/` | Vue 3 + Pinia + Element Plus, hash router |

Security model: `contextIsolation: true`, `nodeIntegration: false`. The renderer talks **only** through `window.electronAPI`.

Packaged app uses `file://`, so routing **must** stay `createWebHashHistory()`. Window is frameless; close hides to tray (`isQuitting` is the only real exit). NSIS requests administrator (`requestedExecutionLevel: requireAdministrator`).

`src/vite-env.d.ts` must **not** redeclare `Window.electronAPI`. Types come from `export type ElectronAPI = typeof api` in `electron/preload.ts`.

## IPC contract (required for every new API)

Keep these three files in lockstep. A method that exists in only one layer is a bug.

1. Implement on the manager in `electron/services/`.
2. Register `ipcMain.handle("ns:method", ...)` in `electron/main.ts`. Channel names are `php:…`, `mysql:…`, `nginx:…`, `redis:…`, `node:…`, `go:…`, `rust:…`, `python:…`, `git:…`, `java:…`, `dotnet:…`, `postgres:…`, `mongodb:…`, `service:…`, `hosts:…`, `config:…`, `log:…`, `app:…`, `window:…`, `shell:…`, `dialog:…`, `composer:…`.
3. Add the same method on the `api` object in `electron/preload.ts` via `ipcRenderer.invoke`.

Renderer calls `window.electronAPI?.php.install(version)` (optional-chain: preload may be missing in a plain Vite tab).

Mutating operations return `{ success: boolean; message: string }` (sometimes extra `details`). Views show `ElMessage.success/error` from `message`.

Push-only events:

- `sendDownloadProgress(type, progress, downloaded, total)` from `electron/main.ts` → `download-progress`
- `service-status-changed` after tray start/stop all

Known drift (do not copy): preload exposes `app.setAutoStartServices` / `getAutoStartServices` with **no** main handlers; main exposes `mysql:reinitialize` with **no** preload method.

## Adding a runtime / page

1. `electron/services/XxxManager.ts` — same class shape as `NodeManager` / `GoManager`.
2. Wire constructor + IPC in `electron/main.ts`.
3. Expose under `api` in `electron/preload.ts`.
4. `src/views/XxxManager.vue` with `defineOptions({ name: 'XxxManager' })` (KeepAlive in `App.vue` matches this name).
5. Hash route in `src/router/index.ts`, nav item + `cachedViews` in `src/App.vue`.
6. Extend `ConfigStore` schema if you persist versions / active path.

## Data layout (`ConfigStore.getBasePath()`)

```
{basePath}/
  php/php-{version}/
  mysql/mysql-{version}/
  nginx/          sites-available/  sites-enabled/  ssl/
  redis/
  nodejs/
  go/
  rust/
  python/python-{version}/
  java/jdk-{version}/
  dotnet/dotnet-{version}/
  postgres/
  mongodb/
  logs/  temp/  www/
  path_backup.txt
```

- Dev: `<repo>/service`
- Packaged: sibling of the install dir (e.g. `C:\DevTools\service` if the app is `C:\DevTools\DevStack\`)

`active*Version` is the display / isActive key. `active*Path` is set when the active tool is a **system** install so helpers do not resolve into the managed tree.

Installed tool entries use `source`: `managed` | `system` (Python also `mise`; Rust `rustup`). System installs only support set-as-default + uninstall. CGI / extensions / ini / logs are managed-only.

## PATH — do not regress this

All user-PATH writes go through `PathManager` (`electron/services/PathManager.ts`) and `withPathLock` (`pathLock.ts`).

- Remove **only** entries under an explicit managed prefix (e.g. `{basePath}\php`). Never glob `*\php-*` or similar.
- Writes backup PATH to `{basePath}/path_backup.txt`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ethanfly/dev-stack](https://github.com/ethanfly/dev-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
