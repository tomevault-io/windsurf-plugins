---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Dugong Gas Planner — a client-side web app for recreational scuba divers to plan dive gas consumption,
decompression, and related calculations (Bühlmann ZHL-16C with gradient factors, RMV/SAC, Nitrox/Trimix,
NDL tables, etc). Runs fully offline as a PWA; no backend. Live instance: https://dugong.online/.

## Commands

Build order matters: the `scuba-physics` library must be built before the `planner` app consumes it.

- `npm run build-lib` — production build of the `scuba-physics` library (run this first after any lib change).
- `npm start` — builds `scuba-physics` then serves `planner` via `ng serve` (dev mode, http://localhost:4200).
- `npm run build` — production build of `planner` into `dist/planner`.
- `npm run start-pwa` — production build of `planner`, then serves `dist/planner` via `http-server` on port `9090` (use this to test service-worker/offline/PWA behavior, which does not work under `ng serve`).
- `npm test` — Karma unit tests for `planner` (interactive/watch, Chrome).
- `npm run test-lib` — Karma unit tests for `scuba-physics` (interactive/watch, Chrome).
- `npm run test-ci` / `npm run test-lib-ci` — headless, single-run variants used in CI.
- `npm run lint` — `ng lint` (Angular ESLint) across both projects.
- `npm run e2e` — Playwright E2E (`playwright-ng-schematics` builder), auto-starts the dev server; single spec currently in `e2e/startupSmoke.spec.ts`.
- Run a single spec file: `ng test --project planner --include='**/foo.spec.ts'` (swap `--project scuba-physics` for library specs).
- Windows PowerShell equivalents of the main build/test flow are in `build/*.ps1`, but the npm scripts above are the primary workflow.

## Architecture

### Workspace layout

Two Angular projects under `projects/`:

- **`scuba-physics`** — framework-agnostic dive-physics/algorithm library, built with `ng-packagr`. Public surface is explicitly re-exported from `projects/scuba-physics/src/public-api.ts`; anything not exported there is internal. Organized by domain, not by layer:
  - `lib/algorithm` — Bühlmann ZHL-16C implementation, compartments, tissues, profile/event calculation.
  - `lib/consumption` — gas consumption, tanks, divers, standard tank definitions.
  - `lib/depths` — dive profile/segment/level modeling.
  - `lib/gases` — gas mixtures, gas properties, gas toxicity, standard gas names.
  - `lib/physics` — unit conversion, pressure/depth conversion, time.
  - `lib/calculators` — standalone calculators (SAC, NDL, CNS/OTU, Nitrox, gas blending, weight, altitude, gas pricing).
  - `lib/common` — shared primitives (precision, feature flags).
- **`planner`** — the standalone-component Angular app (no NgModules). Bootstrapped from `src/main.ts` via `bootstrapApplication(AppComponent, CONFIG)`, where `CONFIG` is assembled in `src/app/app.config.ts`.

### App composition (`app.config.ts` / `app.routes.ts`)

- `app.config.ts` is the single composition root: standalone components are listed in the `STANDALONE` array, injectable services in `SERVICES`, third-party Angular modules (MDB UI Kit, `NgxMdModule`, `ServiceWorkerModule`, etc.) alongside them. When adding a new standalone component or service, register it in the matching array here rather than importing it ad hoc elsewhere.
- `app.routes.ts` defines flat routes per feature/calculator (`KnownViews` enum) plus a catch-all `DashboardComponent` route guarded by `canActivateDashboard`, which redirects an empty-query root navigation back to the last-viewed dashboard view (state tracked in `ViewStates`).

### State & services (`projects/planner/src/app/shared/*`)

Business/state logic is kept out of components and lives in injectable services under `shared/`, e.g. `PlannerService`, `TanksService`, `DepthsService`, `OptionsService`, `DiveSchedules` / `ManagedDiveSchedules`, `PreferencesStore`. The app supports multiple concurrent dive schedules, but only one is displayed at a time; **`ReloadDispatcher`** is the central RxJS event bus that notifies the UI when the *selected* schedule's tanks/depths/options/results change, decoupling multi-schedule state from re-render triggers. Dive plans are shareable via URL — see `PlanUrlSerialization` and `serialization.model.ts` (DTOs) for the (de)serialization format used both for URL sharing and for `PreferencesStore`/local persistence.

### Background computation (Web Workers)

Heavy dive-profile/consumption/decompression math runs off the main thread. `projects/planner/src/app/workers/*.worker.ts` (`profile.worker.ts`, `diveInfo.worker.ts`, `consumption.worker.ts`) wrap `scuba-physics` algorithm calls; `shared/workers.factory.ts` (browser `Worker` instances) implements the `WorkersFactoryCommon` abstraction from `shared/serial.workers.factory.ts` (a synchronous fallback used e.g. in tests/non-worker contexts). Request/response payloads are typed DTOs in `shared/serialization.model.ts`. When changing algorithm inputs/outputs in `scuba-physics`, keep these worker DTOs and both factory implementations in sync.

### Docs / Help


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jirkapok/GasPlanner](https://github.com/jirkapok/GasPlanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
