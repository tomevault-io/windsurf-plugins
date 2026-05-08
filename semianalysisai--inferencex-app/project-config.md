---
trigger: always_on
description: For detailed subsystem docs, see [docs/index.md](./docs/index.md).
---

# AGENTS.md

For detailed subsystem docs, see [docs/index.md](./docs/index.md).

## Project Overview

InferenceX App — Next.js 16 dashboard for ML inference benchmark data. DB-backed with Neon PostgreSQL, React Query for data fetching, D3.js for charts.

- **Framework**: Next.js 16 (App Router, Turbopack)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS 4 + shadcn/ui (Radix UI primitives)
- **Charts**: D3.js — shared library at `src/lib/d3-chart/`, scatter/GPU/bar charts
- **Data**: Neon DB → API routes (`/api/v1/*`) → React Query hooks → Context providers
- **Deployment**: Vercel with daily cron-triggered rebuilds
- **Analytics**: PostHog (`posthog-js`) via `@/lib/analytics` — recommended on all interactive elements (autocapture provides baseline coverage)

## Quick Start

```bash
pnpm install              # Install dependencies
pnpm dev                  # Dev server with Turbopack (http://localhost:3000)
pnpm build                # Production build
pnpm typecheck            # TypeScript type checking (all packages)
pnpm lint                 # Lint with oxlint
pnpm lint:fix             # Auto-fix lint issues
pnpm fmt                  # Format check with oxfmt
pnpm fmt:fix              # Auto-fix formatting
pnpm test:unit            # Vitest unit tests
pnpm test:e2e             # Cypress E2E tests
```

## Monorepo Structure

```
packages/
├── app/                  # Next.js frontend (@semianalysisai/inferencex-app)
│   ├── content/blog/     # MDX blog posts (frontmatter + content)
│   └── src/
│       ├── app/          # Pages, layouts, API routes (/api/v1/*)
│       │   └── blog/     # Blog list + [slug] post pages, OG image generation
│       ├── components/   # Tab sections: inference/, evaluation/, historical-trends/,
│       │                 #   throughput-calculator/, reliability/, gpu-specs/, blog/, ui/
│       ├── hooks/api/    # React Query hooks (use-benchmarks, use-availability, etc.)
│       └── lib/          # Utilities, constants, d3-chart/, chart-utils, blog, data-mappings
├── constants/            # Shared constants (GPU keys, model mappings, SEO)
└── db/                   # DB layer, ETL, migrations, queries, ingest scripts
```

**Path alias**: `@/*` → `packages/app/src/`

## Data Architecture

```
Frontend → React Query hooks (src/hooks/api/) → /api/v1/* routes → Neon DB
```

API routes (`packages/app/src/app/api/v1/`):

- `benchmarks?model=X&date=YYYY-MM-DD` — latest benchmark per (config, concurrency)
- `benchmarks/history?model=X&gpu=Y` — historical benchmark data for trend charts
- `workflow-info?date=YYYY-MM-DD` — runs, changelogs, configs for a date
- `availability` — `Record<model, dates[]>`
- `reliability` — raw `ReliabilityRow[]`
- `evaluations` — raw `EvalRow[]`
- `server-log` — retrieve benchmark runtime logs
- `invalidate` — invalidate API cache (admin)

**API routes return raw DB data** — no presentation logic. Frontend handles all transformations.

Static content routes (no DB):

- `/blog` — blog listing (statically generated from MDX files in `content/blog/`)
- `/blog/[slug]` — blog post page with MDX rendering and OG image generation
- `/feed.xml` — RSS 2.0 feed
- `/llms.txt` — LLM-readable site index
- `/llms-full.txt` — full article content for LLM ingestion
- `/sitemap.xml` — dynamic sitemap (includes blog posts)

## Code Style & Tooling

- **Linter**: oxlint — `pnpm lint` / `pnpm lint:fix`
- **Formatter**: oxfmt — `pnpm fmt` / `pnpm fmt:fix`
- **Type checking**: `pnpm typecheck` (tsc --noEmit, strict mode)
- **Node**: 24.x

## Environment Variables

See `.env.example`. Key vars: `GITHUB_TOKEN`, `DATABASE_READONLY_URL`, `DATABASE_WRITE_URL` (admin only).

## Testing

See [Testing](./docs/testing.md) for full requirements, quality standards, and pre-commit checklist. Tests are **mandatory** — missing/low-quality tests are 🔴 BLOCKING on PR review.

## Analytics Requirement

All interactive elements should have `track()` from `@/lib/analytics` (autocapture provides baseline coverage).

**Convention**: `[section]_[action]` — e.g., `latency_zoom_reset`, `calculator_bar_selected`, `tab_changed`

**Prefixes**: `latency_`, `interactivity_`, `gpu_timeseries_`, `inference_`, `calculator_`, `evaluation_`, `reliability_`, `tab_`, `selector_`, `blog_`, `social_`

## Tab Structure

Order: `inference` → `evaluation` → `historical` → `calculator` → `reliability` → `gpu-specs` (defined in `page-content.tsx` `VALID_TABS`). Tab value = URL hash.

## Unofficial Run Support — Mandatory for Inference / Evaluation Features

Any new feature that operates on inference or evaluation chart data **must** also work for unofficial run overlays — not just the official run rendering path. The overlay path is a separate code branch (`overlayData`, `processedOverlayData`, `overlayRooflines`, `activeOverlayHwTypes`, `overlayRunColor`/`overlayRunIndex` from `@/lib/overlay-run-style`, `useUnofficialRun()` from `@/components/unofficial-run-provider`) that is easy to forget — features that only handle the official path silently degrade for users who load an unofficial run via `?unofficialrun=…`.

When adding a chart feature (toggle, label, overlay, filter, export, share-link param, tooltip enrichment, …):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SemiAnalysisAI/InferenceX-app](https://github.com/SemiAnalysisAI/InferenceX-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
