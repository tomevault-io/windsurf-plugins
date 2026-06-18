---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Documentation First

Before implementing anything that involves a library, framework, or SDK (Next.js, NestJS, Prisma, Apollo, OpenAI, TanStack Query, Zod, etc.), always fetch current documentation via the context7 MCP server. Do not rely on training data — APIs change between versions.

## Commands

### Frontend (`frontend/`)
```bash
cd frontend
pnpm dev          # Start dev server on :3000
pnpm build        # Production build
pnpm lint         # ESLint via Next.js
pnpm typecheck    # TypeScript type check (tsc --noEmit)
pnpm test         # Vitest unit tests
pnpm analyze      # Build with @next/bundle-analyzer — opens .next/analyze/*.html
pnpm size         # size-limit bundle budget check (requires a prior build)
```

### Backend Services (each in `backend/<service>/`)
```bash
# Development
pnpm start        # ts-node (auth-service, text-service, ai-orchestrator)
pnpm start:dev    # ts-node-dev with --respawn (api-gateway, stats-service)

# Build & Production
pnpm build        # prisma generate && tsc → dist/  (auth, text, audio)
                  # tsc → dist/  (api-gateway, stats-service, ai-orchestrator)
pnpm typecheck    # tsc --noEmit (no output, fast type check)
pnpm start:prod   # node dist/main.js  (node dist/server.js for stats-service)

# Database (services with Prisma: auth, text, audio, ai-orchestrator)
pnpm prisma:generate   # Generate Prisma client manually
pnpm prisma:migrate    # Create + apply migration (dev only)
# Production migrations run automatically via entrypoint.sh on container startup
# ai-orchestrator: run prisma:generate after adding DATABASE_URL_AI_ORCHESTRATOR to .env
```

### Tests
```bash
# Unit tests — Vitest (all services)
pnpm --filter frontend test
pnpm --filter audio-service test
pnpm --filter stats-service test
pnpm --filter auth-service test
pnpm --filter ai-orchestrator test
pnpm --filter text-service test
pnpm --filter api-gateway test
pnpm -r test                       # run all at once

# End-to-end smoke test (requires curl + jq, all services must be running)
bash scripts/e2e-test.sh
# Against a remote host:
GW_URL=http://your-host:8080 bash scripts/e2e-test.sh
```

### Full Stack
```bash
docker-compose up -d       # Build and start all services (local dev)
docker-compose ps          # Check health

# Production deploy (uses pre-built GHCR images)
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d --no-build
```

### Root (pnpm workspace)
```bash
pnpm --filter api-gateway build       # Run script in specific workspace package
pnpm --filter frontend dev
```

## Architecture

### Request Flow
```
Frontend (Next.js :3000)
  → /api/graphql (Next.js route → proxies to api-gateway)
    → API Gateway (:8080, NestJS, Apollo Federation Gateway)
      → auth-service    (:4001)
      → text-service    (:4002)  → ai-orchestrator (:4005)
      → audio-service   (:4003)  → ai-orchestrator (:4005)
      → stats-service   (:4004)  → text-service (:4002) + audio-service (:4003)
```

All frontend GraphQL goes through the Next.js `/api/graphql` route, which proxies to the API Gateway. The frontend uses persisted queries with SHA-256 hashes, falling back to full query strings if the persisted query isn't found.

**Direct inter-service HTTP calls are a normal part of the architecture** — not just delegation edge cases:
- `text-service` → `ai-orchestrator` (REST): text analysis, task generation
- `audio-service` → `ai-orchestrator` (REST): transcription (`POST /audio/transcribe`), pronunciation analysis (`POST /audio/pronunciation/analyze`)
- `stats-service` → `text-service` (REST `GET /text/by-language`) + `audio-service` (REST `GET /audio/by-language`): stats aggregation — stats-service has no database of its own

### API Gateway (`backend/api-gateway/`)
- **Apollo Federation Gateway** using `IntrospectAndCompose` — federates **only** auth-service and text-service (the two GraphQL subgraphs); polls schemas every 10s
- **audio-service and stats-service are REST-only** — they are NOT in the subgraph list; never add them
- Applies **JWT auth** via `JwtAuthGuard` on HTTP routes (decorating public routes with `@Public()`); GraphQL requests use `AuthContextService` inside the Apollo `expressMiddleware` context function
- Passes auth context downstream via HTTP headers: `x-user-id`, `x-user-role`, `x-user-language`, `x-trace-id`
- **Circuit breaker** via `opossum` (10s timeout, 50% error threshold, 30s reset)
- **Rate limiting** handled by Nginx; `@nestjs/throttler` removed due to NestJS 11 incompatibility — do not re-add it

### Backend Services

| Service | Port | Framework | GraphQL subgraph? | Database | Notes |
|---------|------|-----------|-------------------|----------|-------|
| auth-service | 4001 | Raw Node.js `http` + `buildSubgraphSchema` | ✅ yes | `auth_db` | No NestJS, uses argon2, JWT via `jsonwebtoken`; exposes `users(limit,offset,cursor)` + `usersCount` queries (admin + internal-token gated); `updateLevel(level: CEFRLevel!): User!` mutation (authenticated users); `CEFRLevel` enum |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sasha39612/Lingua_Pro](https://github.com/sasha39612/Lingua_Pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
