---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

bugzkit is a production-ready full-stack web application template with a Spring Boot backend and SvelteKit frontend, containerized with Docker.

## Common Commands

### Backend (Spring Boot — `backend/spring-boot/`)

| Task                 | Command                                                  |
| -------------------- | -------------------------------------------------------- |
| Build (skip tests)   | `./mvnw -B clean install -DskipTests`                    |
| Run (dev profile)    | `./mvnw spring-boot:run -Dspring-boot.run.profiles=dev`  |
| All tests + coverage | `./mvnw -B clean verify` (report: `target/site/jacoco/`) |
| Unit tests only      | `./mvnw -B test`                                         |
| Format check         | `./mvnw -B spotless:check`                               |
| Format fix           | `./mvnw spotless:apply`                                  |

Integration tests use `*IT.java` suffix and run via Maven Failsafe. They require Testcontainers (Docker must be running).

### Frontend (SvelteKit — `frontend/svelte-kit/`)

| Task       | Command                     |
| ---------- | --------------------------- |
| Install    | `pnpm install`              |
| Dev server | `pnpm run dev`              |
| Build      | `pnpm run build`            |
| Lint       | `pnpm run lint`             |
| Lint fix   | `pnpm run lint:fix`         |
| Unit tests | `pnpm run test:unit`        |
| E2E tests  | `pnpm run test:integration` |
| Type check | `pnpm run check`            |

### Docker (from repo root)

- **DB only** (for local backend dev): `docker-compose -f docker-compose-infra.dev.yml up -d`
- **Full stack**: `docker-compose -f docker-compose.dev.yml up --build -d`
- **API + DB only**: `docker-compose -f docker-compose-api.dev.yml up --build -d`

## Architecture

### Backend (`backend/spring-boot/src/main/java/org/bugzkit/api/`)

Organized by feature module, each with controller/service/repository/payload layers:

- **`auth/`** — JWT authentication, OAuth2 (Google), token management (access, refresh, verify-email, reset-password). Tokens stored in Redis. JWT sent via HTTP-only cookies.
- **`user/`** — User/Role JPA entities, repositories, MapStruct mappers for DTO conversion.
- **`admin/`** — Admin user management endpoints.
- **`shared/`** — Cross-cutting: security config (`SecurityConfig.java`), error handling (standardized error codes in `error-codes.properties` mapped to frontend i18n keys), email service (MJML templates), data initialization (`DataInit.java`), Redis config, i18n messages, rate limiting (`shared/ratelimit/`).

Security: Stateless sessions, JWT filter chain, role-based access (ADMIN/USER). Public endpoints are whitelisted in `SecurityConfig.java`; everything else requires authentication.

**Important design decisions:**
- OAuth2 users (Google) have **null `username` and `password`** — any code touching User fields must null-check these.
- Device revocation (`DeviceService.revoke()`) intentionally invalidates **all** access tokens via `UserBlacklist`, not just the revoked device's token. Active sessions auto-refresh silently via their refresh tokens. This is by design to avoid an extra Redis lookup on every request.
- Access token validation (`AccessTokenService.check()`) runs on every authenticated request — keep it minimal (JWT verify + blacklist check + user blacklist check).
- Token data (userId, roles, deviceId) must only be extracted from a JWT **after** signature verification.
- **Rate limiting** uses Bucket4j (token bucket algorithm) with Redis (Lettuce) as the distributed bucket store. Apply the `@RateLimit(requests, duration)` annotation to controller methods. Buckets are keyed as `rate-limit:{EndpointName}:{ClientIP}` in Redis. Configurable via `rate-limit.enabled` (defaults to `true`, disabled in tests, except @RateLimitIT). Rate-limited endpoints return `429 Too Many Requests` with a `Retry-After` header.

### Frontend (`frontend/svelte-kit/src/`)

- **`routes/`** — SvelteKit file-based routing: `/auth/*` (sign-in, sign-up, forgot-password, etc.), `/profile/`, `/user/[name]/`, `/admin/user/`.
- **`lib/server/apis/api.ts`** — HTTP client (`makeRequest()`) that communicates with the backend, handling cookie-based auth and Set-Cookie propagation.
- **`lib/components/ui/`** — shadcn-svelte component library.
- **`lib/models/`** — TypeScript interfaces for API types.
- **`hooks.server.ts`** — Middleware chain: i18n → JWT verification/refresh → protected route guards.

Form handling uses Superforms + Zod schemas (defined in per-route `schema.ts` files). Error codes from the API are mapped to i18n message keys (`API_ERROR_*`).

i18n: Paraglide.js with English (en) and Serbian (sr). Message files in `messages/`. Generated code in `src/lib/paraglide/` (gitignored).

### Frontend-Backend Communication

The frontend server-side (`hooks.server.ts` and server actions) calls the backend API via `makeRequest()`. Auth tokens flow as cookies: the backend sets them, `makeRequest()` forwards them, and `hooks.server.ts` verifies/refreshes them using the shared `JWT_SECRET`.

## Code Style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [while1618/bugzkit](https://github.com/while1618/bugzkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
