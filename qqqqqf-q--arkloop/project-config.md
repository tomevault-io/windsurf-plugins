---
trigger: always_on
description: Arkloop is an open-source platform for building conversational AI agents. It provides a managed runtime for LLM-powered agents with built-in tool execution, memory, sandboxed code execution, and multi-model routing.
---

# Arkloop

Arkloop is an open-source platform for building conversational AI agents. It provides a managed runtime for LLM-powered agents with built-in tool execution, memory, sandboxed code execution, and multi-model routing.

The codebase is a monorepo split into:

- **Go services** (`src/services/`): Backend microservices
- **Frontend apps** (`src/apps/`): React applications (Vite + TypeScript)
- **Personas** (`src/personas/`): Agent persona templates
- **Infrastructure** (`compose.yaml`): Docker Compose orchestration

## Architecture

```
Client -> Gateway (19000) -> API (19001) -> Worker
                                             |-> LLM (multi-model routing)
                                             |-> Sandbox (code execution)
                                             |-> OpenViking (19010, memory)
```

Infrastructure: PostgreSQL 16 (via PgBouncer) / Redis 7 / MinIO (S3)

## Backend Services

All Go services live under `src/services/` and share a `go.work` workspace (Go 1.26).

| Service | Port | Purpose |
|---------|------|---------|
| `api` | 19001 | Core REST API: auth, RBAC, billing, migrations, job scheduling |
| `gateway` | 19000 | Reverse proxy: rate limiting, geo-IP filtering, risk scoring |
| `worker` | - | Job execution: LLM routing, tool dispatch, agent loop, persona management |
| `sandbox` | - | Code execution: Firecracker VMs (Linux) or Docker containers (macOS/Windows) |
| `bridge` | - | Project bridge service |
| `shared` | - | Shared libraries: config, S3 abstraction, Redis utils, credit policies |

Each service follows a consistent layout:

```
cmd/          # Entry points (main.go)
internal/     # Private packages (DDD-style: app, data, http, domain logic)
Dockerfile
go.mod
```

### Backend Workflow

```bash
# Start infrastructure
docker compose up -d postgres redis minio pgbouncer

# Run migrations
cd src/services/api && go run ./cmd/migrate

# Run a service
cd src/services/api && go run ./cmd/api
cd src/services/gateway && go run ./cmd/gateway
cd src/services/worker && go run ./cmd/worker

# Tests
cd src/services/api && go test ./...
cd src/services/worker && go test ./...
```

### Key Patterns

- Services use dependency injection via composition in `internal/app/`
- Database access through repository pattern (`internal/data/`)
- Worker uses a pipeline architecture: routing -> entitlement -> memory -> agent loop
- Worker supports Lua scripting for custom agent logic (`agent.lua` in personas)
- Sandbox supports dual mode: Firecracker (production/Linux) and Docker (dev/macOS)

## Frontend Apps

All frontend apps live under `src/apps/` as a pnpm monorepo.

| App | Port | Purpose |
|-----|------|---------|
| `web` | 19080 | User-facing chat interface |
| `console` | 19081 | Admin dashboard (~35 management pages) |
| `console-lite` | 19082 | Admin dashboard (lightweight, default in compose) |
| `shared` | - | Shared package: API client, token storage, theme/locale, auth components |

Tech stack: React 19 / TypeScript 5.9 / Vite 7 / Tailwind CSS 4 / React Router 7

### Frontend Workflow

```bash
# Install dependencies (from repo root)
pnpm install

# Development
cd src/apps/web && pnpm dev
cd src/apps/console && pnpm dev

# Build & check
cd src/apps/web && pnpm build
cd src/apps/web && pnpm lint
cd src/apps/web && pnpm type-check

# Tests
cd src/apps/web && pnpm test
```

### Key Patterns

- Both apps proxy `/v1` API requests to the backend via Vite dev server config
- State management via React Context only (no Redux/Zustand)
- Shared package (`@arkloop/shared`) provides API client (`apiFetch`), in-memory token storage, theme/locale context factories (`ThemeProvider`/`useTheme`, `createLocaleContext`), and UI components (`AuthPage`, `BootstrapPage`, `SettingsModal`, `Toast`, `ErrorCallout`, `Turnstile`)
- i18n translations live in each app's `src/locales/`

## Personas

Agent persona templates under `src/personas/`. Each persona defines:

- `persona.yaml`: Configuration (id, tools, budget, temperature)
- `prompt.md`: System prompt
- `agent.lua` (optional): Custom agent loop logic

## Configuration

- Environment: `.env` file (see `.env.example` for all variables)
- Sandbox templates: `config/sandbox/templates.json`
- OpenViking: `config/openviking/ov.conf`

## Testing

- **Unit tests**: `go test ./...` per service, `pnpm test` per app
- **Smoke tests**: `tests/smoke/` (CI-only, requires full stack)
- **Benchmarks**: `tests/bench/` (load testing against running instance)

## Code Conventions

- Follow `CONTRIBUTING.md` for commit format, code style, and PR process
- Go: standard conventions, explicit error handling, short focused functions
- TypeScript: strict mode, ESLint, no `any` types
- Prefer editing existing files over creating new ones
- Dependencies inject through constructors; respect clean architecture boundaries

## Data Model

Organization 概念已移除 (migration 00118)，Account 是唯一的租户单元。

### Account

Account 是核心多租户单元，有两种类型:
- `personal`: 每个用户自动创建一个，作为默认账户
- `workspace`: 多用户共享的工作区账户

关键表: `accounts`, `account_memberships`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qqqqqf-q/Arkloop](https://github.com/qqqqqf-q/Arkloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
