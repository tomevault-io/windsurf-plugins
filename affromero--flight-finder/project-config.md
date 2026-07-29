---
trigger: always_on
description: > **Flight Finder** — The price trail airlines don't show you. Flight price evolution tracker with natural language search and shareable charts.
---

# CLAUDE.md — Flight Finder

> **Flight Finder** — The price trail airlines don't show you. Flight price evolution tracker with natural language search and shareable charts.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 15+ (App Router), TypeScript, CSS Modules |
| Database | PostgreSQL 16 + Prisma ORM |
| Cache | Redis 7 (rate limiting + response caching) |
| AI | Anthropic Claude, OpenAI GPT, Google Gemini, Claude Code CLI, Ollama, llama.cpp, vLLM |
| Browser | Playwright (headless Chromium for Google Flights scraping) |
| Charts | Plotly.js (interactive price evolution) |
| Hosting | Hetzner VPS (Docker Compose + Caddy) — flight-finder.org |
| CI/CD | GitHub Actions (CI + Deploy on push to main) |

## Monorepo

npm workspaces: `@flight-finder/web` (`apps/web/`), `@flight-finder/cli` (`packages/cli/`).
Root `package.json` proxies common scripts to `@flight-finder/web`. `apps/desktop/` is a Tauri (Rust) launcher: deliberately NOT an npm workspace member and excluded from `npm run ci`; it is built only by `.github/workflows/desktop-release.yml`.

The CLI bundles the shared scraper (`apps/web/src/lib/scraper/*`) via relative imports but maps `@/*` to its own `packages/cli/src/`, so any new `@/lib/<x>` import added to a shared scraper file needs a matching shim in `packages/cli/src/lib/` (re-export the real module like `secret-crypto.ts`/`prisma.ts`, or a stub like `admin-recovery.ts`). Web-only checks pass without it; only the full `npm run ci` (web + cli) catches a missing shim.

**Versioning (locked):** `apps/web`, the root `package.json`, `packages/cli`, and `apps/desktop` all carry the SAME version number. `apps/web/package.json` is the source of truth; git tags `vX.Y.Z` track the web release and `desktop-vX.Y.Z` tags the desktop build at the same number (distinct prefixes, no collision). `/create-release` must bump all of them to the new version — `apps/web/package.json`, root `package.json`, `packages/cli/package.json`, and `apps/desktop` (its `package.json`, `src-tauri/Cargo.toml`, and `src-tauri/tauri.conf.json`) — and regenerate both lockfiles (`package-lock.json` and `apps/desktop/src-tauri/Cargo.lock`).

## Environment Variables

All secrets via **Doppler** — NEVER use `.env` files. Project: `flight-finder`, config: `dev`.
Scripts wrap with `doppler run --`. Shared LLM keys from `pricetoken` Doppler project.

Critical: `DATABASE_URL`, `REDIS_URL`, `ANTHROPIC_API_KEY`, `ADMIN_PASSWORD`, `ADMIN_SESSION_SECRET`, `CRON_SECRET`.

## Build Commands

```bash
npm install                    # All workspaces
docker compose -f docker-compose.prod.yml up -d db redis
npx prisma db push --schema=apps/web/prisma/schema.prisma
npx prisma generate --schema=apps/web/prisma/schema.prisma
npm run dev                    # Web app on :3003 (next dev --port 3003, no doppler wrapper at workspace level)
npm run ci                     # lint + typecheck + test + build (both web and cli workspaces)
```

## File Index

### `apps/web/src/app/` — Pages & API routes

| Path | Purpose |
|------|---------|
| `page.tsx` | Landing page — natural language search bar |
| `layout.tsx` | Root layout — fonts, metadata |
| `q/[id]/page.tsx` | Public shareable chart page (no auth) |
| `admin/(auth)/login/page.tsx` | Admin login (legacy, redirects to /login in multi user mode) |
| `admin/(dashboard)/page.tsx` | Admin dashboard — active queries, costs |
| `admin/(dashboard)/queries/page.tsx` | Query management — pause/resume/delete/reassign |
| `admin/(dashboard)/config/page.tsx` | LLM agent config — provider/model selection |
| `admin/(dashboard)/notifications/page.tsx` | Notification channels + new-low alert thresholds |
| `admin/(dashboard)/users/page.tsx` | User management (multi user mode only) — create/reset/delete |
| `login/page.tsx` | Unified login (multi user mode only) — admin + non admin |
| `account/page.tsx` | Logged in user's tracker list (multi user mode only) |
| `account/settings/page.tsx` | Per user preferences — currency, country, airlines, cabin |
| `api/parse/route.ts` | POST — LLM parses natural language flight query |
| `api/queries/route.ts` | POST — create new tracked query (401 anon in multi user mode) |
| `api/queries/[id]/route.ts` | PATCH/DELETE — update or delete a query by deleteToken or user session |
| `api/queries/[id]/prices/route.ts` | GET — public price data for chart |
| `api/queries/[id]/scrape/route.ts` | POST — force-scrape a single query immediately (deleteToken or session auth) |
| `api/queries/active/route.ts` | GET — active non-seed queries for the current user (scoped in multi user mode) |
| `api/queries/status/route.ts` | POST — batch-check active/expired status for up to 20 query IDs |
| `api/cron/scrape/route.ts` | GET — trigger scrape run (CRON_SECRET auth) |
| `api/auth/login/route.ts` | POST — user login (multi user mode only); rate limited |
| `api/auth/logout/route.ts` | POST — clears the shared ft-session cookie |
| `api/auth/me/route.ts` | GET — current user; 401/404 outside multi user mode |
| `api/admin/auth/route.ts` | POST — legacy admin login; 410 in multi user mode |
| `api/admin/auth/logout/route.ts` | POST — admin logout |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [affromero/flight-finder](https://github.com/affromero/flight-finder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
