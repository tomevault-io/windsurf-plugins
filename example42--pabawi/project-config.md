---
trigger: always_on
description: **Last Updated:** January 2026
---

# Pabawi AI Coding Guidelines

**Version:** 0.5.0
**Last Updated:** January 2026

Pabawi is a unified web interface for infrastructure management, executing commands and tasks via Bolt, PuppetDB, Puppetserver, and Hiera. This document guides AI agents through the architecture, conventions, and workflows essential for productive contributions.

## Architecture Overview

### Monorepo Structure

```
pabawi/
├── frontend/          # Svelte 5 + Vite SPA (port 5173 dev, served by backend in prod)
│   └── src/           # Components, pages, stores (router.svelte.ts, api.ts)
├── backend/           # Node.js + Express + TypeScript API (port 3000)
│   ├── integrations/  # Plugin architecture: Bolt, PuppetDB, Puppetserver, Hiera
│   ├── routes/        # REST API endpoints (inventory, commands, tasks, facts, etc.)
│   ├── bolt/          # Bolt CLI execution service with caching and streaming
│   ├── database/      # SQLite execution history repository
│   └── services/      # ExecutionQueue, StreamingExecutionManager, etc.
└── docs/              # Configuration, API reference, architecture details
```

### Plugin Architecture (Critical)

**All backend integrations follow a consistent plugin pattern** (`backend/src/integrations/`):

1. **Plugin Types:** All plugins implement either `ExecutionToolPlugin`, `InformationSourcePlugin`, or both
2. **Base Class:** Extend `BasePlugin` for lifecycle management (initialize, healthCheck, isEnabled)
3. **Registration:** Plugins register with `IntegrationManager` at startup (see [backend/src/server.ts](backend/src/server.ts#L34-L120))
4. **Priority-Based Data Aggregation:** When multiple sources provide the same data, higher priority wins
   - Puppetserver: 20 (highest)
   - Bolt/PuppetDB: 10
   - Hiera: 6

**Key Pattern:** Plugin implements standard interfaces → manager handles registration/initialization → endpoints use manager to query all sources → data aggregation by priority.

### Critical Data Flows

**Multi-Source Inventory:** Request aggregates from Bolt (priority 10) + PuppetDB (priority 10) + Puppetserver (priority 20). Duplicate nodes resolved by priority.

**Command Execution:** Request → Route (`routes/commands.ts`) → ExecutionQueue → BoltService.executeCommand() → StreamingExecutionManager streams output → DatabaseService stores in SQLite.

**Real-time Streaming:** Execution spawns subprocess, pipes stdout/stderr, client WebSocket receives chunks via `/api/stream/:executionId`.

## Development Workflows

### Setup & Running

```bash
# Install all dependencies (frontend + backend)
npm run install:all

# Development: Run both servers separately
npm run dev:backend      # Port 3000, watches src/
npm run dev:frontend     # Port 5173, Vite HMR

# Production-like: Build frontend, serve via backend
npm run build:frontend && npm run copy:frontend && npm run dev:backend
# Access at http://localhost:3000
```

### Testing

- **Backend Unit Tests:** `npm test --workspace=backend` (Vitest, see [backend/vitest.config.ts](backend/vitest.config.ts))
- **Frontend Tests:** `npm test --workspace=frontend`
- **E2E Tests:** `npm run test:e2e` (Playwright, [e2e/](e2e/))
- **Linting:** `npm run lint` (ESLint with `@eslint/js`, TypeScript strict mode)
- **Type Checking:** `npx tsc --noEmit` in both directories

### Build & CI/CD

- **GitHub Actions** (`[.github/workflows/ci.yml](.github/workflows/ci.yml)`): Lint, type-check, test, build on every PR/push
- **Docker:** Multi-arch builds via `scripts/docker-build-multiarch.sh`
- **Versioning:** Update via `scripts/update-version.js` (affects package.json, package-lock.json)

## Code Patterns & Conventions

### Backend (Express + TypeScript)

**Error Handling:**

- Define domain-specific errors in `backend/src/errors/` (e.g., `BoltExecutionError`, `CommandNotAllowedError`)
- Errors extend base class with statusCode property
- Global error middleware (`errorHandler.ts`) catches all errors, formats response based on error type
- Use `asyncHandler()` wrapper in routes to handle Promise rejections
- **Bolt Task Errors:** Extract both `_output` and `_error` fields from failed tasks—combine for comprehensive error messages

**Configuration & Validation:**

- Use `ConfigService` to load/validate environment variables with Zod schemas (`backend/src/config/schema.ts`)
- All integrations read config via `ConfigService.getConfig()`
- Sensitive values (tokens, certs) load from env, never hardcoded

**Services Architecture:**

- `BoltService`: Spawns Bolt CLI, parses JSON output, implements streaming callbacks and caching (30s inventory, 5m facts TTL)
- `DatabaseService`: SQLite schema management, migrations
- `ExecutionRepository`: CRUD operations for execution history with composite indexes
- `IntegrationManager`: Registry and lifecycle management for all plugins
- `CommandWhitelistService`: Validates commands against whitelist (security-critical)
- `ExecutionQueue`: FIFO queue limiting concurrent executions (default: 5 concurrent, 50 max queue)
- `LoggerService`: Structured logging with component, integration, operation metadata
- `ExpertModeService`: Debug info attachment with correlation IDs linking frontend/backend logs

### Frontend (Svelte 5 + Vite)

**Stores & State:**

- Reactive stores in `src/lib/` (e.g., `router.svelte.ts`, `expertMode.svelte.ts`, `theme.svelte.ts`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [example42/pabawi](https://github.com/example42/pabawi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
