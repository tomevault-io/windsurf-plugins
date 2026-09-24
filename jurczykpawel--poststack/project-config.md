---
trigger: always_on
description: **PostStack** — self-hosted, source-available multi-channel social media management platform
---

# AGENTS.md - PostStack

## Project Overview

**PostStack** — self-hosted, source-available multi-channel social media management platform
(Facebook, Instagram, YouTube, Telegram + Gmail reply/inbox; extensible via the provider pattern):
publishing & scheduling + inbox auto-replies + drip sequences + CRM.
**Elastic License 2.0** (source-available, NOT open-source/AGPL — see LICENSE + CLA.md).
One-command startup via Docker Compose.

Alternative to ManyChat / Buffer / Hootsuite, without vendor lock-in.

## Task Tracking (private — never commit)

All planned work lives as **one task per file** under `priv/tasks/*.md`. This directory is **gitignored** — it must never land in the public history.

**Always use this system. Do not invent ad-hoc TODO lists.**

- **Read `priv/tasks/INDEX.md` first** — it's the board (every task with status + priority).
- **One task = one file.** Never lump multiple tasks into a single file. New task → new `priv/tasks/<ID>-<slug>.md` with frontmatter: `id, title, status, priority, area, epic, depends_on, created, tags`.
- **Status:** `queued` | `active` | `parked` | `done`. **Priority:** 1–100 (higher = pilniejsze).
- Starting work → set `status: active`. Finishing → set `status: done` and update `INDEX.md`.
- Found a stray TODO/note (incl. `TODO.md`)? Migrate it into a `priv/tasks/` file; don't leave parallel backlogs.

## Stack

| Layer | Tech |
|-------|------|
| Framework | Hono (web server + API), `hono/html` SSR |
| UI | Server-rendered HTML + htmx + Alpine.js (no client framework) |
| Language | TypeScript 5 |
| Database | PostgreSQL + Drizzle ORM |
| Queue | PostgreSQL (graphile-worker) |
| Styling | Plain CSS (CSS variables, dark theme) — no UI framework |
| Platforms | Facebook, Instagram (extensible via provider pattern) |
| Auth | Custom JWT (jose) |
| Encryption | AES-256-GCM (Node.js crypto) |
| Runtime | Bun (web + worker); Vitest on Node for tests |
| Infra | Docker Compose |

## Architecture

```
Web process (Hono, on Bun):
  - src/server/app.ts             → Hono app: security headers, CORS, routing
  - /api/webhooks/meta            → enqueue jobs
  - /api/oauth/facebook|instagram|instagram-login → OAuth callbacks (instagram-login = Instagram Business Login)
  - /api/cron/token-refresh       → token refresh trigger
  - /api/v1/*                     → REST API (handlers in src/server/handlers/v1, delegated)
  - /inbox, /channels, ...        → server-rendered dashboard (htmx + Alpine)

Worker process (graphile-worker, on Bun):
  - incoming-messages worker  → contact upsert → rule engine → enqueue reply
  - outgoing-messages worker  → Meta Graph API send
  - token-refresh worker      → refresh expiring OAuth tokens
  - sequence-steps worker     → deliver drip sequence messages
```

> **Instagram Business Login routing:** when a channel has no Facebook page token (an IG-Login-only
> account), Instagram messaging/comments/publish calls route to `graph.instagram.com` using the
> IG-Login (IGQW) token instead of `graph.facebook.com` with a page token.

## Landing / marketing site (READ THIS before "deploying the landing")

The `landing/` Astro site is **NOT a separate deployment** and does **NOT** live on a separate
domain or Cloudflare Pages. It is **built into the app Docker image** and served by the app itself:

- `docker/Dockerfile` has a `landing` build stage (`npm run build` in `landing/`) and copies
  `landing/dist` into the runtime image.
- `src/server/routes/landing.ts` (`serveLandingFile`) serves it at `/` and `/privacy` (+ `/_astro/*`).
  Logged-out visitors see the marketing site; logged-in visitors are redirected to `/overview`.

**So the landing and the app are one deployment, on one domain (`poststack.techskills.academy`).**
To ship landing changes, **cut a normal app release** (bump version → tag `v*` → `release.yml` builds the
image incl. the fresh `landing/dist` → auto-deploy TEST → manual PROD). There is no CF Pages step and no
DNS cutover. (The `app.poststack.techskills.academy` split mentioned in some marketing copy was an
abandoned plan — a cert issue on the subdomain — so the app currently lives on the apex; treat any
"app on a subdomain / deploys independently" wording as stale.)

## API-First Design

PostStack is API-first for operational data and automation at `/api/v1/*`. Interactive channel
connection and reconnection remain logged-in dashboard flows at `/api/oauth/*`.

- **Dual auth:** session JWT cookie (dashboard) + `Authorization: Bearer sk_live_<key>` (external, prefix from `BRAND.idPrefix`)
- **CORS:** enabled on all `/api/v1/*` routes
- **OpenAPI spec:** `src/lib/api/openapi.ts` — update when adding new routes
- **Scalar UI:** `/api/docs` (no npm dep, CDN-loaded)
- **Response shape:** always `{ data, error, meta? }` — use helpers from `src/lib/api/response.ts`
- **REST auth:** use `authenticateWithScope(request, scope)` for scoped `/api/v1/*` operations
- **Interactive OAuth auth:** starts use `authenticateSession(request)` and callbacks use the shared
  `authenticateOAuthCallback(request, state, flow)` boundary; API keys must not start or finish these flows

## Key Directories

```
src/
├── server/               # Hono app

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jurczykpawel/poststack](https://github.com/jurczykpawel/poststack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
