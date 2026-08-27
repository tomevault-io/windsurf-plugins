---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**ArguMesh(论脉)** — open-source, local-first research workbench: `Literature → Evidence → Idea`. This is the self-contained OSS edition: **pure Node + local SQLite, zero cloud dependencies**. (The original prototype at `../prototype` was Cloudflare Workers + Turso; this repo intentionally removed that stack — do not reintroduce Cloudflare bindings, wrangler, or Turso-account requirements.)

- Frontend: React 19 SPA (Vite 6) — projects, library, PDF reader, evidence matrix, knowledge, ideas, tasks, search, settings (single-user; no admin/auth UI).
- Backend: Hono 4 app served by `@hono/node-server` (`server/node.ts`), SQLite via libSQL `file:` URL + Drizzle ORM.
- Auth: **none — single-user local workbench.** No login, no accounts, no tokens; the API is open on localhost. (Migration 0008 removed the `accounts` table, `owner_id` columns, and `/api/login` + `/api/users` routes; the live DB was migrated in place.)
- AI: optional, any OpenAI-compatible provider. **Primary: a single global config via the settings page form** (Base URL default `https://api.openai.com/v1`, API Key, model name — stored in `ai_settings` as the single `account_id='local'` row, `GET/PUT/DELETE /api/ai/config`, key never returned, only masked). Global config fully overrides the env fallback (`AI_PROVIDERS` JSON env / `STEPFUN_*`) and ignores client-sent provider/model. Unconfigured → AI endpoints return 400 `AI_NOT_CONFIGURED` pointing at the settings page; nothing else breaks.

## Commands

All commands run from this directory (`ArguMesh/`). Use **pnpm**.

```bash
pnpm install          # deps (use --reporter=append in noisy environments)
pnpm run dev          # API on 127.0.0.1:8787 (tsx watch) + Vite on :5173 (proxies /api)
pnpm run build        # tsc --noEmit + vite build → dist/
pnpm start            # single port 8787: serves dist/ static + API (build first)
pnpm run typecheck    # tsc --noEmit
pnpm run test         # Vitest: tests/unit (happy-dom) + tests/api (node + temp SQLite)
pnpm run test:watch
pnpm exec vitest run tests/api/<file>.test.ts   # single API test file
pnpm run db:seed      # idempotent: creates all tables + demo project (fresh-install path; no accounts)
pnpm run db:migrate   # applies unapplied drizzle/ migrations (0000-0007 only; 0008 已由 migrate-custom 处理)
pnpm run db:generate  # drizzle-kit generate after editing server/db/schema.ts
pnpm run db:backup    # JSON snapshot of all tables → backups/
pnpm run db:studio    # drizzle-kit studio
```

After schema changes: `db:generate` → `db:migrate` (0000-0007), then re-run `pnpm exec tsx scripts/migrate-custom.ts` if 0008 (column drops) needs re-applying. After adding env config: update `.env.example`.

Migrations run `drizzle/0000`–`0007` (0000-0006 are the core; 0007 creates the entire research arc). **0008 was removed from the journal** — the 2026-08-25 single-user port dropped the `accounts` table, `owner_id` columns, and rebuilt `ai_settings` as a global single-row table via `scripts/migrate-custom.ts` (run individually per statement; the drizzle migrator's libsql batch path triggers `SQLITE_UNKNOWN_0` on multi-statement migrations). The live DB was migrated in place.

## Runtime / Env

- `server/env.ts` builds `AppBindings` from `process.env` (`loadBindings()`); `server/node.ts` and scripts use it. Tests construct bindings directly.
- `DATABASE_URL` defaults to `file:./data/argumesh.db` (relative to repo root); remote `libsql://` URLs also work with `DATABASE_AUTH_TOKEN`.
- No auth tokens — single-user local workbench, the API is open on localhost. Do NOT expose the API port to untrusted networks.
- `.env` is optional (dotenv). `.env.example` documents AI config; never ship real keys.
- `data/`, `backups/`, `dist/`, `node_modules/` are gitignored.

## Architecture

### Backend (`server/`)

- `node.ts` — Node entry: mounts `app` for `/api/*`, serves `dist/` static + SPA fallback for everything else.
- `index.ts` — Hono app: only `/api/health` is public; **no auth gate** (single-user local workbench — API open on localhost). 16 route modules mount on `/api`; `onError` re-emits `HTTPException` as JSON.
- (auth/ removed in the 2026-08-25 single-user port — migration 0008 dropped `accounts`, `owner_id`, `/api/login`, `/api/users`; no session/password/ownership code.)
- `db/client.ts` — `createDatabase(env)`; libSQL clients are **cached by connection URL** (file mode: one handle per URL). `db/schema.ts` is the canonical table list (**23 tables** — 10 core incl. `ai_settings` (global, no accounts), plus 13 research-arc tables below).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fyuan0206/ArguMesh](https://github.com/Fyuan0206/ArguMesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
