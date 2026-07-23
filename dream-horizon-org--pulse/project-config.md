---
trigger: always_on
description: Guidance for Claude Code in this repo. This is the **map**; per-area conventions live in `.agents/rules/*.mdc` (auto-loaded by file type; also reachable as `.cursor/rules/` and `.claude/rules/` via symlinks).
---

# CLAUDE.md

Guidance for Claude Code in this repo. This is the **map**; per-area conventions live in `.agents/rules/*.mdc` (auto-loaded by file type; also reachable as `.cursor/rules/` and `.claude/rules/` via symlinks).

## What Pulse Is

Real-time mobile + web observability platform on OpenTelemetry. Mobile/web SDKs send OTLP signals → Collector → ClickHouse. A React dashboard lets teams drill into crashes, sessions, network, interactions, and web vitals.

## Monorepo Layout

| Directory | Service | Tech | Port |
|---|---|---|---|
| `backend/server/` | REST API | Java 17, Vert.x 4.5, Guice, Maven | 8080 |
| `backend/pulse-alerts-cron/` | Alert cron | Java/Vert.x | 4000 |
| `backend/ingestion/` | OTEL Collector configs + ClickHouse schema | YAML/SQL | — |
| `pulse-ui/` | Dashboard | React 18, TS, Mantine v7 | 3000 |
| `pulse_ai/` | AI agent | Python, Google ADK + Gemini | 8000 |
| `pulse-android-otel/` | Android SDK | Kotlin, OTel | — |
| `pulse-ios-otel/` | iOS SDK | Swift | — |
| `pulse-react-native-otel/` | RN SDK | TypeScript | — |
| `pulse-web-otel/` | Web SDK (alpha) | TypeScript, OTLP | — |
| `deploy/` | Docker Compose, scripts | — | — |

Auxiliary: `backend/db/` (MySQL migrations), `backend/spark/` (batch jobs), `backend/{heatmap-screenshot,session-capture,session-replay}-*/` (specialty ingestion), `pulse-mcp/`, `pulse-mobile-benchmarks/`, `vector/`, `docs/`, `scripts/`, `product/` (PRDs + web panel).

## Data Flow

```
Mobile/Web SDKs → OTEL Collector (4317/4318) → ClickHouse (otel DB)
Custom Events   → Vector (14317/14318) → S3 (Parquet) → Athena
```

## Build & Dev Commands

```bash
# Backend
cd backend/server && mvn verify        # build + checkstyle + JaCoCo
mvn -Dtest=MyClass#myMethod test       # single test

# Frontend
cd pulse-ui && yarn install && yarn start    # :3000
yarn build && yarn lint && yarn test
yarn test --testPathPattern=ComponentName

# Web SDK (pulse-web-otel)
cd pulse-web-otel && yarn install
cd pulse-web-otel && yarn build
cd pulse-web-otel && yarn test
cd pulse-web-otel && yarn workspace ecommerce-demo dev   # React demo :3002
cd pulse-web-otel && yarn demo:docs                      # vanilla demo :3003

# AI Agent
cd pulse_ai && ./setup.sh              # :8000

# Full stack
cd deploy && ./scripts/quickstart.sh   # build + start all
./scripts/start.sh -d                  # detached
./scripts/logs.sh [service]            # server/ui/ai/cron/otel-collector
./scripts/stop.sh [-v]                 # -v removes volumes
```

## Auth

- **Prod:** Google OAuth 2.0 → JWT (access 24h, refresh 30d).
- **Dev** (`GOOGLE_OAUTH_ENABLED=false`): mock users `mock-user-1` / `mock-user-2`, project `default-project`, key `default-project_devkey01`.

## ClickHouse Essentials

DB `otel`. Key tables: `otel_traces`, `otel_logs`, `otel_metrics_gauge`, `stack_trace_events`, `interaction_heatmaps_daily`.

Always use materialized columns over map access:

| Column | Source attribute |
|---|---|
| `ProjectId` | `ResourceAttributes['project.id']` |
| `PulseType` | `SpanAttributes` / `LogAttributes['pulse.type']` |
| `Platform` | `ResourceAttributes['os.name']` |
| `AppVersion` | Traces: `ResourceAttributes['app.version']`. Logs: `ResourceAttributes['app.build_name']`. `stack_trace_events`: use column `AppVersion`. |
| `SessionId` | `SpanAttributes` / `LogAttributes['session.id']` |

Every query must include: `Timestamp` range, `LIMIT`, `ProjectId` filter. Use tenant credentials — never admin. Multi-tenant isolation via row policies per project. Detail: `clickhouse-sql.mdc`.

## Architecture Pointers

Per-area conventions live in `.agents/rules/*.mdc` (the same files appear under `.cursor/rules/` and `.claude/rules/` via symlinks). Highlights only:

- **Backend (`backend/server/`, `backend/pulse-alerts-cron/`)** — Layer order: Controller (`resources/<domain>/`) → Service (`service/<domain>/`, RxJava3) → DAO (`dao/<domain>/` with `Queries.java`) → SQL. Guice DI, Lombok, MapStruct, `ServiceError`-coded errors. Primary CRUD store: **MySQL** (`vertx-mysql-client`); ClickHouse is OTEL/analytics only. Multi-tenancy: `tenant/TenantContext` + `TenantFilter` resolve `projectId`/user from JWT. Authorization: **OpenFGA** (`config/OpenFgaConfig.java`). **JaCoCo: 80% on changed files.** Detail: `java-backend.mdc`, `alerts-cron.mdc`, `java-testing.mdc`.
- **Frontend (`pulse-ui/`)** — Folder-per-screen/component (`Name.tsx` + `Name.module.css` + `index.ts`). State: TanStack Query (server) + Zustand (client). Routing config in `routes/routes.tsx`; URL constants in `constants/Constants.ts`. **API: always `makeRequest<T>()` from `helpers/makeRequest/`** — never raw fetch. Detail: `react-frontend.mdc`, `react-testing.mdc`.
- **Android SDK (`pulse-android-otel/`)** — Kotlin, Gradle multi-module. Two package roots that must never mix: `io.opentelemetry.android.*` (upstream) and `com.pulse.*` (Pulse). Detail: `android-sdk.mdc`.
- **iOS SDK (`pulse-ios-otel/`)** — Swift, SwiftPM/CocoaPods.
- **RN SDK (`pulse-react-native-otel/`)** — TS strict. Single `Pulse` facade in `index.tsx`. Detail: `react-native-sdk.mdc`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dream-horizon-org/pulse](https://github.com/dream-horizon-org/pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
