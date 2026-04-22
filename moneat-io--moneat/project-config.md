---
trigger: always_on
description: Moneat is a Sentry-compatible, OpenTelemetry-compatible observability platform built with Kotlin/Ktor backend, React frontend, PostgreSQL for operational data, and ClickHouse for high-performance event analytics.
---

# Copilot Instructions for Moneat

Moneat is a Sentry-compatible, OpenTelemetry-compatible observability platform built with Kotlin/Ktor backend, React frontend, PostgreSQL for operational data, and ClickHouse for high-performance event analytics.

## Build, Test, and Lint

### Backend (Kotlin/Ktor)
```bash
cd backend
./gradlew build          # Build and test
./gradlew test           # Run tests only
./gradlew run            # Run locally (uses port 8080)
./gradlew shadowJar      # Build fat JAR for production
./gradlew seedE2EData    # Seed E2E test data
```

**Run single test class:**
```bash
./gradlew test --tests com.moneat.services.EventServiceTest
```

**Run single test method:**
```bash
./gradlew test --tests com.moneat.services.EventServiceTest.testEventIngestion
```

### Dashboard (React/Vite)
```bash
cd dashboard
npm run dev              # Dev server (port 5173)
npm run build            # Production build
npm run lint             # ESLint
npm run preview          # Preview production build
```

### Documentation (Docusaurus)
```bash
cd docs
npm install
npm run start            # Dev server (port 3000)
npm run build            # Production build (output in docs/build/)
npm run serve            # Serve built docs locally
```

The docs are built alongside the dashboard in the Docker image and served at `/docs/` by nginx.

### Email Templates (Maizzle)
```bash
cd emails
npm run dev              # Preview with live reload (port 3001)
npm run build:production # Build for production
```

Built templates are in `emails/build/templates/email/` with `{{ variable }}` placeholders for Kotlin template engine.

### Infrastructure

**IMPORTANT:** 
- **Database services** (PostgreSQL, ClickHouse, Redis) run via Docker (see `docker-compose.yml`)
- **Backend** (Kotlin/Ktor) runs **locally** on port 8080 and connects to the database services
- **Frontend** (React/Vite) runs **locally** on port 5173

Start infrastructure with `docker-compose up -d`, then run the backend and frontend locally.

Backend connects to services at (configurable via `.env`):
- PostgreSQL: `localhost:5499`
- ClickHouse: `localhost:8123` (HTTP), `localhost:9000` (native)
- Redis: `localhost:6379`

## Architecture Overview

### Request Flow
1. **Sentry ingestion**: Client → `/api/{projectId}/envelope/` → `IngestRoutes.kt` → `EventService.kt` → PostgreSQL + ClickHouse
2. **OTLP ingestion**: Client → `/v1/{logs,traces,metrics}/otlp` → `OtlpTraceRoutes.kt` / `OtlpMetricsRoutes.kt` / `LogRoutes.kt` → ClickHouse (authenticated via `Authorization: Bearer <OTLP API key>`)
3. **Dashboard API**: React → `/v1/*` → `ApiRoutes.kt` → `DashboardService.kt` → PostgreSQL/ClickHouse
4. **Authentication**: All `/v1/*` endpoints require JWT (except `/auth/*` and OTLP ingestion endpoints which use OTLP API keys)

### Data Storage Strategy
- **PostgreSQL**: Users, organizations, projects, project_keys, subscriptions (relational data)
- **ClickHouse**: Events, issues (materialized view), sessions, spans, logs (high-volume time-series)
- **Redis**: Caching, rate limiting, background job queues

### Backend Package Structure
```
com.moneat/
├── Application.kt           # Entry point, configures plugins
├── config/                  # Environment, Sentry, ClickHouse, Redis clients
├── plugins/                 # Ktor plugins: Security, HTTP, Routing, Databases, etc.
├── routes/                  # HTTP endpoints: IngestRoutes, ApiRoutes, AuthRoutes
├── services/                # Business logic: EventService, DashboardService, AuthService
├── models/                  # Data classes: SentryModels, ApiModels, database tables
├── otlp/                    # OpenTelemetry (OTLP) ingestion: routes, services, auth, parsing
│   ├── routes/              # OtlpTraceRoutes, OtlpMetricsRoutes
│   ├── services/            # OtlpTraceService, OtlpMetricsService, OtlpApiKeyService
│   └── models/              # OtlpApiKeyModels
├── utils/                   # Shared utilities
└── logging/                 # Custom logging configuration
```

### Frontend Structure
```
dashboard/src/
├── routes/                  # TanStack Router file-based routes
├── components/              # Reusable UI components (shadcn/ui)
├── lib/api.ts               # API client with JWT auth
├── hooks/                   # React hooks
└── contexts/                # React contexts (auth, etc.)
```

### Dashboard: TypeScript & ESLint (Sonar-friendly)
To avoid Sonar/ESLint code smells in `dashboard/`:

**Exports & globals:**
- ❌ **Don't export mutable `let`**: `export let x = null`
- ✅ **Use `const` with object/ref**: `export const x = { current: null }` for test hooks, etc.
- ✅ **Prefer `globalThis.window`** over `window` (SSR-safe)
- ✅ **Prefer `globalThis.sessionStorage`** / `globalThis.localStorage` over bare globals

**Assignments & operators:**
- ✅ **Use nullish coalescing assignment** when assigning only if null: `x ??= value` instead of `if (!x) x = value`

**Strings & DOM:**
- ❌ **Avoid nested template literals**: `` `${base}/path${qs ? `?${qs}` : ''}` ``

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moneat-io/moneat](https://github.com/moneat-io/moneat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
