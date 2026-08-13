---
trigger: always_on
description: > **NEVER commit directly to `main`.** All changes go through a branch and PR. No exceptions.
---

# stratif.io OSS

> **NEVER commit directly to `main`.** All changes go through a branch and PR. No exceptions.

stratif.io is a full-stack product analytics dashboard with a React/TypeScript frontend and Python/FastAPI backend. It connects to any SQL analytics warehouse (DuckDB, BigQuery, Snowflake, Redshift, etc.) via SQLGlot for dialect transpilation.

## Monorepo layout

```
apps/
  analytics/          # Main analytics frontend (React/Vite)
  design-docs/        # Design documentation site
  docs/               # Product documentation
  event-simulator/    # Event simulation tooling
packages/
  design-system/      # Shared UI component library (@stratif-io/design-system)
services/             # Python/FastAPI backend services
  analytics/          # Analytics API service
```

## Commands

```bash
# Development
bun run dev              # Frontend dev server (port 5173)
bun run build:lib        # Build design-system library (run before build)
bun run build            # TypeScript type-check + production bundle
uv run serve             # Backend server (port 8000)
uv run seed              # Seed configured connections from a preset (see services/event_simulator/README.md)

# Testing
bun run test:run         # Unit tests once (Vitest)
bun run test             # Unit tests in watch mode
bun run test:coverage    # Unit tests with coverage

# Quality checks (run before committing)
bun run lint             # ESLint (zero warnings allowed)
bun run format:check     # Prettier check
```

## Architecture

**Frontend** (`apps/analytics/frontend/`): React 18, Vite 6, Tailwind CSS v4, shadcn/ui, React Router v6

**Design system** (`packages/design-system/`): Shared components, design tokens, and utilities published as `@stratif-io/design-system`. Always run `bun run build:lib` before `bun run build`.

**Backend** (`services/analytics/`): FastAPI, pydantic-settings, SQLGlot for SQL transpilation. Product DB (users, connections, credentials) defaults to SQLite in local/dev but can be any SQL database in production.

### State management

- **Server state**: TanStack Query v5 — all API data goes through custom hooks (`apps/analytics/frontend/features/*/hooks/useXxxData.ts`). Never use raw `fetch` in components:
  ```typescript
  // CORRECT
  const { data, isLoading } = useTrendData({
    dateRange,
    selectedEvent,
    granularity,
  });
  // WRONG — never do this
  useEffect(() => {
    fetch("/api/trend")
      .then((r) => r.json())
      .then(setData);
  }, []);
  ```
- **Client state**: Zustand store (`apps/analytics/frontend/stores/app-store.ts`) — theme, dateRange, sidebarOpen, selectedEvent, selectedDevice. Persisted to localStorage.

### Feature structure

Each feature under `apps/analytics/frontend/features/` is self-contained:

```
apps/analytics/frontend/features/<feature>/
├── components/       # Feature-specific UI
├── hooks/            # TanStack Query data hooks
└── <Feature>Page.tsx # Page component
```

Shared components live in `packages/design-system/components/` and are imported via `@stratif-io/design-system`.

### Backend layers

- `services/analytics/api/` — FastAPI routers (trend, retention, events, paths, conversion, pivot, sessions)
- `services/analytics/services/` — Business logic (path_analyzer, transpiler)
- `services/analytics/db/` — DuckDB connection management and seeding
- `services/analytics/core/` — Auth (API key verification)

### API endpoints

All prefixed with `/api/`: `trend`, `retention`, `events`, `events/top`, `raw/events`, `raw/sessions`, `sessions/summary`, `paths`, `conversion`, `pivot`. WebSocket at `/ws`.

## Security

stratif.io stores encrypted credentials for client analytics databases. Security is non-negotiable.

### Credential storage

- Credentials encrypted with Fernet (AES-128-CBC + HMAC-SHA256) via `services/analytics/services/crypto.py`
- Encryption key: 32+ char string → SHA-256 → Fernet key
- Key stored in `STRATIFIO_ENCRYPTION_KEY` env var (never in code or git)
- Product DB: SQLite at `STRATIFIO_PRODUCT_DB_PATH` (never expose this file)

### Auth

- Passwords: bcrypt + SHA-256 pre-hash (`services/analytics/core/password.py`)
- Sessions: JWT in HTTP-only, Secure, SameSite=Lax cookie (`sio_session`)
- Rate limiting on login (10/min) and register (3/min) via slowapi

### Production config flags

- `STRATIFIO_DEBUG=false` (default) — hides `/docs` and `/redoc`
- `STRATIFIO_CORS_ORIGINS` — set to your exact frontend domain (not `*`)
- `STRATIFIO_ENCRYPTION_KEY` — must be 32+ chars; generate with `openssl rand -base64 32`

### Never do

- Never log credentials, tokens, or the encryption key
- Never commit `.env` files or the SQLite product DB
- Never use `STRATIFIO_DEBUG=true` in production

## Code conventions

- **Imports**: Use `@/` path alias for `apps/analytics/frontend/` imports; use `@stratif-io/design-system` for shared components
- **Styling**: Tailwind CSS v4 + `cn()` utility
- **Validation**: Zod schemas in `apps/analytics/frontend/lib/schemas/`
- **Types**: `apps/analytics/frontend/types/index.ts` or co-located with feature
- **API functions**: `apps/analytics/frontend/lib/api/queries.ts`
- **Charts**: Recharts wrappers in `apps/analytics/frontend/components/charts/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stratif-io/stratif.io](https://github.com/stratif-io/stratif.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
