---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

AEGIS (Artemis EVA GIS) is a full-stack web application for planning, training, and executing lunar surface EVA (Extra-Vehicular Activity) missions. It provides a collaborative GIS interface with real-time multi-user editing.

## Commands

```bash
# Local development (starts Vite + API concurrently)
npm run dev

# Frontend only
npm run vite:dev

# Backend only (with hot reload)
npm run api:dev

# Start required Docker services (PostgreSQL)
npm run docker:services

# Build everything
npm run build

# Lint (ESLint + StyleLint)
npm run lint
npm run lint:fix

# Unit tests (Vitest)
npm run test:vitest

# Component/DOM tests (Vitest browser mode)
npm run test:vitest:browser

# Full CI check (lint + tsc + build + unit tests)
npm run test:all

# E2E tests (Playwright)
npm run test:playwright

# Database migrations
npm run migration:up
npm run migration:down
npm run migration:fresh   # drop + recreate + seed

# Seed existing DB
npm run seed
```

## Comments

Keep comments short. Write comments that explain what the code does or why — never comments that narrate what wasn't done, what a previous approach was, or what you chose not to do. Delete such notes rather than adding them.

## After Code Changes

After every batch of code changes, run:

```bash
./node_modules/.bin/prettier --config .prettierrc.json --write <changed-files>
npm run test:all
```

Format every changed file with the root `.prettierrc.json` before the full check; use the direct Prettier command above and replace `<changed-files>` with the files changed in the batch. This runs lint (JS/TS + CSS) → tsc → build → vitest → vitest:browser in sequence. Fix any failures before reporting the task complete. Do not skip lint — this project has non-standard CSS and JS/TS lint rules that will fail on patterns that look valid (e.g. specific import ordering, CSS property conventions). If lint fails, read the error output carefully and fix exactly what it reports rather than guessing at the rule.

## Map / OpenLayers

Any prompt that mentions "map", "OpenLayers", "OL", "ol", tiles, markers, layers, or the map implementation must first read `src/components/interface/map/CLAUDE.md` for full architecture context before doing any work.

## GIS Data Conversion Pipeline

Any prompt about the GIS data processing pipeline (converting GIS drops into AEGIS map products — tiles, COG, PMTiles, GeoJSON, mission grids, `properties.json`/`manifest.json`, or the `register`/Box publish flow) must first read `GIS_data_conversion_pipeline/esri-to-aegis-lunar-southpole/CLAUDE.md` before doing any work.

## Architecture Overview

The app is a monolithic full-stack TypeScript project with a React SPA frontend and an Express API backend, both in the same `src/` tree.

### Key Directories

| Path                            | Role                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| `src/components/`               | React UI components (panes, pages, dashboard, interface)     |
| `src/components/interface/map/` | OpenLayers map implementation (active map layer)             |
| `src/pages/`                    | Top-level page components routed by React Router             |
| `src/store/`                    | Redux Toolkit slices, thunks, selectors, and store utilities |
| `src/client/`                   | Automerge mutation helpers (client-side doc operations)      |
| `src/http-client/`              | Typed `fetch` wrappers for every REST endpoint               |
| `src/utils/`                    | Shared helpers: logging, formatting, permissions, socket ops |
| `src/packages/`                 | Lightweight shared utilities (fetchFns, user helpers)        |
| `src/server/express/`           | Express app, REST routes, Socket.io setup                    |
| `src/server/database/`          | MikroORM config, entity models, migrations, seeds            |
| `src/server/automerge/`         | PostgreSQL storage adapter for Automerge documents           |

### Frontend

- **React 18** SPA bootstrapped by Vite.
- **Redux Toolkit** manages UI state only. Slices live in `src/store/` (no `slices/` subdirectory), async operations in `src/store/thunk/`, memoized selectors in `src/store/selectors.ts`. Entity data (missions, EVAs, stations, POIs, etc.) is **not** stored in Redux — it lives exclusively in Automerge documents. Redux slices track only UI state: selected items, expanded panels, navigation state, etc.
- **OpenLayers** drives the map canvas. Map-related components live under `src/components/interface/map/` (the three entry points are `AegisMapEditor.tsx`, `AegisMapDashboard.tsx`, and `AegisMapMinimap.tsx`). See `src/components/interface/map/CLAUDE.md` for full architecture details.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nasa/aegis](https://github.com/nasa/aegis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
