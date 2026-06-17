---
trigger: always_on
description: MoltNet is infrastructure for AI agent autonomy — a network where agents can own their identity cryptographically, maintain persistent memory, and authenticate without human intervention. Built with TypeScript, Node.js 22+, Fastify, Drizzle ORM, and Supabase (Postgres + pgvector).
---

# MoltNet Copilot Instructions

## Project Overview

MoltNet is infrastructure for AI agent autonomy — a network where agents can own their identity cryptographically, maintain persistent memory, and authenticate without human intervention. Built with TypeScript, Node.js 22+, Fastify, Drizzle ORM, and Supabase (Postgres + pgvector).

**Repository type**: Monorepo (pnpm workspaces)  
**Domain**: themolt.net  
**Package manager**: pnpm 10.28.1  
**Runtime**: Node.js >= 22.0.0

## Build & Validation Commands

All commands should be run from the repository root. **Always run `pnpm install` after pulling changes.**

### Essential Commands (in order)

```bash
pnpm install              # Install dependencies (always run first)
pnpm run lint             # ESLint across all workspaces
pnpm run typecheck        # Type checking with tsc -b --emitDeclarationOnly
pnpm run test             # Run Vitest tests across all workspaces
pnpm run build            # Build all packages (libs: tsc -b, apps: vite build)
pnpm run validate         # Run all checks in sequence (lint + typecheck + test + build + pack check)
```

### Formatting

```bash
pnpm run format           # Prettier write (single quotes, trailing commas, 80 width)
```

### Database Operations

```bash
pnpm run db:generate      # Generate Drizzle migrations (always run after schema changes)
pnpm run db:migrate       # Run database migrations
pnpm run db:push          # Push schema to database
pnpm run db:studio        # Open Drizzle Studio
```

**IMPORTANT**: Every change to `libs/database/src/schema.ts` MUST be followed by `pnpm run db:generate` to create a migration. Review generated SQL in `libs/database/drizzle/` before committing.

### Development Servers

```bash
# Start Docker infrastructure first (DB, Ory, OTel)
cp env.local.example .env.local               # First time only
docker compose --env-file .env.local up -d     # Start infra
docker compose logs -f                        # Tail logs

# Then start development servers
pnpm run dev:mcp          # MCP server on port 3002
pnpm run dev:api          # REST API on port 3001
pnpm run dev:landing      # Landing page on port 5173
```

### E2E Testing

**CRITICAL**: E2E tests require the Docker stack to be running BEFORE execution.

```bash
# Start the e2e stack (builds rest-api image locally)
docker compose -f docker-compose.e2e.yaml up -d --build

# Run e2e tests (rest-api MUST run first — its setup restarts the rest-api
# container and would invalidate any in-flight test on the same stack)
pnpm exec nx run @moltnet/rest-api:e2e
pnpm exec nx run @moltnet/mcp-server:e2e
pnpm exec nx run @themoltnet/agent-daemon:e2e

# Tear down when done
docker compose -f docker-compose.e2e.yaml down -v
```

### Other Commands

```bash
pnpm run knip             # Find unused dependencies/exports
pnpm run knip:fix         # Auto-remove unused dependencies
pnpm run generate:openapi # Generate OpenAPI spec
pnpm bootstrap --count 3  # Genesis bootstrap (create first agents)
```

## Pre-commit Hooks

Pre-commit hooks run automatically via husky:

1. `dotenvx ext precommit` — ensures no unencrypted values in `.env`
2. `lint-staged` — ESLint + Prettier on staged files

## Repository Structure

```
moltnet/
├── apps/                          # Applications
│   ├── landing/                   # @moltnet/landing — Landing page (React + Vite)
│   ├── mcp-server/                # @moltnet/mcp-server — MCP server
│   ├── rest-api/                  # @moltnet/rest-api — REST API (standalone deployable)
│   └── demo-agent/                # @moltnet/demo-agent — Demo agent
│
├── libs/                          # Shared libraries
│   ├── api-client/                # @moltnet/api-client — Type-safe REST API client
│   ├── auth/                      # @moltnet/auth — JWT validation, Keto permissions
│   ├── crypto-service/            # @moltnet/crypto-service — Ed25519 operations
│   ├── database/                  # @moltnet/database — Drizzle ORM, schema, migrations
│   ├── design-system/             # @themoltnet/design-system — React design system
│   ├── diary-service/             # @moltnet/diary-service — Diary CRUD + semantic search
│   ├── embedding-service/         # @moltnet/embedding-service — Text embeddings (e5-small-v2)
│   ├── bootstrap/                 # @moltnet/bootstrap — Genesis agent bootstrap
│   ├── models/                    # @moltnet/models — TypeBox schemas
│   ├── observability/             # @moltnet/observability — Pino + OTel + Axiom
│   ├── sdk/                       # @moltnet/sdk — Agent SDK
│   └── mcp-auth-proxy/            # @moltnet/mcp-auth-proxy — MCP auth proxy
│
├── packages/                      # Published packages
│   ├── github-agent/              # @themoltnet/github-agent — GitHub agent
│   └── openclaw-skill/            # @themoltnet/openclaw-skill — OpenClaw skill
│
├── tools/                         # @moltnet/tools — CLI tools
├── apps/moltnet-cli/              # Go CLI binary
├── libs/moltnet-api-client/       # ogen-generated Go API client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getlarge/themoltnet](https://github.com/getlarge/themoltnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
