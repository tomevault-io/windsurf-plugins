---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

This is a monorepo with two npm packages that ship together as the `homebridge-config-ui-x` plugin:

- **`/` (root)** — Nest.js backend (TypeScript, ESM, Fastify adapter). Compiles to `dist/`. Requires Node `^22.12.0 || ^24.0.0`.
- **`/ui`** — Angular 22 frontend (private package). Compiles to `public/` (served as static assets by the backend — `outputPath` in `ui/angular.json` is `../public`).

The UI package has its own `package.json`, `node_modules`, and tsconfig. **You must `npm install` in both root and `ui/` separately.**

## Common commands

Run from the repo root unless noted.

```sh
# First-time setup
npm install && npm install --prefix ui

# Full build (server + ui)
npm run build               # ~30s; runs build:server then build:ui
npm run build:server        # tsc -p tsconfig.build.json → dist/
npm run build:ui            # ng build production → public/ (prebuild regenerates the Font Awesome subset)

# Dev (live reload, two processes via concurrently)
npm run watch               # UI dev server on :4200, backend on :8581

# Lint (eslint flat config, antfu base, max-warnings=0; covers both packages)
npm run lint
npm run lint:fix

# Tests — Vitest e2e, ~45s full suite, runs against real Nest module instances
npm run test
npm run test -- test/e2e/auth.e2e-spec.ts          # single file
npm run test -- -t "should reject invalid login"   # single test by name
npm run test-coverage

# Translation key sync (en.json is the master)
npm run lang-sync
```

## Three entry points to understand

The plugin can be loaded three ways, and each goes through a different bootstrap path. This is unusual and worth knowing before navigating `src/bin/`:

1. **As a Homebridge plugin** (`src/index.ts`) — Homebridge calls `registerPlatform`. The plugin class does almost nothing; it just sets `UIX_CONFIG_PATH`/`UIX_STORAGE_PATH` env vars from the Homebridge API. The actual UI server is launched by Homebridge as a separate child process via `src/bin/fork.ts`, which then loads `main.ts`.
2. **Via `hb-service`** (`src/bin/hb-service.ts`, exposed as the `hb-service` bin) — the supported way to run Homebridge as an OS service on Linux/macOS/Windows/FreeBSD. `hb-service run` forks both Homebridge itself and the UI, manages restarts, and pipes logs. Platform-specific installers live in `src/bin/platforms/{darwin,linux,win32,freebsd}.ts`.
3. **Standalone** (`src/bin/standalone.ts`) — for development or `npm run start`. Just sets `UIX_STORAGE_PATH` then imports `main.js`.

All three eventually call `bootstrap()` in `src/main.ts`, which builds the Nest app on Fastify, registers helmet/multipart/CSP, mounts the SPA at `/`, the API at `/api`, Swagger at `/swagger`, a Socket.io gateway under namespace `app`, and (optionally) advertises the UI over mDNS/Bonjour.

## Backend ↔ Homebridge IPC

The UI doesn't import Homebridge as a library — it talks to the running Homebridge process over Node IPC. The bridge is `src/core/homebridge-ipc/homebridge-ipc.service.ts`, which extends `EventEmitter` and is wired up by `hb-service` after it forks Homebridge (it calls `setHomebridgeProcess()` on the exported `HomebridgeIpcService` from `main.ts`). Events like `childBridgeStatusUpdate` and `serverStatusUpdate` flow through this service to the rest of the app and out via WebSocket gateways.

When standalone or in dev watch mode (`npm run watch`), there's no Homebridge process attached, so IPC-dependent features (child bridge controls, restart, log tail) won't work end-to-end — that's expected.

## Backend module layout

`src/app.module.ts` imports feature modules from `src/modules/` and infrastructure from `src/core/`:

- **`core/`** — cross-cutting: `auth` (JWT + passport, HTTP guards plus WS guards in `guards/`), `config` (loads/parses `config.json`, holds runtime env detection — Docker/Synology/RPi/etc.), `feature-flags`, `fs`, `homebridge-ipc`, `logger`, `matter` (interfaces), `node-pty` (terminal), `scheduler`, `spa` (catch-all filter so non-`/api` routes serve `index.html`), `ssl`.
- **`modules/`** — one folder per feature surface, each with a `*.module.ts`, controller, service, gateway (when WS-enabled), and DTOs. The set is: `accessories`, `backup`, `child-bridges`, `config-editor`, `custom-plugins`, `log`, `platform-tools`, `plugins`, `server`, `setup-wizard`, `status`, `users`.

## Frontend layout

`ui/src/app/` splits into `core/` (singletons, guards, interceptors), `modules/` (routed feature areas), and `shared/`:

- **`core/communication/`** — `api.service.ts` (HTTP wrapper for `/api`), `ws.service.ts` (socket.io-client), `notification.service.ts`. JWT handling lives in `core/auth/` (via `@auth0/angular-jwt`).
- **`modules/`** — mirror the backend feature modules (`config-editor`, `plugins`, `status`, `users`, `platform-tools`, …). When adding a feature, expect to touch a backend module + its UI counterpart.
- In dev mode, `ui/src/environments/environment.ts` hard-codes the backend at port `8581` on the current hostname — that's why `npm run watch` runs the backend on 8581.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [homebridge/homebridge-config-ui-x](https://github.com/homebridge/homebridge-config-ui-x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
