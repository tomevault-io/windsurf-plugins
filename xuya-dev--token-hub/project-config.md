---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TokenHub is an API sharing/proxy platform with revenue splitting. It relays requests from consumers (using `sk-hub-xxx` keys) to upstream LLM providers (OpenAI, Anthropic, Google, DeepSeek, etc.) with load balancing, format conversion, rate limiting, and per-token billing. Channel owners earn a configurable revenue share when their channels are used.

**Stack**: Java 21 / Spring Boot 3.3.5 backend + Vue 3 / Vite 8 / Naive UI frontend (monorepo in `tokenhub-ui/`).

## Build & Run Commands

### Backend (Maven)

```bash
# Run in dev mode (requires local MySQL + Redis, see application-dev.yml defaults)
mvn spring-boot:run

# Package (skips tests — no test suite exists)
mvn clean package -DskipTests

# Run the jar
java -jar target/tokenhub-1.0.3.jar
```

App starts on port 3000. Dev profile is active by default (`application-dev.yml`).

### Frontend (pnpm, run from `tokenhub-ui/`)

```bash
pnpm install              # install deps (workspace monorepo)
pnpm dev                  # dev server on port 9527, proxies API to localhost:3000
pnpm build                # production build -> dist/
pnpm lint                 # oxlint --fix && eslint --fix .
pnpm fmt                  # oxfmt
pnpm typecheck            # vue-tsc --noEmit --skipLibCheck
```

Pre-commit hook runs: `typecheck && lint && fmt`.

### Docker

```bash
# Full stack (MySQL 8 + Redis 7 + app) — set env vars in .env first
docker compose up --build

# Standalone (external DB/Redis)
docker compose -f docker-compose.standalone.yml up --build
```

Required env vars for prod: `JWT_SECRET_KEY`, `ENCRYPTION_KEY`, `CORS_ALLOWED_ORIGINS`, `MYSQL_PASSWORD`, `REDIS_PASSWORD`.

## Architecture

### Backend — Domain-Organized Layers

Each domain package under `dev.xuya.tokenhub.*` follows: `controller/` → `service/` → `mapper/` with `model/` containing `entity` (DO), `dto` (input), `vo` (output).

Key domains: `relay/`, `billing/`, `channel/`, `apikey/`, `auth/`, `user/`, `log/`, `redemption/`, `checkin/`, `email/`, `system/`.

### Relay Engine (`relay/`) — The Core Subsystem

`RelayDispatcher` orchestrates the full request lifecycle:
1. Extract model from request body
2. Rate limit check (Redis sliding window, per API key)
3. Pre-consume quota (optimistic lock: `UPDATE balance WHERE balance >= ?`)
4. Channel selection via `DistributorService` (priority groups → weighted random → health filtering)
5. Format conversion if needed (OpenAI ↔ Claude via `OpenAiClaudeConverter`, `ClaudeStreamToOpenAiConverter`)
6. Stream/non-stream relay via OkHttp to upstream
7. Retry loop (excludes failed channels, rotates multi-keys)
8. Settle billing (actual usage vs. pre-consumed) or refund on failure

Billing is three-phase: **pre-consume → settle → refund**. All monetary values use `BigDecimal` with 8-decimal precision (USD).

### Channel Health & Load Balancing

- `DistributorService`: priority-based grouping + weighted random selection
- `ChannelHealthTracker`: Redis-backed failure counters with configurable threshold and TTL
- Multi-key rotation: channels store multiple upstream keys (newline-separated), round-robin with 60s unhealthy marking

### Format Conversion (`relay/converter/`)

`RelayFormat` enum: `OPENAI`, `CLAUDE`, `OPENAI_RESPONSES`. Converters handle request body, response JSON, and SSE stream translation between OpenAI Chat Completions and Anthropic Messages formats.

### Security

- Channel upstream keys: AES-GCM encrypted at rest (`ChannelKeyCipher` in `config/`)
- User API keys: stored as SHA-256 hashes; raw key shown only once at creation
- Auth: Sa-Token with JWT + API Key plugin; `ApiKeyAuthFilter` as servlet filter
- Admin actuator endpoints gated by bearer token

### Database

Flyway manages migrations at `src/main/resources/db/migration/{vendor}/`. Supports MySQL and PostgreSQL via vendor-specific migration paths. 9 tables: `user`, `channel`, `ability`, `api_key`, `redemption`, `log`, `option`, `model_pricing`, `check_in`.

### Redis Usage

- Rate limiting (sliding window)
- Channel health tracking
- Pricing cache with pub/sub invalidation across instances
- Distributed locks via Redisson

### Frontend (`tokenhub-ui/`)

Based on SoybeanAdmin template. **pnpm workspace monorepo** — shared packages live in `packages/`:
- `@sa/alova` — HTTP client wrapper (Alova)
- `@sa/hooks` — shared Vue composables
- `@sa/materials` — reusable UI components
- `@sa/utils`, `@sa/color`, `@sa/axios`, `@sa/scripts`, `@sa/uno-preset`

Routing uses `@elegant-router/vue` (file-based route generation from `src/views/`). Static route mode with role-based access (`R_SUPER`, `R_ADMIN`). i18n supports zh-CN and en-US.

Key frontend directories: `src/views/` (pages), `src/service/` (API layer), `src/store/` (Pinia stores), `src/locales/` (i18n), `src/layouts/` (shell layout).

The Spring Boot app serves the SPA frontend from `resources/static/` with SPA fallback via `SpaWebMvcConfig`.

## API Endpoints

Relay (OpenAI/Anthropic-compatible):
- `POST /v1/chat/completions` — OpenAI Chat Completions
- `POST /v1/messages` — Anthropic Messages
- `POST /v1/responses` — OpenAI Responses API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xuya-dev/token-hub](https://github.com/xuya-dev/token-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
