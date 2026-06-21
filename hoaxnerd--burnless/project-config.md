---
trigger: always_on
description: Guidance for AI coding agents (and humans) working in this repository. Style: terse and dense — written for fast machine reading. Every line below is a stable technical fact, not status.
---

# AGENTS.md

Guidance for AI coding agents (and humans) working in this repository. Style: terse and dense — written for fast machine reading. Every line below is a stable technical fact, not status.

## Project overview

Burnless is an open-source, AI-native financial planning & analysis (FP&A) platform for startups and the people who run them. It reads real financials and turns them into forecasts, scenarios, and board-ready reports, with an AI companion that can answer questions and take actions on the model through tools. It self-hosts in one command with an embedded database (no Docker, no external Postgres required); a managed cloud edition runs the same codebase with more capabilities enabled.

Monorepo, pnpm workspaces + turbo. Node ≥ 20.9.

```
apps/web        → Next.js 15 (app router), React 19 — frontend + API routes + middleware
packages/db     → Drizzle ORM schema, queries, migrations, seed (PGlite local / Postgres cloud)
packages/engine → Pure-TS financial calc library. No DB, no I/O. Decimal.js precision.
packages/ai     → Provider-agnostic LLM layer (Anthropic / OpenAI / OpenRouter / Ollama)
packages/cli    → The `burnless` CLI (start / update / mcp serve / …) — the only public npm package
packages/mcp    → Model Context Protocol support (consume external MCPs + expose Burnless as one)
packages/types  → Shared TS types
packages/ui     → Shared React components
```

Internal workspace deps reference `workspace:*`. The `burnless` CLI is the public package; the rest are private.

## Setup commands

- Node **≥ 20.9**, pnpm.
- Install: `pnpm install`
- Run all dev servers: `pnpm dev` (web on `:3000`, auto-bumps the port if busy)
- Standalone instance: `burnless start` (binds loopback `127.0.0.1`, embedded database, onboarding wizard)

Dev database options:
- **Embedded PGlite (default).** No external service needed — works out of the box.
- **Postgres via Docker:** `pnpm docker:up` (Postgres + Redis + Mailpit), `pnpm docker:down` to stop, `pnpm docker:reset` to wipe.

Schema management:
- `pnpm db:push` — push schema to a dev DB (no migration files)
- `pnpm db:generate` — generate migration SQL into `packages/db/drizzle/` via drizzle-kit
- `pnpm db:migrate` — run pending migrations
- `pnpm db:studio` — Drizzle Studio
- `pnpm db:seed` — demo data (guarded against prod, idempotent)

Required env: `DATABASE_URL`, `AUTH_SECRET`. AI is optional (pick one provider): `AI_PROVIDER` plus the matching key (`ANTHROPIC_API_KEY` / `OPENAI_API_KEY` / `OPENROUTER_API_KEY`) or `OLLAMA_BASE_URL`. See `.env.example` for the full list. Env detection helpers live in `apps/web/src/lib/env.ts`.

## Build, test & lint

- `pnpm check` — type-check + lint (run before considering work done)
- `pnpm build` / `pnpm lint` / `pnpm type-check` / `pnpm test` — individual root scripts (turbo-orchestrated)
- Per-package work: `pnpm --filter @burnless/<pkg> <script>` (e.g. `pnpm --filter @burnless/db test`)
- Single test file: `pnpm --filter @burnless/web vitest run path/to/file.test.ts`
- Single test by name: `pnpm --filter @burnless/db vitest run -t "test name"`
- E2E (Playwright, in `apps/web`): `pnpm e2e` / `pnpm e2e:ui`

**Run tests per-package, not the whole suite at once.** The DB tests spin up real in-memory Postgres (PGLite) and the full web suite can time out if run all together — scope to the package or file you changed.

## Code style & conventions

- **Decimal for all money.** Never use raw `number` arithmetic for currency inside the engine. Use the Decimal helpers (`D()`, `dAdd`, `dMul`, `dSum`, `dRound2`). Output is 2-decimal numbers at the boundary.
- **Centralized currency formatter.** Never hardcode currency symbols (`$`/`€`/`£`/`¥`/`₹`) or `new Intl.NumberFormat(..., { currency: "USD" })`. In React, use `useLocale().fmtCurrency` / `fmtCompact`. In server/non-React code, use `formatCurrency(value, currency, locale, opts?)` from `@burnless/types`. Guarded by a regression test that scans for reintroductions.
- **Companies are tenants.** Every business table is scoped by `companyId` and indexed. Always thread `companyId` through queries; use the generic helpers in `packages/db/src/queries/crud.ts` (`findByIdForCompany`, `updateForCompany`, …). API routes gate access with `requireCompanyAccess()` (role order: owner > admin > editor > viewer).
- **Scenarios are the read filter.** Don't query base tables directly when a path is scenario-aware. Use `getResolvedData()` (the override resolver) or a compute helper that already applies overrides.
- **Never set `X-Scenario-Id` manually.** That header is injected in exactly one place — `apiFetch` in `apps/web/src/lib/api-fetch.ts`, derived from the `active-scenario-id` cookie. A second source drifts from the cookie and 409-locks the user out of editing. Guarded by a regression test.
- **Cache-tag revalidation.** Mutating routes call `revalidateTag(...)`. AI-tool mutations drive this through the mutation-tool / cache-tag sets in `apps/web/src/lib/ai-tools/index.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoaxnerd/burnless](https://github.com/hoaxnerd/burnless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
