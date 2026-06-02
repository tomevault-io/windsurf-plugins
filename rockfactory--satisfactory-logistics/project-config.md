---
trigger: always_on
description: **Satisfactory Logistics** is a React/TypeScript single-page application (SPA) for planning factories in the game [Satisfactory](https://www.satisfactorygame.com/). Key features:
---

# Satisfactory Logistics — AGENTS.md

## Project Overview

**Satisfactory Logistics** is a React/TypeScript single-page application (SPA) for planning factories in the game [Satisfactory](https://www.satisfactorygame.com/). Key features:

- **Logistics tracking** — define factory inputs/outputs and visualize resource flows
- **Calculator** — an LP (linear programming) solver that computes optimal production chains given constraints
- **Charts** — Sankey diagrams and node graphs for resource flow visualization
- **Savegame import** — parse `.sav` game files to seed the planner with real in-game data
- **Cloud sync** — optional Supabase-backed authentication and remote save/load

Live site: https://satisfactory-logistics.xyz

---

## Requirements

- **Node.js v22+** — use [nvm](https://nvm.sh); `nvm use` picks the version from `.nvmrc`
- **npm 10.8.3** (declared in `packageManager` field)

---

## Quick Start

```bash
npm install
npm run dev        # starts Vite dev server at http://localhost:5173
```

---

## Available Scripts

| Script | Description |
|---|---|
| `npm run dev` | Start Vite development server |
| `npm run build` | TypeScript check + Vite production build |
| `npm run preview` | Serve the production build locally |
| `npm test` | Run Vitest unit tests |
| `npm run check-types` | TypeScript type check without emitting output |
| `npm run lint` | Biome check on `src/` (linting + formatting) |
| `npm run format` | Biome auto-format `src/` (write in place) |
| `npm run parse-docs` | Parse Satisfactory game data files into JSON (see below) |
| `npm run supabase:types` | Regenerate `src/core/database.types.ts` from Supabase schema |

---

## Project Structure

```
satisfactory-logistic/
├── src/
│   ├── auth/              # Supabase auth, session manager, remote sync
│   ├── core/              # Store setup, Zustand helpers, migrations, logger, Supabase client
│   ├── factories/         # Factory domain: list, detail, inputs/outputs, charts, store slices
│   ├── games/             # Game/save management: create, import, settings, store slices
│   ├── layout/            # App-level layout: Header, Footer, sticky elements
│   ├── recipes/           # Game data: items, recipes, buildings, schematics (JSON + types)
│   ├── routes/            # React Router route definitions
│   ├── solver/            # LP solver: algorithm, graph layout, share, store slices, tests
│   ├── third-party/       # External integrations (Ko-fi, feedback)
│   ├── utils/             # Shared utilities and components
│   ├── App.tsx            # Root component
│   ├── main.tsx           # Entry point
│   └── theme.ts           # Mantine dark theme configuration
├── data/
│   ├── docs-en.json       # Raw English Satisfactory game docs (source for parse-docs)
│   ├── docs-it.json       # Italian variant
│   └── assets/            # Exported game textures (used by parse-docs --with-images)
├── scripts/
│   └── parseDocs.ts       # parse-docs script: generates JSON data in src/recipes/
├── public/                # Static assets (favicon, logo)
└── dist/                  # Build output (git-ignored)
```

---

## Architecture

### State Management (Zustand + Immer)

State is organised into **slices** composed into a single Zustand store. The helper utilities live in [src/core/zustand-helpers/](src/core/zustand-helpers/).

```
authSlice         → Supabase session
gamesSlice        → game list, selected game, per-game factory IDs
gameSaveSlice     → local/remote persistence state
factoriesSlice    → factory definitions (inputs, outputs, progress)
factoryViewSlice  → UI state (grid / spreadsheet / kanban view mode)
solversSlice      → LP solver instances, request params, node layout
chartsSlice       → chart visualization preferences
```

- Mutations use **Immer** (mutation-style code, immutable result).
- State is persisted to **IndexedDB** via `idb-keyval`.
- Migrations exist for versions v2 → v4 in [src/core/migrations/](src/core/migrations/).
- `gameSave` slice is excluded from persistence.

### Routing

React Router v6, defined in [src/routes/FactoriesRoutes.tsx](src/routes/FactoriesRoutes.tsx):

| Path | View |
|---|---|
| `/login` | Login page |
| `/privacy-policy` | Privacy policy |
| `/factories` | Factories list (grid / spreadsheet / kanban) |
| `/factories/:id` | Factory detail + inline calculator |
| `/factories/:id/calculator` | Factory's solver view |
| `/factories/charts` | Sankey / graph charts |
| `/factories/calculator` | Standalone LP calculator |
| `/factories/calculator/shared/:sharedId` | Shared solver import |
| `/games/*` | Game management pages |
| `*` | Redirect → `/factories` |

### LP Solver (HIGHS)

The calculator uses **HIGHS** (linear programming library compiled to WebAssembly) to compute optimal production chains. Key files:

- [src/solver/algorithm/solveProduction.ts](src/solver/algorithm/solveProduction.ts) — core solve logic
- [src/solver/page/useSolverSolution.ts](src/solver/page/useSolverSolution.ts) — React hook integrating HIGHS with the store
- [src/solver/store/solverSlice.ts](src/solver/store/solverSlice.ts) — solver state

### Game Data


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rockfactory/satisfactory-logistics](https://github.com/rockfactory/satisfactory-logistics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
