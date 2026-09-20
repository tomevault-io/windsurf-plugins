---
trigger: always_on
description: Agent entry point for WorldMonitor. Read this first, then follow links for depth.
---

# AGENTS.md

Agent entry point for WorldMonitor. Read this first, then follow links for depth.

## What This Project Is

Real-time global intelligence dashboard. TypeScript SPA (Vite + Preact) with 161 top-level TypeScript component files, 80+ Vercel Edge API endpoint entries, a Tauri desktop app with Node.js sidecar, and a Railway relay service. Aggregates geopolitics, military, finance, climate, cyber, maritime, and aviation data across 35 freshness-tracked source groups.

## Repository Map

```
.
├── src/                    # Browser SPA (TypeScript, class-based components)
│   ├── app/                # App orchestration (data-loader, refresh-scheduler, panel-layout)
│   ├── bootstrap/          # Startup/recovery (chunk reload, deferred Sentry, SW update)
│   ├── components/         # 161 top-level TypeScript component files
│   ├── config/             # Variant configs, panel/layer definitions, market symbols
│   ├── services/           # Business logic (197 service modules and domain directories)
│   ├── shared/             # Cross-cutting helpers (premium paths, registries, staleness)
│   ├── embed/              # Embeddable widget loader
│   ├── styles/             # Global CSS (layers, themes, panel styles)
│   ├── shims/              # Runtime shims (child-process for sidecar)
│   ├── data/               # Static JSON datasets (conservation, renewable, happiness)
│   ├── e2e/                # Map test harnesses (consumed by Playwright specs)
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
├── data/                   # Static data (telegram channels, OREF threat translations, gamma irradiators)
├── public/                 # Static assets served as-is (favicons, textures, .well-known, llms.txt)
├── scripts/                # Seed scripts, build helpers, data fetchers
├── src-tauri/              # Tauri desktop shell (Rust + Node.js sidecar)
│   └── sidecar/            # Node.js sidecar API server
├── consumer-prices-core/   # Consumer-price scrapers (Playwright, per-country baskets; Railway/Docker)
├── workers/                # Cloudflare Workers (edge CORS preflight for api.worldmonitor.app)
├── tests/                  # Unit/integration tests (node:test runner)
├── e2e/                    # Playwright E2E specs
├── pro-test/               # Standalone Pro QA app (separate package)
├── docs/                   # Mintlify documentation site
├── docker/                 # Docker build for Railway services
├── deploy/                 # Deployment configs (nginx)
└── blog-site/              # Static blog (built into public/blog/)
```

## How to Run

```bash
npm ci                   # Deterministic install (also runs blog-site postinstall)
npm run dev              # Start Vite dev server (full variant)
npm run dev:tech         # Start tech-only variant
npm run dev:energy       # Start energy-security variant
npm run typecheck        # tsc --noEmit (strict mode)
npm run typecheck:api    # Typecheck API layer separately
npm run test:data        # Run unit/integration tests
npm run test:sidecar     # Run sidecar + API handler tests
npm run test:e2e         # Run all Playwright E2E tests
make generate            # Regenerate proto stubs + per-service & unified OpenAPI specs (requires buf + sebuf v0.11.1 plugins)
npm run worktree:bootstrap          # Fresh worktree: link local env files + npm ci with tmp cache
npm run worktree:bootstrap:test-only # Fresh docs/test worktree: same, but npm ci --ignore-scripts
npm run worktree:env                # Link ignored local env files only
```

## Fresh Worktree Bootstrap

Worktrees usually start without ignored local state. When creating or entering one:

1. Start from `origin/main` or the requested base, not a dirty local branch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sheikhhassnain/Koala73worldmoniter](https://github.com/Sheikhhassnain/Koala73worldmoniter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
