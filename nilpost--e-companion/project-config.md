---
trigger: always_on
description: > Primary reference for all agents and humans working on this project.
---

# AGENTS.md — E-Companion

> Primary reference for all agents and humans working on this project.
> Read this at the start of every session. Keep it short and accurate — it is not a tutorial.

---

## Project

Pet care companion app. Users manage pets, schedule feeding/care reminders, and log health events.

- **Live URL**: https://companion.postiusgroup.com
- **Repo**: https://github.com/nilpost/E-Companion
- **Owner**: nilpost

---

## Session files — read all three at session start

| File | Purpose |
|------|---------|
| `AGENTS.md` | Stack, conventions, agent registry (this file) |
| `STATUS.md` | Current sprint, in-progress items, blockers, decisions log |
| `BACKLOG.md` | All tasks P0 (blocking) → P3 (someday) |

**PO agent**: reads all three + `git log --oneline -10` before doing anything. Updates `STATUS.md` and `BACKLOG.md` and commits them at end of session.

---

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + TypeScript + Vite + Tailwind CSS + shadcn/ui |
| Backend | Express.js + Node.js (ESM modules) |
| Real-time | WebSockets (ws) |
| Auth | Passport.js — session-based (not JWT) |
| ORM | Drizzle ORM |
| Database | PostgreSQL via Supabase |
| Hosting | Railway — auto-deploys from `main` branch |
| DNS / CDN | Cloudflare — `companion.postiusgroup.com` |

---

## Key file paths

```
shared/
  schema.ts           ← Drizzle schema — single source of truth for DB shape
  types.ts            ← shared TypeScript types (client + server)

server/
  index.ts            ← entry point
  routes.ts           ← ALL API route definitions
  auth.ts             ← Passport.js config, isAuthenticated middleware
  storage.ts          ← ALL Drizzle queries (no DB calls in routes)
  db.ts               ← DB connection (Drizzle + pg)
  vite.ts             ← Vite dev server (dev only) + serveStatic (prod)

client/src/
  pages/              ← route-level components (one per page)
  components/         ← reusable UI components
  hooks/              ← custom React hooks
  lib/                ← utilities, API client

.claude/agents/       ← agent definitions (all agents live here)
nixpacks.toml         ← Railway build config
```

---

## Database

| Setting | Value |
|---------|-------|
| Provider | Supabase |
| Project ref | `ynegbxsokzonsjrandyi` |
| Region | AWS ap-northeast-1 |
| Schema file | `shared/schema.ts` |
| Migration style | Push-based (`npx drizzle-kit push`) — no migration files |
| Runtime connection | `DATABASE_URL` env var |
| Migration connection | `DATABASE_URL_DIRECT` env var (Supavisor session-mode pooler, port 5432) |

---

## Environment variables

| Variable | Required | Notes |
|----------|----------|-------|
| `DATABASE_URL` | yes | Supabase connection string (pooled) |
| `DATABASE_URL_DIRECT` | migrations only | Supavisor **session-mode** pooler (`...pooler.supabase.com:5432`) — use for `drizzle-kit push`. Session mode supports session-level DDL and is IPv4-safe for Railway; the direct `db.<ref>.supabase.co:5432` endpoint is IPv6-only and fails from Railway. **As of 2026-07-21, `drizzle.config.ts` code reads this correctly (PR #14), but the value itself still needs to be set in Railway's service variables — see STATUS.md blockers / BACKLOG.md INFRA-06.** |
| `SESSION_SECRET` | yes | Passport session secret — must be long random string in prod |
| `NODE_ENV` | yes | `production` on Railway, `development` locally |
| `PORT` | no | Railway sets this automatically; defaults to 5000 locally |

Never hardcode these. Never commit `.env`.

---

## Build & deploy

```bash
# Install (Railway uses this — includes devDeps needed for build)
npm install --include=dev

# Build (Vite for client + esbuild for server)
npm run build

# Start (production)
npm run start

# Local dev
npm run dev
```

**Railway specifics**:
- Auto-deploys on push to `main`
- `nixpacks.toml` overrides install to `npm install --include=dev`
- Node 18 on Railway — do NOT use `import.meta.dirname` (Node 21+ only)
- Use `fileURLToPath(import.meta.url)` or `process.cwd()` instead

**Verifying a deploy went live**: Claude Code web/agent sessions run behind an outbound network allowlist that does NOT include `companion.postiusgroup.com` by default — `curl`/`WebFetch` to the live URL will 403 at the proxy, not at the app. Don't report this as "site is down"; report it as "cannot verify from this sandbox" and ask the human to check the Railway dashboard, or to widen the session's network policy (see https://code.claude.com/docs/en/claude-code-on-the-web) if live verification is needed repeatedly.

---

## Postmortems / Learnings

> Durable lessons from real incidents. Add an entry when a bug reveals a pattern likely to recur — not for routine fixes.

- **2026-07-17 — Unhandled promise rejections crash the whole process (BUG-01 / HARDEN-01)**: Any `async` Express handler (route handler, or a callback library like `passport-local` that doesn't await/catch its callback) whose rejection is never caught becomes an unhandled promise rejection. Node exits on this by default, and Railway's `ON_FAILURE` restart policy then serves 502s to *every* user until it restarts — not just the one request that failed. Fix pattern: never leave an `async` callback unguarded.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nilpost/E-Companion](https://github.com/nilpost/E-Companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
