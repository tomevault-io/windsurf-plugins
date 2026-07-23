---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

Polyglot monorepo for **Byclaw (BeyondAI)** — an enterprise AI application platform with agent management, multi-turn chat, knowledge bases (RAG), and tool orchestration. Product name: "鲸智百应".

Modules: `byclaw-fe/` (React frontend), `byclaw-be/` (Java backend), `byclaw-exe/` (Python CLIs, stub). Supporting dirs: `byclaw-data/`, `byclaw-qa/`, `docs/`, `examples/`, `scripts/`.

## Build, lint, and test commands

### Frontend (`byclaw-fe/`)

```bash
cd byclaw-fe && pnpm install --frozen-lockfile   # install deps (pnpm 8.15+, Node 18.20+)
pnpm run dev                                      # dev server on :8000, proxies /byaiService → :8086
pnpm run build                                    # production build (uses `max build`)
pnpm run lint                                     # lint all (ESLint + stylelint + prettier)
pnpm run lint:fix                                 # auto-fix lint issues
pnpm run test                                     # run all tests (Jest, passWithNoTests)
pnpm run test:unit -- --testPathPattern=<pattern> # run a single test file
pnpm run test:coverage                            # coverage report
```

### Backend (`byclaw-be/`)

```bash
mvn -B -f byclaw-be/pom.xml compile               # compile (Java 21 required)
mvn -B -f byclaw-be/pom.xml test                   # run tests
mvn -B -f byclaw-be/pom.xml verify                 # full verification
mvn -B -f byclaw-be/pom.xml package -DskipTests    # package JAR
mvn -B -f byclaw-be/pom.xml spring-boot:run        # run (reads config/ + .env)
```

### Python (`byclaw-exe/`, when initialized)

```bash
cd byclaw-exe && pip install -e ".[dev]" && ruff check . && pytest
```

### One-command start

```bash
./scripts/start.sh --all              # start all modules
./scripts/start.sh --fe               # frontend only
./scripts/start.sh --be               # backend only
./scripts/start.sh --install --fe     # install deps first, then start FE
```

Logs go to `logs/<module>.log`. The script traps SIGTERM/SIGINT for cleanup.

## Environment configuration

Copy `.env.example` to `.env` and fill in values. Key variables: `DB_URL`, `DB_USER`, `DB_PASS`, `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `SERVER_PORT` (default 8086), `MID_FTP_*` for SFTP file storage, `INIT_USER_AUTH_RESOURCES_REDIS_ENABLED`.

Backend config lives in `byclaw-be/config/application.properties` — it reads from env vars with `${VAR:default}` syntax (no YAML profiles). The app auto-discovers the `config/` directory by walking up from CWD, so `mvn spring-boot:run` works from either the repo root or `byclaw-be/`.

## Architecture

### Frontend (React / Umi Max)

- **Framework**: Umi Max 4.4.2, React 18.2.0, Ant Design 5.27, TypeScript 5
- **State management**: DVA (Redux wrapper); models in `src/models/`, auto-registered by Umi
- **Data fetching**: Axios + `@tanstack/react-query` 4.24
- **Routing**: `byclaw-fe/config/route.config.ts` — main areas: chat hub (`/`), manager (`/manager/*`), mobile (`/mobile/*`), SSO (`/single`)
- **Config**: `.umirc.ts` — proxy, aliases, locale, webpack overrides, `umi-plugin-keep-alive` for component state persistence across navigation
- **Path aliases**: `@` and `@beyond` both resolve to `src/`
- **Locale**: default `zh-CN`, stored in localStorage, injected into every API request header
- **Dev proxy**: `/{base}byaiService` → `http://localhost:8086`

**Theming (critical)**: Ant Design uses custom prefix `beyond`, NOT the default `ant`. All component CSS selectors are `.beyond-*` instead of `.ant-*`. Theme overrides in `src/styles/antdDefaultTheme.ts` and `config/managerLessTheme.ts`.

**API request layer** (`src/service/common/request.ts`):
- Axios with request/response interceptors
- **HMAC request signing**: every request is signed via `generateSignature()` before sending
- **Token management**: reads from localStorage keys `tokenKey`, `ssotokenKey`, `sessionKey`
- **Response format**: expects `{ code, msg, data }` — code 0 = success; non-zero triggers error modal
- **Auth**: 401/403 → automatic logout redirect
- **Concurrent limiting**: `maxQuantity` option cancels oldest requests when threshold exceeded per URL
- Exports: `get()`, `post()`, `put()`, `del()` with TypeScript generics

### Backend (Java / Spring Boot 3)

- **Java 21**, Spring Boot 3.4.5, Spring Cloud 2024.0.1, Spring Security 6.5, MyBatis 3.5
- **Single-module project**: one `pom.xml` (parent is `spring-boot-starter-parent`)
- **Entry point**: `com.iwhalecloud.byai.ByaiServerApplication`
- **Context path**: `/byaiService`
- **Database**: PostgreSQL via Druid pool (maxActive=500, 800 worker threads)
- **Sessions**: Redis-backed (`spring.session.redis.namespace=common_system`)
- **File upload limit**: 2GB per file, 300MB form post

**Package layout** under `com.iwhalecloud.byai`:

| Package | Role |
|---------|------|
| `state` | Core business logic, DDD-style: `interfaces/controller`, `domain/` (agent, chat, auth, knowledge, toolkit, workspace), `application/service`, `infrastructure/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beyonai/ByClaw](https://github.com/beyonai/ByClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
