---
trigger: always_on
description: Single-owner 0DTE SPX options trading tool. Vite + React 19 frontend, Vercel Serverless Functions backend (TypeScript), Python ML scripts, Railway Python sidecar for Databento futures + ES options ingestion.
---

# 0DTE SPX Strike Calculator

Single-owner 0DTE SPX options trading tool. Vite + React 19 frontend, Vercel Serverless Functions backend (TypeScript), Python ML scripts, Railway Python sidecar for Databento futures + ES options ingestion.

## Architecture

```text
src/              React 19 SPA (Tailwind CSS 4, no router)
  components/     UI components (50+ TSX files, feature-grouped folders)
  hooks/          Custom React hooks (useAppState, useMarketData, useChainData, etc.)
  utils/          Pure calculation modules (black-scholes, strikes, hedge, iron-condor, pin-risk, etc.)
  types/          Shared TypeScript types
  data/           Static data (market hours, VIX stats — VIX OHLC has a cutoff date)
  constants/      App-wide constants

api/              Vercel Serverless Functions
  _lib/           21+ shared modules (see "Backend Modules" below)
  auth/           Schwab OAuth flow (init.ts, callback.ts)
  cron/           35 scheduled jobs (market data fetching, feature building, lesson curation)
  journal/        Journal CRUD + DB init/migrate
  ml/             ML data export endpoint

sidecar/          Databento futures data ingestion (Python, Railway, NOT Vercel)
  src/            Python 3 service using databento SDK + psycopg2
                  Ingests 7 futures symbols (ES, NQ, ZN, RTY, CL, GC, DX) + ES options
                  Own requirements.txt, pyproject.toml, Dockerfile
                  Uses psycopg2 (not @neondatabase/serverless) for Neon Postgres
                  Sentry SDK for error tracking; VX deferred pending Databento availability
                  vercel.json ignoreCommand skips deploys for sidecar/, ml/, scripts/, pine/, docs/, *.md changes

ml-sweep/         PAC backtest runner (Python, Railway, NOT Vercel — sibling to sidecar)
                  FastAPI + bearer auth gating /run, /status/{id}, /logs/{id}, /hydrate, /hydrate/status
                  Spawns whitelisted sweep scripts (pac_backtest CPCV + Optuna) as subprocesses
                  5 GB volume at /data hydrated from Vercel Blob archive (parquet per year)
                  Heartbeat (30s) + orphan recovery on container restart; uploads JSON to Blob
                  Own Dockerfile, README.md, TEARDOWN.md — auto-deploys on ml-sweep/** or ml/** pushes
                  Do NOT enable scale-to-zero (kills sweeps mid-flight — HTTP idle counter is
                  blind to subprocess CPU)

scripts/          Backfill scripts (backfill-etf-tide.mjs, backfill-greek-exposure.mjs, etc.)

ml/               Python ML pipeline (clustering, EDA, classification, visualization)
  src/            Source modules (utils, clustering, eda, phase2_early, pin_analysis, etc.)
  tests/          Pytest test files (test_clustering, test_phase2, etc.)
  docs/           Phase specs and design docs (ROADMAP.md, PHASE-*.md)
  plots/          Generated plots — tracked in git, do NOT gitignore
  experiments/    JSON experiment results (phase2_early runs)
  .venv/          Python venv — run scripts with `ml/.venv/bin/python`, not system python3
  conftest.py     Adds ml/src/ to sys.path for test imports

docs/             Design artifacts
  superpowers/    specs/ and plans/ for feature design documents

e2e/              Playwright specs (23 specs including a11y)
```

## Commands

```bash
npm run dev          # Vite dev server (frontend only)
npm run dev:full     # Full stack via vercel dev with pino-pretty
npm run build        # tsc + vite build
npm run lint         # tsc --noEmit && eslint (MUST run after code changes)
npm run test         # vitest watch mode
npm run test:run     # vitest single run
npm run test:e2e     # playwright
npm run format       # prettier --write
```

## Development Workflow (Get It Right)

Every code change follows this implement-verify-review loop. No exceptions. This applies to the main session and all subagents that write code.

### Plan First (Large Changes)

For any change that spans **3+ files, introduces a new feature end-to-end, or was scoped across multiple conversation turns**, write a plan doc to `docs/superpowers/specs/` BEFORE starting the Get It Right loop. Context compaction can silently drop the scoping conversation — the plan doc is the durable handoff to the next session (or this session post-compaction).

The plan must include:

- **Goal** — one sentence on what this feature does and why
- **Phases** — numbered, each independently shippable, with rough scope estimates
- **Files to create/modify** — concrete list, grouped by phase
- **Data dependencies** — new tables, migrations, env vars, external APIs
- **Open questions** — anything undecided, with default picks noted
- **Thresholds / constants** — any magic numbers agreed on during scoping

Skip the plan doc only for:

- Bug fixes within a single file
- Refactors contained to one module
- Config-only changes (`.json`, `.md`, ESLint/Prettier tweaks)

When in doubt, write the plan. A plan doc is ~10 minutes; rediscovering scope is much more.

### The Loop

**1. Implement** — Write the code. Investigate first, understand existing patterns, then make changes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cobriensr/Options-Strike-Calculator](https://github.com/cobriensr/Options-Strike-Calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
