---
trigger: always_on
description: > Personal baking PWA. Self-hosted on the existing Proxmox + Coolify + Cloudflare Tunnel stack.
---

# Bakebook — CLAUDE.md

> Personal baking PWA. Self-hosted on the existing Proxmox + Coolify + Cloudflare Tunnel stack.
> Working name: **Bakebook**. Rename freely (`pnpm rename <new>` script provided in M0).

> **Read order:** this file → `DESIGN.md` → `phases/M{current}/task.md`. Do not skip ahead between phases.

---

## 1. What this is

A single-user baking companion that runs as an installable PWA on phone and desktop. It does five things, in priority order:

1. **Holds recipes** as structured data — ingredients with grams, ordered steps, optional per-step timers and target temperatures. Per-recipe macros, cost, full baker's ratios.
2. **Runs an active bake** — full-screen, large-tap, step-by-step mode with timers that survive backgrounding and a Wake Lock so the screen stays on.
3. **Captures structured outcomes** after each bake — rating, photos, measurements (rise, internal temp, weight loss), crumb/crust scores, free-text notes, and structured "tweaks" (e.g. *+10 g water, dough was tight*) that surface on the recipe next time.
4. **Versions recipes** — every meaningful change either edits in place (with audit trail) or forks. Lineage is visible. Tweaks one-click apply to a new version.
5. **Surfaces patterns** in an Insights screen once enough bakes have been logged — bakes per month, common tweaks, equipment success rates, calendar heatmap.

Non-goals (do not build in v1):
- Social / sharing / public profiles.
- Recipe scraping from URLs (manual entry only — the structure is the value).
- Meal planning, calendars, calorie tracking as a primary lens.
- Multi-user accounts. Single user, single household.
- Native iOS/Android apps. PWA only.
- Pantry inventory / restock alerts (deferred to v2).

---

## 2. Tech stack

Standard Nicolas-stack with no surprises. Every choice below is fixed; do not substitute without a written ADR in `/decisions/`.

| Layer | Choice | Notes |
|---|---|---|
| Frontend | Next.js 14 (App Router) + TypeScript + Tailwind + shadcn/ui | Mobile-first. Use `next-pwa` for service worker + manifest. |
| State | Zustand for client UI state, TanStack Query for server state | No Redux. |
| Backend | FastAPI (Python 3.12) | Pydantic v2 models. |
| DB | PostgreSQL 16 | One schema, no multi-tenancy. |
| ORM | SQLAlchemy 2.0 + Alembic | Async session. |
| Object storage | Cloudflare R2 | Photos. Signed URLs for read, presigned PUT for upload. |
| Auth | Cloudflare Access in front of the Tunnel | No app-level auth in v1. The Tunnel is the perimeter. App reads `Cf-Access-Authenticated-User-Email` header. |
| Deploy | Coolify on Proxmox | Two services: `web` (Next.js) and `api` (FastAPI). One Postgres. |
| Edge | Cloudflare Tunnel → `bakebook.nicolasschaerer.ch` | TLS terminated at Cloudflare. |
| Tests | Vitest (unit), Playwright (e2e + screenshots), Pytest (api) | Playwright is the source of truth for "feature done." |
| Package mgr | pnpm (web), uv (api) | |

**Deployment fallback noted, not chosen:** plain docker-compose on a Proxmox VM is viable if Coolify becomes annoying. If you switch, document it in `/decisions/0001-deployment.md` — the rest of the spec is Coolify-agnostic.

---

## 3. Architecture

```
┌─────────────────────┐
│   iPhone / Desktop  │  PWA, installable, offline reads
└──────────┬──────────┘
           │  HTTPS via bakebook.nicolasschaerer.ch
┌──────────▼──────────┐
│  Cloudflare Tunnel  │  + Cloudflare Access (email-restricted to me)
└──────────┬──────────┘
           │
┌──────────▼──────────┐    ┌──────────────────┐
│  Next.js (web)      │───▶│  FastAPI (api)   │
│  Coolify container  │    │  Coolify cont.   │
└─────────────────────┘    └────────┬─────────┘
                                    │
                          ┌─────────▼─────────┐    ┌───────────────┐
                          │  Postgres (cool.) │    │  Cloudflare R2│
                          └───────────────────┘    └───────────────┘
```

Two long-lived browser concerns:

- **Service worker** caches static shell + last-viewed recipes for offline reads. Network-first for `/api/*`, stale-while-revalidate for recipe GETs.
- **Active bake worker** (a dedicated Web Worker, not the SW) holds timer state in IndexedDB so a tab refresh or backgrounding doesn't lose timers.

Bundled at build time:
- `apps/api/data/nutrition.json` — ~80 baking ingredients with per-100g macros, derived once from USDA FoodData Central via a build script. No external API at runtime.

---

## 4. Data model

Postgres. All `id`s are `uuid v7` (time-ordered). All timestamps `timestamptz`. Soft-delete via `deleted_at` on user-facing tables.

### 4.1 Recipes (versioned)

```sql
create table recipes (
  id              uuid primary key,
  version_of_id   uuid references recipes(id),     -- null for v1
  parent_recipe_id uuid references recipes(id),    -- null unless this is a fork
  version_number  int not null default 1,
  title           text not null,
  slug            text not null,
  category        text not null,                   -- 'bread' | 'sweet' | 'quick' | 'pizza' | 'other'
  summary         text,
  yields          text,                            -- '1 loaf, ~900 g' (free text)
  servings        int not null default 1,          -- for per-serving math

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NlcoIas/bakebook](https://github.com/NlcoIas/bakebook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
