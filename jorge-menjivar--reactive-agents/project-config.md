---
trigger: always_on
description: This file provides guidance to AI assistants when working with code in this repository.
---

# Repository Guidelines

This file provides guidance to AI assistants when working with code in this repository.

## Project Structure & Module Organization

This is a **TypeScript monorepo** using pnpm workspaces with three packages:

```
packages/
├── web/           # Vite + TanStack Router SPA (React 19)
│   └── src/
│       ├── routes/        # TanStack Router file-based routes
│       ├── components/    # React components
│       ├── providers/     # React context providers
│       ├── hooks/         # Custom React hooks
│       └── api/           # API client functions
├── api/           # Hono API server (Node.js)
│   └── src/
│       ├── api/           # API routes
│       ├── ai-providers/  # AI provider integrations (40+)
│       ├── connectors/    # Database connectors
│       └── middlewares/   # Hono middlewares
└── shared/        # Shared types, Zod schemas, utilities
    └── src/
        ├── types/         # TypeScript types
        └── utils/         # Shared utilities
```

Other directories:
- **`tests/`**: Vitest suites mirroring packages (client, server, shared)
- **`examples/`**: Example implementations
- **`supabase/`**: Local dev DB config, migrations, `seed.sql`
- **`docker/`**: Docker configuration files

**Key path aliases**:
- `@web/*` - Web package (`packages/web/src/*`)
- `@api/*` or `@server/*` - API package (`packages/api/src/*`)
- `@shared/*` - Shared package (`packages/shared/src/*`)

## Essential Commands

**You must run these commands after modifying any file to ensure code quality:**
```bash
pnpm typecheck  # TypeScript type checking (uses Turborepo) - REQUIRED
pnpm check      # Biome linting and formatting - REQUIRED
pnpm check:fix  # Auto-fix linting and formatting issues
```

### Development Commands
```bash
# Installation and setup
pnpm install

# Database
supabase start  # Start local Supabase database
supabase stop   # Stop local database

# Development server (runs both web and API via Turborepo)
pnpm dev        # Start all dev servers in parallel
pnpm dev:web    # Start only web dev server (Vite on port 3000)
pnpm dev:api    # Start only API dev server (Hono on port 8787)

# Testing
pnpm test                      # Run all tests (excludes in-depth integration tests)
pnpm test path/to/test.ts      # Run specific test file
pnpm test:watch                # Run tests in watch mode

# In-depth integration tests (slower, more comprehensive)
INCLUDE_IN_DEPTH=true pnpm test

# Build (uses Turborepo with caching)
pnpm build      # Build all packages
pnpm build:web  # Build only web package
pnpm build:api  # Build only API package

# Code quality
pnpm lint       # Run linter
pnpm format     # Check formatting
pnpm format:fix # Auto-fix formatting

# API testing (all requests go through port 3000, proxied to API)
curl "http://localhost:3000/v1/endpoint" -H "Authorization: Bearer reactive-agents"
```

## Architecture

### Web Application (packages/web)
- **Framework**: Vite + TanStack Router (SPA mode)
- **Routing**: File-based routing in `src/routes/`
  - `_main.tsx` - Layout wrapper with sidebar
  - `$paramName` - Dynamic route parameters
  - `.index.tsx` - Index routes for parent paths
- **Auto-generated**: `routeTree.gen.ts` (do not edit, in .gitignore)

### API Server (packages/api)
- **Framework**: Hono web framework
- **Entry**: `src/server.ts` (Node.js) or `src/index.ts` (Cloudflare Workers)
- **Routes**: `src/api/v1/`

### Request Flow
```
Browser (port 3000) → Vite/nginx proxy → API (port 8787)
                           ↓
                    /v1/* requests proxied
```

In development, Vite proxies `/v1/*` requests to the API server.
In Docker, nginx handles the proxying.

## API Structure (Hono-based)

Key API endpoints:
- `/v1/chat/completions` - OpenAI-compatible chat API
- `/v1/reactive-agents/agents` - Agent management
- `/v1/reactive-agents/evaluations` - Dataset and evaluation management
- `/v1/reactive-agents/observability/logs` - Request logging

**Hono Syntax**: Always use chained method syntax for proper type inference:
```typescript
// Use this pattern:
const app = new Hono<AppEnv>().get().post().fetch();

// Instead of:
const app = new Hono<AppEnv>();
app.get();
app.post();
app.fetch();
```

## Database Integration (Supabase)

Uses **connector pattern** for data access:
- Abstract interfaces: `UserDataStorageConnector`, `LogsStorageConnector`
- Concrete implementation: Supabase connector
- All CRUD operations use Zod schema validation

Core data models:
- `Agent` - AI agent configurations
- `Dataset`/`Log` - Training/evaluation data with many-to-many relationships
- `EvaluationRun`/`LogOutput` - Model evaluation system
- `Feedback`/`ImprovedResponse` - User feedback loop

Database management:
- Migrations: `supabase/migrations/`
- Seed data: `supabase/seed.sql`
- Start/stop: `supabase start|stop`

## Coding Style & Naming Conventions

- **Language**: TypeScript, React 19, Vite, TanStack Router
- **Formatting via Biome**: 2-space indent, LF, single quotes, semicolons, import organize
  - Auto-fix: `pnpm check:fix` or `pnpm format:fix`
- **Files**: kebab-case for filenames (e.g., `add-logs-dialog.tsx`)
- **Components**: PascalCase exports
- **Paths**: prefer `@web`, `@api`, `@shared` over long relative paths


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jorge-menjivar/reactive-agents](https://github.com/jorge-menjivar/reactive-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
