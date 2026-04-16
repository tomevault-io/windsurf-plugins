---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Commands

```sh
# Install
pnpm install

# Dev servers
pnpm nx serve @temar/web                      # Next.js frontend (port 5173)
pnpm nx serve @temar/fsrs-service              # FSRS service (port 3334)
pnpm nx serve @temar/question-gen-service      # Question gen (port 3335)
pnpm nx serve @temar/answer-analysis-service   # Answer analysis (port 3336)
pnpm nx serve admin                              # Admin panel (port 3000)

# Build
pnpm nx build @temar/web                       # single project
pnpm nx run-many -t build                      # all projects
pnpm nx affected -t build                      # changed only

# Test
pnpm nx test @temar/fsrs-service               # single project
pnpm nx affected -t test                       # changed only

# Lint
pnpm nx lint @temar/web                        # single project
pnpm nx affected -t lint                       # changed only

# DB schema changes & migrations
# 1. Edit the schema in libs/db-client/src/schema/*.ts
# 2. Generate migration SQL (NEVER create SQL files manually):
npx drizzle-kit generate --config libs/db-client/drizzle.config.ts
# 3. Apply migration:
pnpm migrate                                   # runs drizzle-kit migrate from libs/db-client

# Docker
docker compose -f docker-compose.dev.yml up    # local dev
docker build --target web-app-service -t temar-web .
docker build --target fsrs-service -t temar-fsrs .
```

---

## Architecture

**Temar** is a spaced-repetition learning platform. Users organize study material as **Topics -> Notes -> Chunks**, create content via a **Lexical rich-text editor**, and review it through AI-powered question generation and FSRS-based scheduling.

### Core data flow

```
User <-> Next.js (web) <-> NestJS microservices <-> PostgreSQL
              |                    |
         Lexical editor       LLM providers (via question-gen / answer-analysis)
         (content CRUD)
```

- **Monorepo:** Nx 22.x + pnpm workspaces
- **Frontend:** Next.js 15 (App Router) + React 19 + Tailwind 4 + shadcn/ui (shared via `@temar/ui`)
- **Backend:** NestJS microservices, each with Swagger docs at `/api/docs`
- **Database:** PostgreSQL 16 + Drizzle ORM (schemas in `libs/db-client/src/schema/`)
- **AI:** Vercel AI SDK (`ai` package), multi-provider (OpenAI, Anthropic, Google), BYOK support
- **Editor:** Lexical (migrated from Plate.js) for both chunk content authoring and review answers
- **Payments:** Paddle v2 (migrated from Stripe), Pass-based billing
- **Deploy:** Docker multi-stage -> GHCR -> VPS + Caddy

### Applications (`apps/`)

| App                       | Port | Purpose                                                                    | Status                    |
| ------------------------- | ---- | -------------------------------------------------------------------------- | ------------------------- |
| `web`                     | 5173 | Next.js frontend -- dashboard, materials browser, review sessions, billing | **Active**                |
| `fsrs-service`            | 3334 | NestJS -- ts-fsrs engine, tracking cascades, review lifecycle              | **Active**                |
| `question-gen-service`    | 3335 | NestJS -- LLM question + rubric generation per chunk                       | **Active**                |
| `answer-analysis-service` | 3336 | NestJS -- LLM semantic answer evaluation, mapped to FSRS ratings           | **Active**                |
| `admin`                   | 3000 | Next.js admin panel -- analytics, AI model management                      | **Active**                |
| `notion_sync-service`     | 3333 | NestJS -- Notion sync (OAuth, webhooks, markdown conversion)               | **Retired, not deployed** |
| `api`                     | --   | NestJS API scaffold                                                        | **Vestigial, ignore**     |

### Libraries (`libs/`)

| Library            | Purpose                                                                                      |
| ------------------ | -------------------------------------------------------------------------------------------- |
| `db-client`        | Drizzle ORM client, all schema definitions, crypto utilities, re-exported Drizzle operators  |
| `shared-types`     | Cross-app TypeScript interfaces and DTOs (`ModelConfig`, `OperationType`, etc.)              |
| `pricing-service`  | In-memory cached pricing engine -- computes Pass costs, records usage with balance deduction |
| `payment-provider` | Strategy-pattern billing abstraction (Paddle adapter)                                        |
| `ui`               | Shared shadcn/ui component library -- 24 components used by both web and admin apps          |

### Inter-service communication

All HTTP REST with `x-api-key` headers. The web app calls services via server-side fetch wrappers (`fsrsServiceFetch`, `questionGenServiceFetch`, etc.) -- never from the browser. All NestJS services use `app.setGlobalPrefix('api')`, so endpoint env vars **must include `/api`** (e.g., `http://fsrs-service:3334/api`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leykun-gizaw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
