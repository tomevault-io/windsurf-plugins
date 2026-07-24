---
trigger: always_on
description: **To build the project, use the `/build` skill.** It handles dependency checks (`npm install`) and all build variants. See `.github/skills/build/SKILL.md` for full details.
---

# Copilot Instructions for Cosmos Explorer

## Build, Test, and Lint

**To build the project, use the `/build` skill.** It handles dependency checks (`npm install`) and all build variants. See `.github/skills/build/SKILL.md` for full details.

Quick reference:

```bash
npm install              # Install dependencies (runs patch-package and i18n key generation automatically)
npm run build            # Full build: format check → lint → compile → strict compile → webpack prod → copy
npm run build:ci         # CI build: same as above but uses webpack dev mode (faster)
npm start                # Dev server with hot reload at https://localhost:1234 (see /dev-server skill)

npm run compile          # TypeScript check only (no emit)
npm run compile:strict   # TypeScript strict mode check (subset of files in tsconfig.strict.json)
npm run lint             # ESLint across all .ts/.tsx files
npm run format           # Prettier format (write)
npm run format:check     # Prettier format (check only)
```

### Testing

**To run unit tests, use the `/run-unit-tests` skill.** It handles dependency checks and all test variants. See `.github/skills/run-unit-tests/SKILL.md` for full details.

Quick reference:

```bash
npm test                 # Run all unit tests with Jest (includes coverage)
npm run test:file -- path/to/file.test.ts   # Run a single test file (no coverage)
npm run test:debug       # Run tests serially for debugging (--runInBand)

# E2E tests (requires .env config and running dev server)
npm run test:e2e         # Playwright E2E tests
npx playwright test test/sql/document.spec.ts  # Run a single E2E spec
```

Unit tests live adjacent to source files (`*.test.ts` / `*.test.tsx` in `src/`). E2E tests are in `test/` organized by API type (sql, mongo, cassandra, gremlin, tables).

### Dev Server

**To start the local dev server and connect a browser, use the `/dev-server` skill.** It handles dependency checks, server startup, readiness polling, and Playwright browser navigation. See `.github/skills/dev-server/SKILL.md` for full details.

The dev server runs at `https://localhost:1234` with a `/_ready` health-check endpoint. Entry points include `hostedExplorer.html` (standalone), `explorer.html` (portal iframe), and `index.html` (emulator).

## Architecture

### Platform Modes

The app runs in four hosting contexts, determined by `ConfigContext.platform`:
- **Portal** – Embedded as an iframe inside Azure Portal
- **Hosted** – Standalone at cosmos.azure.com, supports AAD, connection string, and resource token auth
- **Emulator** – Connects to local Cosmos DB Emulator via master key
- **Fabric** – Embedded in Microsoft Fabric, communicates via postMessage RPC

Platform-specific code lives in `src/Platform/{Emulator,Hosted,Fabric}/`. The active platform is set during initialization in `src/hooks/useKnockoutExplorer.ts`, which orchestrates bootstrapping for all modes.

### Global State (Module Singletons + Zustand)

State is **not** managed by React Context or Redux for most application concerns. Instead:

- **`userContext`** (`src/UserContext.ts`) – Module-level singleton holding current account info, auth tokens, API type, and feature flags. Updated via `updateUserContext()`. Not a React store; components read it directly.
- **`configContext`** (`src/ConfigContext.ts`) – Module-level singleton for environment/endpoint configuration. Updated via `updateConfigContext()`.
- **Zustand stores** (`src/hooks/use*.ts`) – Used for UI state that React components need to subscribe to reactively. Key stores:
  - `useDatabases` – Database/collection tree state
  - `useTabs` – Open tab management
  - `useQueryCopilot` – Copilot query assistant state
  - `useNotificationConsole` – Console notifications
  - `useSidePanel` – Side panel visibility
  - `useSelectedNode` – Currently selected tree node

### Data Access Layer

`src/Common/dataAccess/` contains all Cosmos DB CRUD operations (createCollection, readDatabases, queryDocuments, etc.). These functions call either the Cosmos SDK (`@azure/cosmos`) directly or go through proxy endpoints depending on the API type and auth method.

ARM (Azure Resource Manager) clients are auto-generated in `src/Utils/arm/generatedClients/` — regenerate with `npm run generateARMClients`.

### Multi-API Support

Cosmos DB supports multiple APIs: SQL, Mongo, Gremlin, Tables, Cassandra, Postgres, and VCoreMongo. The current API type is determined from `userContext.apiType` (derived from the database account's capabilities). API-specific UI components branch on this value.

### Entry Points

Webpack builds multiple independent entry points (see `webpack.config.js`):
- `src/Main.tsx` → `explorer.html` (Portal iframe)
- `src/Index.tsx` → `index.html` (Emulator)
- `src/HostedExplorer.tsx` → `hostedExplorer.html` (cosmos.azure.com)
- Plus: terminal, cellOutputViewer, galleryViewer, selfServe, connectToGitHub, quickstart

### Knockout → React Migration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/cosmos-explorer](https://github.com/Azure/cosmos-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
