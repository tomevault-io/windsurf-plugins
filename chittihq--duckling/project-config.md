---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a high-performance DuckDB server service that replicates data from MySQL databases using a Sequential Appender architecture with ACID transactions. It provides a scalable analytical layer over operational MySQL data with 5-10x better query performance than traditional approaches.

## Monorepo Structure

This project uses **pnpm workspaces** to manage multiple packages:

```
duckling/
├── packages/
│   ├── server/          # @chittihq/duckling-server - DuckDB server with MySQL replication
│   │   ├── src/         # TypeScript source code
│   │   ├── public/      # Static web dashboard files
│   │   ├── dist/        # Compiled JavaScript (after build)
│   │   └── package.json
│   ├── frontend/        # @chittihq/duckling-frontend - Nuxt 4 web dashboard
│   │   ├── app/         # Nuxt pages, components, layouts
│   │   ├── assets/      # CSS and static assets
│   │   └── package.json
│   ├── sdk/             # @chittihq/duckling - WebSocket SDK for DuckDB queries
│   │   ├── src/         # SDK source code
│   │   ├── examples/    # Usage examples
│   │   └── package.json
│   └── shared/          # @chittihq/duckling-shared - Shared TypeScript types
│       ├── src/         # Shared types and constants
│       └── package.json
├── pnpm-workspace.yaml  # Workspace configuration
├── package.json         # Root package with workspace scripts
└── docker-compose.yml   # Development containers for server + frontend
```

### Package Dependencies

- **server** depends on: `shared`
- **frontend** depends on: `shared`, `sdk`
- **sdk** depends on: `shared`
- **shared** has no dependencies (foundation package)

## Development Commands

### Package Management
- Use `pnpm` instead of npm for all dependency management
- Install all dependencies: `pnpm install` (from root)
- Add package to specific workspace: `pnpm --filter @chittihq/duckling-server add <package-name>`
- Add package to root: `pnpm add -w <package-name>`

### Build & Development

**IMPORTANT:** Since development happens in Docker containers, always run build/lint commands **inside the running container**:

```bash
# ✅ CORRECT - Run commands inside Docker containers:
docker exec duckling-server pnpm run build:server
docker exec duckling-frontend pnpm run build:frontend
docker exec duckling-server pnpm run lint
docker exec duckling-frontend pnpm run lint

# ❌ WRONG - Don't run locally (different Node versions, missing dependencies):
pnpm run build:server
pnpm run lint
```

**Build Commands (run inside container):**
- Build server: `docker exec duckling-server pnpm run build:server`
- Build frontend: `docker exec duckling-frontend pnpm run build:frontend`
- Build SDK: `docker exec duckling-server pnpm run build:sdk`
- Build shared: `docker exec duckling-server pnpm run build:shared`
- Build all: `docker exec duckling-server pnpm run build`

**Lint Commands (run inside container):**
- Lint server: `docker exec duckling-server pnpm run lint`
- Lint frontend: `docker exec duckling-frontend pnpm run lint`

**Development Mode:**
- Development runs automatically in containers via docker-compose (hot reload enabled)
- No need to manually run `pnpm run dev` - containers start with dev mode by default
- Check logs: `docker-compose logs -f duckdb-server` or `docker-compose logs -f duckdb-frontend`

### CLI Operations (Server)

Run CLI commands **inside the Docker container**:

```bash
# Run CLI commands inside container
docker exec duckling-server node packages/server/dist/cli.js <command>

# Examples:
docker exec duckling-server node packages/server/dist/cli.js health
docker exec duckling-server node packages/server/dist/cli.js sync
docker exec duckling-server node packages/server/dist/cli.js tables
docker exec duckling-server node packages/server/dist/cli.js query "SELECT COUNT(*) FROM User"
```

**Available CLI Commands:**
- Health check: `docker exec duckling-server node packages/server/dist/cli.js health`
- Full sync: `docker exec duckling-server node packages/server/dist/cli.js sync`
- Incremental sync: `docker exec duckling-server node packages/server/dist/cli.js sync-incremental`
- System status: `docker exec duckling-server node packages/server/dist/cli.js status`
- Validate data: `docker exec duckling-server node packages/server/dist/cli.js validate`
- List tables: `docker exec duckling-server node packages/server/dist/cli.js tables`
- Execute query: `docker exec duckling-server node packages/server/dist/cli.js query "SELECT * FROM table_name"`

### MySQL Query Utility

Direct MySQL query execution (bypasses DuckDB, queries source directly):

```bash
# Run inside Docker container
docker exec duckling-server node scripts/mysql.js "SELECT COUNT(*) FROM User"
docker exec duckling-server node scripts/mysql.js "SHOW TABLES"
```

Returns JSON results. Uses `MYSQL_CONNECTION_STRING` from environment.

### Docker Development

**Development Environment:**
All development happens in a **docker-compose environment running in dev mode** with hot reload enabled:
- Both server and frontend run inside Docker containers
- Volume mounts sync local code changes to containers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chittihq/duckling](https://github.com/chittihq/duckling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-24 -->
