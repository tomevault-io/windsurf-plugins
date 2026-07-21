---
trigger: always_on
description: AnGe-Panel is a single-repository Go + Vue application.
---

# AGENTS.md

## Project Shape

AnGe-Panel is a single-repository Go + Vue application.

- Backend: Go module `sun-panel`, Gin, Gorm, SQLite/MySQL, sources at the repository root (`main.go`, `api/`, `router/`, `initialize/`, `models/`, `lib/`, `global/`, `structs/`).
- Frontend: Vue 3 + Vite + TypeScript + Pinia + Naive UI under `src/`.
- Built frontend output is served by the backend from `./web` at runtime. The committed `dist/` bundle is used by Docker and release packaging.
- Runtime mutable data lives in `conf/`, `database/`, `uploads/`, and `runtime/`; Docker maps these to `/data`.

## Important Compatibility Constraints

- Keep the Go module path as `sun-panel` unless doing a full repository-wide migration. Many imports and linker flags depend on it.
- Expect historical `Sun-Panel` naming in code, scripts, and UI metadata. Do not rename broadly unless the task explicitly calls for a branding migration.
- The current active project layout has the Go backend at the repository root. Some legacy scripts/workflows still reference `service/`; treat those references as stale unless you are intentionally fixing release automation.
- Do not remove or casually regenerate committed seed/runtime assets:
  - `seed/database/database.db`
  - `seed/uploads/**`
  - `dist/**`
  - executable `ange-panel`
- Do not overwrite user data paths (`database/`, `uploads/`, `conf/conf.ini`, `runtime/`) during development or migration work.
- `conf/conf.example.ini` is the source template for generated config; keep it in sync when adding config keys.

## Build And Verification Commands

Use the narrowest command that proves the change.

- Frontend install/type checks may be run with `pnpm install` and `pnpm run type-check`; frontend dev/build commands are intentionally blocked because the legacy `src/` styles do not match the current 3005 release UI.
- Frontend type check only: `pnpm run type-check`.
- Frontend lint: `pnpm run lint`.
- Backend compile: `go build ./main.go`.
- Backend package checks: `go test ./...` if tests exist or backend behavior changed.
- Docker image build expects `dist/` to already contain the frontend bundle: `docker build .`.

Notes:

- The frontend package manager is pnpm; avoid introducing new npm/yarn lockfile churn.
- `pnpm run build`, `pnpm run build:frontend`, `pnpm run build-only`, and `pnpm run dev` are blocked on purpose so the old Vite frontend cannot overwrite the current 3005 bundle.
- `add-frontend-version.js` mutates `.env` by updating `VITE_APP_VERSION`; be aware of that expected side effect.

## Runtime Behavior

- Backend startup is `initialize.InitApp()` then `router.InitRouters(":" + http_port)`.
- Default backend port comes from `[base] http_port` in `conf/conf.ini`/`conf.example.ini`, currently `3005`.
- Local development should use one visible port by default: the Go backend port from `conf/conf.ini` (currently `3005` in this workspace).
- Do not start the Vite dev server unless the user explicitly asks for frontend HMR.
- In this workspace, the current correct 3005 frontend styling is the backend-served `dist/` bundle. The checked-in `src/` frontend can produce an older-looking UI if rebuilt directly.
- Do not overwrite the current `dist/` bundle with `vite build`/`pnpm run build`. The package scripts intentionally block Vite dev/build; do not re-enable them unless the legacy frontend source/style mismatch has been resolved and the user explicitly asks for a source rebuild workflow.
- For urgent frontend fixes that must appear on 3005 while preserving the current visual style, patch the backend-served `dist/assets/**` file that `dist/index.html` actually references, then verify through port `3005`.
- After code changes, the working result must be visible through the backend-served app on port `3005`. Do not consider frontend work complete if it only works on a Vite development port.
- Keep backend-served frontend files cache-safe: `index.html`, `/assets/**`, `/custom/**`, and service-worker kill-switch files must not allow stale browser/PWA cache to keep an old 3005 UI alive after deployment.
- Vite dev server config exists only for deliberate frontend rebuild work. Because it creates a second origin with separate browser storage, it can look different from the backend-served app.
- Backend serves static frontend files from `./web` and uploaded files from `[base] source_path`, currently `./uploads`.
- SQLite is the default database. If the database file is absent, startup may seed from `seed/database/database.db`.
- Startup also seeds uploads from `seed/uploads` when `uploads/` is missing or effectively empty.

## API And Backend Conventions

- API routes are mounted under `/api`.
- Add backend endpoints by updating both the router package (`router/system`, `router/panel`, or `router/openness`) and the handler package (`api/api_v1/...`).
- Use the shared response helpers in `api/api_v1/common/apiReturn`; frontend expects JSON shaped like `{ code, msg, data? }`.
- Success code is `0`. Existing frontend auth/error behavior depends on codes such as `1000`, `1001`, and `1005`.
- Reuse middleware in `api/api_v1/middleware` for login/admin/public-mode access checks.
- Gorm uses singular table names. New persistent models must be added to `initialize/database/connect.go` `AutoMigrate`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liandu2024/AnGe-Panel](https://github.com/liandu2024/AnGe-Panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
