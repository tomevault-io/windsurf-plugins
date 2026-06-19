---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository.

## Project Overview

DJ (Data JSON) Framework is a VS Code extension that revolutionizes dbt development through a structured, JSON-first approach. Users define dbt models and sources as validated `.model.json` and `.source.json` files that automatically generate corresponding SQL and YAML configurations.

The extension provides a rich visual UI built with React, including interactive model and column lineage graphs, a visual model creation wizard, query result previews, and a data modeling canvas -- all rendered as VS Code webviews.

**Key Technologies:**

- TypeScript extension backend running in Node.js/VS Code extension host
- React 18 + Vite frontend for webviews (interactive lineage graphs, model wizards, data explorer)
- dbt Core integration with manifest.json parsing
- Trino CLI integration for data catalog browsing and query execution
- Lightdash CLI integration for BI dashboards

## Common Commands

Run `npm install` first. The live source of truth for every script is `package.json`; the ones below cover day-to-day work.

| Command                                     | Purpose                                                                 |
| ------------------------------------------- | ----------------------------------------------------------------------- |
| `npm run dev`                               | Start all watchers (recommended for active development)                 |
| `npm test`                                  | Run the full Jest suite (`npm run fixtures:update` to refresh fixtures) |
| `npm run lint:all` / `npm run lint:fix:all` | Lint (or autofix) both the extension and web surfaces                   |
| `npm run compile` / `npm run compile:web`   | Build the extension / web TypeScript                                    |
| `npm run schema`                            | Regenerate TypeScript types after editing `schemas/`                    |
| `npm run format` / `npm run format:check`   | Apply / verify Prettier formatting                                      |
| `npm run package`                           | Produce the `.vsix` package                                             |

Local iteration is covered under [Development Tips](#development-tips).

## Architecture

### Dual-Architecture System

**Extension Backend (`src/`)** — TypeScript running in VS Code extension host:

- Entry point: `src/extension.ts` activates the `Coder` service
- Core orchestrator: `src/services/coder/index.ts` wires up all services
- Uses **ServiceLocator pattern** for lazy dependency injection to avoid circular dependencies

**Web Frontend (`web/`)** — React app rendered in VS Code webviews:

- Built with Vite, outputs to `dist/web`
- Pages: ModelCreate, SourceCreate, Home, QueryView, ModelRun, ModelTest, LightdashPreviewManager
- Lineage views: DataExplorer (model lineage), ColumnLineage, ModelLineage
- Message-based RPC communication with extension host
- State management with Zustand stores

**Shared Code (`src/shared/`)** — Cross-environment utilities used by both backend and frontend

### Service Architecture

Services follow a **layered architecture**:

```text
VS Code Commands & Views (UI Layer)
         ↓
Api Router (Message Bus) - src/services/api.ts
         ↓
Domain Services:
  • Framework (src/services/framework/index.ts) - JSON↔SQL/YAML sync
  • Dbt (src/services/dbt.ts) - manifest parsing, tree views
  • Trino (src/services/trino.ts) - query execution, catalog browsing
  • DataExplorer (src/services/dataExplorer.ts) - model lineage graph
  • ColumnLineage (src/services/columnLineage.ts) - column-level lineage
  • ModelLineage (src/services/modelLineage.ts) - model-level lineage
  • Lightdash (src/services/lightdash/index.ts) - BI integration
         ↓
Shared Utilities & Types (src/shared/)
```

**ServiceLocator Pattern** (`src/services/ServiceLocator.ts`):

- Breaks circular dependencies between services
- Lazy instantiation on first access via factory registration
- Type-safe access via `SERVICE_NAMES` constants
- Example:

```typescript
locator.register('dbt', () => new Dbt(locator.get('logger')));
const dbt = locator.get<Dbt>(SERVICE_NAMES.Dbt);
```

**Handler Pattern**:

- Large services split into specialized handlers
- Framework service uses handlers in `src/services/framework/handlers/`: UIHandlers, ModelDataHandlers, ModelCrudHandlers, ColumnLineageHandlers, SourceHandlers, PreferencesHandlers

### The Core Framework: JSON to SQL/YAML Sync

**Flow:**

```text
.model.json (user-editable, JSON Schema validated)
    ↓
Framework.handleJsonSync() triggers after debounce (configurable via dj.syncDebounceMs)
    ↓
SyncEngine orchestration (src/services/sync/SyncEngine.ts):
  1. Discovery - scan all .model.json files
  2. Dependency Resolution - build dependency graph (dependencyGraph.ts)
  3. Validation - validate against schemas (ValidationService.ts)
  4. Processing - ModelProcessor & SourceProcessor in dependency order
  5. Execution - generate and write SQL/YAML files (fileOperations.ts)
    ↓
.sql + .yml files (auto-generated, read-only)
```

**Key Points:**

- **JSON Schema Validation** using Ajv with 93 schemas in `schemas/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Workday/dj](https://github.com/Workday/dj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
