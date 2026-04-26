---
trigger: always_on
description: Agent entry point for WorldMonitor. Read this first, then follow links for depth.
---

# AGENTS.md

Agent entry point for WorldMonitor. Read this first, then follow links for depth.

## What This Project Is

Real-time global intelligence dashboard. TypeScript SPA (Vite + Preact) with 86 panel components, 60+ Vercel Edge API endpoints, a Tauri desktop app with Node.js sidecar, and a Railway relay service. Aggregates 30+ external data sources (geopolitics, military, finance, climate, cyber, maritime, aviation).

## Repository Map

```
.
├── src/                    # Browser SPA (TypeScript, class-based components)
│   ├── app/                # App orchestration (data-loader, refresh-scheduler, panel-layout)
│   ├── components/         # 86 UI panels + map components (Panel subclasses)
│   ├── config/             # Variant configs, panel/layer definitions, market symbols
│   ├── services/           # Business logic (120+ service files, organized by domain)
│   ├── types/              # TypeScript type definitions
│   ├── utils/              # Shared utilities (circuit-breaker, theme, URL state, DOM)
│   ├── workers/            # Web Workers (analysis, ML/ONNX, vector DB)
│   ├── generated/          # Proto-generated client/server stubs (DO NOT EDIT)
│   ├── locales/            # i18n translation files
│   └── App.ts              # Main application entry
├── api/                    # Vercel Edge Functions (plain JS, self-contained)
│   ├── _*.js               # Shared helpers (CORS, rate-limit, API key, relay)
│   ├── health.js           # Health check endpoint
│   ├── bootstrap.js        # Bulk data hydration endpoint
│   └── <domain>/           # Domain-specific endpoints (aviation/, climate/, etc.)
├── server/                 # Server-side shared code (used by Edge Functions)
│   ├── _shared/            # Redis, rate-limit, LLM, caching, response headers
│   ├── gateway.ts          # Domain gateway factory (CORS, auth, cache tiers)
│   ├── router.ts           # Route matching
│   └── worldmonitor/       # Domain handlers (mirrors proto service structure)
├── proto/                  # Protobuf definitions (sebuf framework)
│   ├── buf.yaml            # Buf configuration
│   └── worldmonitor/       # Service definitions with HTTP annotations
├── shared/                 # Cross-platform data (JSON configs for markets, RSS domains)
├── scripts/                # Seed scripts, build helpers, data fetchers
├── src-tauri/              # Tauri desktop shell (Rust + Node.js sidecar)
│   └── sidecar/            # Node.js sidecar API server
├── tests/                  # Unit/integration tests (node:test runner)
├── e2e/                    # Playwright E2E specs
├── docs/                   # Mintlify documentation site
├── docker/                 # Docker build for Railway services
├── deploy/                 # Deployment configs
└── blog-site/              # Static blog (built into public/blog/)
```

## How to Run

```bash
npm install              # Install deps (also runs blog-site postinstall)
npm run dev              # Start Vite dev server (full variant)
npm run dev:tech         # Start tech-only variant
npm run typecheck        # tsc --noEmit (strict mode)
npm run typecheck:api    # Typecheck API layer separately
npm run test:data        # Run unit/integration tests
npm run test:sidecar     # Run sidecar + API handler tests
npm run test:e2e         # Run all Playwright E2E tests
make generate            # Regenerate proto stubs (requires buf + sebuf plugins)
```

## Architecture Rules

### Dependency Direction

```
types -> config -> services -> components -> app -> App.ts
```

- `types/` has zero internal imports
- `config/` imports only from `types/`
- `services/` imports from `types/` and `config/`
- `components/` imports from all above
- `app/` orchestrates components and services

### API Layer Constraints

- `api/*.js` are Vercel Edge Functions: **self-contained JS only**
- They CANNOT import from `../src/` or `../server/` (different runtime)
- Only same-directory `_*.js` helpers and npm packages
- Enforced by `tests/edge-functions.test.mjs` and pre-push hook esbuild check

### Server Layer

- `server/` code is bundled INTO Edge Functions at deploy time via gateway
- `server/_shared/` contains Redis client, rate limiting, LLM helpers
- `server/worldmonitor/<domain>/` has RPC handlers matching proto services
- All handlers use `cachedFetchJson()` for Redis caching with stampede protection

### Proto Contract Flow

```
proto/ definitions -> buf generate -> src/generated/{client,server}/ -> handlers wire up
```

- GET fields need `(sebuf.http.query)` annotation
- `repeated string` fields need `parseStringArray()` in handler
- `int64` maps to `string` in TypeScript
- CI checks proto freshness via `.github/workflows/proto-check.yml`

## Variant System

The app ships multiple variants with different panel/layer configurations:

- `full` (default): All features
- `tech`: Technology-focused subset
- `finance`: Financial markets focus
- `commodity`: Commodity markets focus
- `happy`: Positive news only

Variant is set via `VITE_VARIANT` env var. Config lives in `src/config/variants/`.

## Key Patterns

### Adding a New API Endpoint

1. Define proto message in `proto/worldmonitor/<domain>/`
2. Add RPC with `(sebuf.http.config)` annotation
3. Run `make generate`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GeneralJerel/world-monitor-copilotkit](https://github.com/GeneralJerel/world-monitor-copilotkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
