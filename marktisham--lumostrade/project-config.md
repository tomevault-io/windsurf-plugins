---
trigger: always_on
description: <!-- Copilot / AI assistant instructions for the Lumos monorepo -->
---

<!-- Copilot / AI assistant instructions for the Lumos monorepo -->
# Lumos — Copilot Instructions

This repo is a small TypeScript monorepo plus a Python-based AI workspace. The core packages are `LumosApp` (Express web UI), `LumosTrade` (core trade/order processing library), and `LumosCLI` (CLI wrappers). `LumosAI` hosts agents (like `LumosAgents`) and tools (like `LumosDB`) used by the web app. The guidance below is intentionally concise and actionable for code changes, tests, and running/debugging locally.

- **Architecture & dependency boundaries** (CRITICAL — respect these):
  - `LumosTrade` is the **portable core library** for broker trading and data collection. It is NOT a UI layer — all display logic belongs in consumers.
  - `LumosApp` is the **web frontend** to display trade data. It depends on `LumosTrade` via `file:../LumosTrade` in package.json.
  - `LumosCLI` is a **thin CLI** to test `LumosTrade`. Same dependency relationship.
  - **One-way dependencies only**: `LumosApp` → `LumosTrade` ← `LumosCLI`. NO circular dependencies. `LumosTrade` must remain UI-agnostic.
  - Inside `LumosTrade/src/interfaces/*`: general-purpose types (not broker-specific). Broker-specific logic extends `BrokerClient` (e.g., `ETClient` in `ETrade/`) and maps to these shared interfaces.
  - `LumosAI` is an AI workspace (Python) with **Agents** and **Tools**. `LumosApp` calls the `LumosChat` agent (hosted on `LumosAgents`) for the chat experience. Agents call tools (for example, `LumosDB`) to access the trading database.
  - The end-to-end flow is: `LumosApp` → `LumosAgents` → tool services (e.g., `LumosDB`) → Cloud SQL.

- **Root environment launcher** (CRITICAL — use this):
  - `./lumos` is the single entrypoint for environment-aware actions. It loads environment variables from `config/<environment>.expanded.env` and delegates to `shell/_invoker.sh`.
  - **Environment persistence**: Each terminal window automatically remembers its environment selection (stored in `/tmp/lumos-env-<shell-pid>`). Different terminal windows can use different environments simultaneously.
  - **Environment management**: 
    - `./lumos env set <environment>` — Set/create environment (e.g., `./lumos env set development`). Selection is remembered for the current terminal.
    - `./lumos env list` — See all available environments
    - `./lumos env validate` — Check current environment configuration
    - `./lumos env show` — Display source, expanded, and current environment variables
    - `./lumos env update` — Regenerate expanded environment file
  - **Environment variables**: Edit `config/<environment>.env` directly (e.g., `config/development.env`, `config/production.env`). Format is simple `KEY=VALUE` (no export statements).
  - Environment configs use `LUMOS_ENVIRONMENT` variable (replaces old `ENVIRONMENT` variable)
  - Template available at `config/template.env` for creating new environments
  - Available actions (see `shell/_invoker.sh`): `auth`, `build clean|watch|lumostradetool`, `deploy all|lumosapp|lumosagents|lumosdb|lumostradetool`, `run lumosdb|lumosdb-agent|lumosagents|lumostradetool|lumostradetool-agent`, `service update|test`, `sql`, `env set|list|validate|show|update`.
  - `deploy all` runs in order: `LumosDB` → `LumosTradeTool` → `LumosAgents` → `LumosApp`.

- **Build & watch**: Use the root scripts or environment launchers.
  - `npm run build:all` — compile all packages (`tsc -p ...`) and copy required config/templates/public files to `dist`.
  - `npm run watch:all` — starts `tsc --watch` in all packages (uses `npm-run-all`).
  - `./lumos build clean` — wipes node_modules/dist and rebuilds via root scripts.
  - `./lumos build watch` — runs `npm run watch:all` via `shell/_watch_all.sh`.
  - Per-package builds: `npm run build:lumosapp`, `npm run build:lumostrade`, `npm run build:lumoscli`.

- **Run / debug LumosApp**:
  - After `npm run build:lumosapp`, run `cd LumosApp && NODE_ENV=development PORT=8080 node dist/index.js`.
  - The app reads config from `LumosApp/config/*` (copied to `dist/config` during build). Use `process.env.PORT` and `process.env.BUILD_NUMBER` to influence runtime.
  - When using Cloud Run or the ./lumos launcher, environment variables are sourced from `config/<environment>.env`.
  - For VS Code debugging, use the "Dev Web App" or "Prod Web App" launch configurations which automatically load the appropriate config/development.expanded.env or config/production.expanded.env file.

- **Tests**:
  - `LumosTrade` has comprehensive unit test coverage using Jest (18 test suites covering core domain logic, trade/order processing, broker response mapping, date utilities, rollup calculations, and expected moves). Run tests from that package: `cd LumosTrade && npm test`.
  - Test files are located in `LumosTrade/src/test/*.test.ts` and cover critical business logic including trade construction from orders, multi-leg options, account transfers, rollup calculations, and broker-specific response parsing.
  - The `./lumos service test` target is a placeholder for service-account verification; use `./lumos service update` to apply roles and bindings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marktisham/LumosTrade](https://github.com/marktisham/LumosTrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
