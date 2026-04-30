---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Repository Layout

Turborepo + pnpm workspaces. Node >= 22, pnpm 10.11+. Workspace globs: `apps/*`, `packages/**/*`.

```
apps/frontend/              Next.js 15 app (workspace name: "next-app-template")
packages/api/               tRPC v11 router + Express webhooks + BullMQ workers
packages/auth/              NextAuth (v4 + v5-beta transition — see trpc.ts note)
packages/db/                Prisma 6 client + MongoDB schema
packages/email/             React Email templates + Resend
packages/validations/       Zod schemas (single source of truth for API + forms)
packages/utils/             Shared utilities, feature flags
packages/web3/              Web3 helpers
packages/configs/           eslint / prettier / tailwind / tsconfig workspaces
.claude/rules/              Domain-specific rules — read before editing a domain
```

## Architecture

### Data flow

`apps/frontend` → tRPC client → `packages/api` (`appRouter` in `packages/api/src/root.ts`) → domain router → service → repository → `@package/db` (Prisma + Mongoose where noted).

### API layering (every domain in `packages/api/src/routers/{domain}/`)

```
{domain}.repository.ts    Prisma/Mongoose queries only — no business logic
{domain}.service.ts       Business rules, orchestration, TRPCError throws
{domain}.router.ts        tRPC procedures — validate input (Zod), delegate to service
index.ts                  Re-exports
```

Do not inline Zod schemas in routers. Import from `@package/validations`.

### tRPC procedures (defined in `packages/api/src/trpc.ts`)

- `publicProcedure` — no auth
- `protectedProcedure` — requires session; blocks un-onboarded users via `featureFlags.onboardingFlow` (except `user.updateUserOnboarding`)
- `adminProcedure` — requires `role === "ADMIN"`
- `chiefProcedure` — `ADMIN` or `CHIEF`
- `loggedProcedure` — fire-and-forget activity logging for non-ADMIN users; sanitizes password/token fields and truncates strings >200 chars

### Domain routers (17 — see `packages/api/src/root.ts`)

`user`, `admin`, `request`, `newsletter`, `transaction`, `bill`, `invoice`, `cloudflare`, `booking`, `calendar`, `chatbot`, `chat`, `prospect`, `infrastructure`, `permission`, `activity`, `document`

### Storage / external services

- **MongoDB** via Prisma (`MONGO_URL`) — must run as replica set `rs0` for transactions
- **Cloudflare R2** via `@aws-sdk/client-s3` (S3-compatible, pre-signed URLs)
- **Twilio** webhooks (WhatsApp) — Express router in `packages/api/src/webhooks/`, runs on port 3003, requires signature verification
- **Resend** for transactional email (React Email templates)
- **Google Calendar** OAuth — bidirectional sync with `booking` domain
- **Genkit AI** (`packages/api/src/config/ai.config.ts`) with Mistral + Google AI + Deepseek plugins; prompts live in `packages/api/src/prompts/`
- **BullMQ + Redis** for background jobs (`packages/api/src/jobs/chat/`, `packages/api/src/jobs/document/`)

### Frontend specifics

- Custom Next.js server (`apps/frontend/server.ts`) — required for live chat WebSockets on `/ws`. `pnpm dev` uses this server, not `next dev`.
- App Router (`apps/frontend/app/`), server components by default; `"use client"` only when needed (hooks, event handlers, R3F)
- Three.js via `@react-three/fiber` + drei; lazy-load scenes with `dynamic(() => import('./Scene'), { ssr: false })`
- tRPC UI docs at `/api/docs` in development only
- `tsconfig` target is `es5` — use `Array.from()` for Set/Map iteration

### API package export boundary

`packages/api/package.json` exposes only `"."` and `"./types"`. New public symbols must be re-exported from `packages/api/src/index.ts`.

## Commands

### Root (from repo root)

```
pnpm dev                     Redis (docker) + turbo dev (full stack)
pnpm dev:no-redis            turbo dev without starting redis
pnpm build                   turbo build (cascades db:generate first)
pnpm typecheck               turbo typecheck
pnpm lint                    turbo lint + manypkg check (monorepo consistency)
pnpm format                  prettier across repo
pnpm db:push                 push Prisma schema to MongoDB (no migrate)
pnpm db:studio               Prisma Studio
```

### Workspace-scoped

```
pnpm --filter @package/api webhooks:dev       Express Twilio webhook server (port 3003)
pnpm --filter @package/api worker:chat        BullMQ chat worker
pnpm --filter @package/api worker:document    BullMQ document worker
pnpm --filter @package/api typecheck          type-check API only
pnpm --filter next-app-template dev           frontend only (custom ws server)
pnpm --filter next-app-template typecheck     tsc --noEmit on frontend
pnpm --filter next-app-template lint          eslint frontend
```

### Database (from `packages/db/`)

```
pnpm db:generate             prisma generate (runs automatically on install)
pnpm db:push                 prisma db push --skip-generate
pnpm db:migrate              prisma migrate dev
pnpm db:deploy               prisma migrate deploy (production)
pnpm db:seed                 run prisma/seed.ts (requires MONGO_URL env + rs0 replica set)
pnpm db:seed:admin           seed admin user only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alpadev-fx/alpadev_ai_monorepo](https://github.com/alpadev-fx/alpadev_ai_monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
