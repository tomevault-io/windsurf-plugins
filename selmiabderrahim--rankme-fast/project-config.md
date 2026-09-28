---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# RankMeFast — Claude Code Guide

## 1. Project Overview

**RankMeFast** is a self-hosted, open-source (AGPL-3.0) SEO platform. Operators supply their own vendor keys; the app buys raw SEO signal from vendor APIs (DataForSEO including Lighthouse lab scans, Google Search Console/Analytics and optional PSI/CrUX, optionally Anthropic Claude for AI summaries) behind a provider interface, cache aggressively, and ship a plain-language report. The product is self-hostable — one `docker compose up` boots the whole stack. There is no "install on customer host" agent, no Docker socket access, no VPS orchestration; every external signal comes through a typed provider adapter.

Users authenticate against the app (Better Auth, email+password or Google OAuth), add a site, run an audit (crawl → rule engine → fix-now list + optional Claude summary), track rank and Lighthouse lab results over time (plus CrUX when the Google field-data provider is configured), connect Google Search Console for URL Inspection, research keywords, and inspect backlinks/competitors.

## 2. Stack

**Client** (`client/`)
- React 18 + ReactDOM 18 (SPA dashboard; SSR public docs — see §9). The self-hosted edition ships no marketing/landing site: `/` and each locale root redirect to `/login`.
- Redux Toolkit 2, react-redux 9 — feature slices under `features/<name>/store/`
- React Router 6 (`createBrowserRouter` for SPA; `createStaticHandler` + `renderToString` for SSR)
- Native `fetch` via `shared/api/client.ts` — no axios; always `credentials: 'include'`
- Vite 6, TypeScript 5 (strict)
- **Tailwind CSS v4** (CSS-first, `@tailwindcss/vite`; tokens in `client/src/styles/tailwind.css` — no `tailwind.config.js`)
- **shadcn/ui** (Radix, `new-york` style) primitives under `client/src/shared/ui/` — imported via `@shared/ui/*`
- **Better Auth** React client (`better-auth/react`, `useAuthSession()` — no Redux mirror of auth state)
- **i18n** via `i18next` + `react-i18next`, 7 locales (`en, ar, fr, de, es, ru, zh`) with `navigator.language` auto-detect and RTL for `ar`
- react-helmet-async for SSR meta / OG / hreflang / JSON-LD
- Vitest + React Testing Library + jsdom
- zod for form/API validation
- lucide-react icons (no emoji as icons)

**Server** (`server/`)
- Express 4 + TypeScript 5
- **Mongoose 8** — the default document store (users mirror, sites, audit runs, snapshots)
- **Drizzle ORM + PostgreSQL 17** — scoped exception for relational, ordered time-series data (rank history, keywords, backlinks, competitors, subscriptions, usage counters). Migrations under `server/drizzle/`, generated via `drizzle-kit generate` and auto-applied on boot.
- **Better Auth 1.6** with the Drizzle/PostgreSQL adapter — owns sessions, scrypt hashing, CSRF (Origin / trusted-origins), email verification, password reset, Google OAuth. Mounted at `/api/auth/*` BEFORE `express.json()`.
- **BullMQ 5 + Redis 7** — `audits` and `ranks` queues consumed by a dedicated `worker` deployable (`server/src/worker.ts` → `dist/worker.cjs`). Dead-letter queue on terminal failure.
- **Vendor provider registry** — `server/src/shared/providers/` — every vendor call goes through a typed capability interface (`audit`, `rank`, `keyword`, `backlink`, `competitor`, `pagespeed`, `gsc`, `summary`, `contentSource`, `contentMonitor`, `appData`). `PROVIDER_<capability>=fake` boots deterministic test/demo fixtures; production refuses any fake backend unless `ALLOW_FAKE_PROVIDERS=true` is explicitly set. Feature modules import the interface, never a vendor SDK (ESLint-restricted).
- **Resend** transactional email
- **Socket.IO 4** (present; not currently used by product routes)
- **pino / pino-http** structured logging
- zod for env validation (`src/config/env.ts`)
- AES-256-GCM envelope for at-rest secrets (`shared/crypto/`, `MASTER_ENCRYPTION_KEY`)
- Vitest + supertest for integration tests; `mongodb-memory-server` + PGlite (WASM Postgres) for in-process datastores
- Playwright + `@axe-core/playwright` E2E + accessibility smoke against the composed stack

**Ops**
- Docker Compose (`mongo`, `postgres`, `redis`, `api`, `worker`, `web`) — internal `rankme` network; every secret comes from the ignored root `.env`
- CI: `.github/workflows/ci.yml` in `node:20-bookworm` containers; gates typecheck → lint → skip-policy → vitest + 100% v8 coverage → `npm audit --audit-level=high` (zero high/critical, no allow-list) → `docker compose build` → Playwright smoke + axe

## 3. Repo Layout

```
rankme.fast/
├── client/
│   ├── server.js                 # Node SSR entrypoint — renders public docs, redirects / → /login
│   └── src/
│       ├── app/                  # store.ts, routes.tsx, providers
│       ├── entry-client.tsx      # SPA hydrate (dashboard)
│       ├── entry-server.tsx      # SSR renderToString (docs, portals, shares)
│       ├── features/             # auth, sites, ranks, report, keyword-research,
│       │                         # backlinks, competitors, google, billing,
│       │                         # docs, dashboard, admin, superadmin
│       │   └── <name>/
│       │       ├── components/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SelmiAbderrahim/rankme.fast](https://github.com/SelmiAbderrahim/rankme.fast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
