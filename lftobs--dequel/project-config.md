---
trigger: always_on
description: Self-hosted deployment platform. Deploy apps from Git, ZIP, or Docker Compose with zero infrastructure setup.
---

# Dequel

Self-hosted deployment platform. Deploy apps from Git, ZIP, or Docker Compose with zero infrastructure setup.

## Tech Stack

- **Runtime**: Bun
- **Backend**: ElysiaJS (`apps/api/`) — TypeScript, port 3001
- **Frontend**: React 18 + Vite + TanStack Router + TanStack Query (`apps/web/`) — port 3000
- **Docs**: Astro 4 + Tailwind CSS (`apps/docs/`) — deployed to Vercel
- **Database**: SQLite (`data/dequel.db`) — raw SQL
- **Queue**: Redis (`ioredis`) for async job queue
- **Container build**: Railpack CLI + BuildKit daemon
- **Container runtime**: Docker Engine API (mounted Docker socket)
- **Ingress**: Caddy (dynamic route files + auto SSL)

## Architecture

```
Caddy ──▶ API ──▶ Buildkit
  │          │
  ▼          ▼
 Web      SQLite    Redis

Observability: cAdvisor → Prometheus → Grafana / Loki
```

Services run in Docker Compose: Caddy, API, Web, Buildkit, Redis, cAdvisor, Prometheus, Loki, Promtail, Grafana.

## Directory Structure

```
├── apps/
│   ├── api/          # Backend orchestrator (Bun + ElysiaJS)
│   │   ├── src/
│   │   │   ├── api/          # Route handlers
│   │   │   ├── db/           # Database migrations + queries
│   │   │   ├── orchestrator/ # Deployment orchestration logic
│   │   │   ├── scaling/      # Auto-scaling engine
│   │   │   ├── monitoring/   # Alert evaluation, Prometheus metrics
│   │   │   ├── servers/      # Server management
│   │   │   └── git/          # Git operations
│   │   └── Dockerfile
│   ├── web/          # React dashboard (Vite + TanStack)
│   │   ├── src/
│   │   │   ├── routes/       # TanStack Router route definitions
│   │   │   ├── components/   # Shared UI components
│   │   │   ├── api/          # API client
│   │   │   └── hooks/        # Custom hooks
│   │   └── Dockerfile
│   └── docs/         # Documentation site (Astro)
│       ├── src/
│       │   ├── content/docs/ # Markdown doc pages (content collection)
│       │   ├── layouts/      # Layout component with sidebar
│       │   ├── components/   # Landing page components
│       │   └── styles/       # Global CSS
│       └── vercel.json
├── infra/
│   ├── caddy/        # Caddyfile + dynamic route files
│   └── monitoring/   # Prometheus, Loki, Grafana configs
├── scripts/
│   ├── install.sh    # One-command install script
│   └── dequel        # CLI for managing the platform
├── data/             # SQLite database (persisted)
├── workspace/        # Build staging area
├── docker-compose.yml
├── VERSION           # Single source of truth for version
├── CHANGELOG.md      # Release history
└── AGENTS.md         # This file
```

## Key Files

| File | Purpose |
|------|---------|
| `apps/api/src/index.ts` | API entry point — bootstraps DB, queue, scaling engine, etc. |
| `apps/api/src/db/schema.ts` | Drizzle ORM schema definitions (all tables) |
| `apps/api/src/db/drizzle.ts` | Drizzle client wrapper (wraps `bun:sqlite`) |
| `apps/api/src/db/migrations/` | Drizzle Kit migration files (`drizzle-kit generate` outputs here) |
| `apps/api/drizzle.config.ts` | Drizzle Kit configuration |
| `apps/web/src/main.tsx` | Frontend entry point |
| `apps/web/src/routes/index.tsx` | TanStack Router tree definition |
| `apps/web/src/routes/Dashboard.tsx` | Main dashboard page |
| `apps/web/src/components/Layout.tsx` | Shared app layout (sidebar, header) |
| `apps/docs/src/layouts/Layout.astro` | Docs layout with sidebar (auto-generated from content collection) |
| `apps/docs/src/pages/docs/[...slug].astro` | Catch-all route rendering content collection entries |
| `apps/docs/src/content.config.ts` | Astro content collection schema |
| `scripts/install.sh` | Install script — downloads configs, pulls images, installs CLI |
| `scripts/dequel` | CLI tool — `start`, `stop`, `status`, `logs`, `update`, `uninstall` |
| `.github/workflows/release.yml` | On `v*` tag: build Docker images → ghcr.io, create GitHub Release |
| `.github/workflows/deploy-docs.yml` | On push to `main`/`dev` (docs changes): deploy to Vercel |

## Commands

```bash
# Development (API)
bun apps/api/src/index.ts

# Development (Web)
bun apps/web/src/main.tsx

# Inside apps/web/
bun run build        # Vite build
bun run dev          # Vite dev server

# Inside apps/api/
bun test             # Run tests

# Inside apps/docs/
bun run dev          # Astro dev server
bun run build        # Astro build

# Docker
docker compose up -d               # Start full stack
docker compose up -d --build       # Rebuild and start

# Install / Manage
curl -fsSL https://github.com/Lftobs/dequel/releases/latest/download/install.sh | bash
scripts/dequel start               # Start platform
scripts/dequel uninstall           # Remove everything (prompts)

# Drizzle migrations (run from apps/api/)
bunx drizzle-kit generate          # Generate migration from schema changes
bunx drizzle-kit push              # Push schema directly (dev only)

# Version sync
bun run sync-versions              # Syncs VERSION → sub-package.json files
```

## Code Conventions

- No comments in source code unless absolutely necessary
- no file should be above 500 lines of code...if it really is, refactor and split into smaller files properly managed in a folder not scattered across the codebase (proper feature grouping).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lftobs/dequel](https://github.com/Lftobs/dequel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
