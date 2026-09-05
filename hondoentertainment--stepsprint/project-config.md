---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

StepSprint is a month-long step-challenge platform with teams, daily step submissions, weekly/monthly leaderboards, and an admin console. Monorepo with a React client, Express API, and Prisma database (SQLite in dev, Postgres planned for prod).

Note: `package.json` still names the root workspace `schaffer-shufflers` — legacy demo name. User-facing product name is **StepSprint**.

## Workspace layout

```
client/             Vite + React 19 + TypeScript frontend (React Router 7)
  public/icons/     PWA icons (192, 512)
  e2e/              Playwright specs (desktop + mobile)
server/             Express 5 + TypeScript API (JWT auth, Zod validation)
  prisma/           Canonical Prisma schema (SQLite active;
                    Postgres variant at schema.postgresql.prisma), migrations
  src/              App code, routes, middleware, logger, sentry, openapi
  src/seed.ts       Seed entry point (wired via server/prisma.config.ts)
api/                Vercel Function entry (api/[...all].js) — wraps the
                    compiled Express app at /api/*
docs/               LAUNCH.md, DEPLOYMENT.md, PRODUCTION.md, design notes
.github/workflows/  CI (server + client tests + lint + build + smoke + E2E)
scripts/            vercel-build.mjs (Vercel build orchestrator),
                    check-api-health.mjs, switch-to-postgres-schema.mjs
vercel.json         Single Vercel project (SPA + Function + Cron)
```

## Commands

Run from repo root unless stated:

```bash
npm test                # client + server unit/integration (Vitest)
npm run test:coverage   # both with coverage
npm run test:e2e        # Playwright (desktop + Pixel 5 + iPhone 13)
npm run lint            # client eslint
npm run build           # client + server build
npm run check:api       # npm run check:api -- https://your-host [ --strict ]; launch gate validates release + transactionalEmail + CORS when --strict
npm run postgres:parity # swap Prisma to Postgres files for local parity (see docs/DEPLOYMENT.md)
```

Per-workspace:

```bash
cd server && npm run dev          # API on :3001 (tsx watch)
cd server && npm run db:migrate   # prisma migrate dev
cd server && npm run db:seed      # admin + sample users + demo challenge
cd client && npm run dev          # Vite on :5173
cd client && npm run lint         # eslint
cd client && npm run build        # tsc -b && vite build (emits PWA SW)
cd client && npm run test:e2e:ui  # Playwright UI mode
```

Required env (see `.env.example`):
- `JWT_SECRET` — min 16 characters in development; **min 32 in production** (enforced at startup; Vitest uses a longer CI test secret)
- `DATABASE_URL` — defaults to `file:./dev.db` (SQLite)
- `APP_ORIGIN` — CORS origin for the client
- Optional: `SENTRY_DSN`, `VITE_SENTRY_DSN`, `VITE_POSTHOG_KEY`, `LOG_LEVEL`, SMTP vars

Seed users after `db:seed`:
- Admin: `admin@stepsprint.local`
- Participant (used by E2E): `user1@stepsprint.local`
- Password (all seeded users): `password123`

## Architecture

### Server (`server/src`)
- `app.ts` / `index.ts` — Express wiring: helmet → pino-http (request IDs) → rate limiters → CORS → JSON → routes → Sentry error handler → 404
- `routes/` — modular routers: `auth`, `admin`, `challenges`, `submissions`, `leaderboards`, `summary`, `analytics`, `invites`, `notifications`, `integrations`, `openapi`. Tests colocate (`*.test.ts`, Supertest)
- `middleware/auth.ts` — JWT verification + role gates (`ADMIN` / `PARTICIPANT`)
- `middleware/rateLimit.ts` — four tiers: `authLimiter`, `apiLimiter`, `generalLimiter`, `passwordResetLimiter` (prod only)
- `logger.ts` — pino (pretty in dev, JSON in prod), level from `LOG_LEVEL`
- `sentry.ts` — `initSentry()`; no-op without `SENTRY_DSN`
- `openapi.ts` + `routes/openapi.ts` — OpenAPI 3 spec at `/api/openapi.json`, Swagger UI at `/api/docs`
- `config.ts` — Zod-validated env loader (`JWT_SECRET`, `DATABASE_URL`, `APP_ORIGIN`, email `RESEND_API_KEY`/`SMTP_*`, optional `ALLOW_PRODUCTION_WITHOUT_EMAIL` for non-public prod)
- `services/` — email (Nodemailer; no real SMTP provider configured)
- `utils/` — Luxon date helpers, bcrypt, reset tokens
- `prisma.ts` — Prisma client via `@prisma/adapter-better-sqlite3`, reads `DATABASE_URL`
- Input validation: Zod schemas on every mutating route
- Errors: custom `ApiError` class; central handler converts to JSON

### Client (`client/src`)
- `main.tsx` — `initSentry()` → React root wrapped in `ErrorBoundary` → `<App>`; registers PWA SW in prod
- `App.tsx` — router + providers; admin/leaderboard/standings routes are `React.lazy` code-split with `<Suspense>` fallback
- `api.ts` — fetch wrapper, `ApiError`, auth token handling
- `components/` — page-level (`Home`, `Admin`, `Login`, `WeeklyLeaderboard`, `Submit`, `TeamStandings`, `InvitePage`, `ForgotPassword`, `ResetPassword`) and UI (`Layout`, `ConfirmDialog`, `WeekPicker`, `ErrorBoundary`)
- `contexts/` — `WeekContext` for the selected calendar week
- `hooks/` — `useAuth`, `useChallenges`, etc.
- `sentry.ts` — `initSentry()` + `captureException` helper; `ErrorBoundary.componentDidCatch` forwards here

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hondoentertainment/StepSprint](https://github.com/hondoentertainment/StepSprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
