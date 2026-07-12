---
trigger: always_on
description: <!-- nx configuration start-->
---

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

# General Guidelines for working with Nx

- For navigating/exploring the workspace, invoke the `nx-workspace` skill first - it has patterns for querying projects, targets, and dependencies
- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- Prefix nx commands with the workspace's package manager (e.g., `pnpm nx build`, `npm exec nx test`) - avoids using globally installed CLI
- You have access to the Nx MCP server and its tools, use them to help the user
- For Nx plugin best practices, check `node_modules/@nx/<plugin>/PLUGIN.md`. Not all plugins have this file - proceed without it if unavailable.
- NEVER guess CLI flags - always check nx_docs or `--help` first when unsure

## Scaffolding & Generators

- For scaffolding tasks (creating apps, libs, project structure, setup), ALWAYS invoke the `nx-generate` skill FIRST before exploring or calling MCP tools

## When to use nx_docs

- USE for: advanced config options, unfamiliar flags, migration guides, plugin configuration, edge cases
- DON'T USE for: basic generator syntax (`nx g @nx/react:app`), standard commands, things you already know
- The `nx-generate` skill handles generator discovery internally - don't call nx_docs just to look up generator syntax

<!-- nx configuration end-->

---

# Project: nestjs-fastify-nx

Production-grade NestJS + Fastify + Nx monorepo. DDD/CQRS, Better Auth (cookie sessions), GraphQL (Mercurius), Socket.io, BullMQ, OpenTelemetry, Sentry. Four runnable apps: `api`, `worker`, `scheduler`, `migration`.

## Stack quick reference

- **Runtime**: Node 22, pnpm 10.33, TypeScript 6
- **Framework**: NestJS 11 + Fastify 5
- **ORM**: Prisma 7 with `@prisma/adapter-pg`; schema at `prisma/schema.prisma`
- **Auth**: Better Auth 1.6 — **NOT** JWT. Cookie name is `better-auth.session_token`. Mounted at `/api/auth/*` by `BetterAuthModule` (in `libs/infra/auth`). The auth surface is published at `/api/auth/reference`.
- **Test runner**: Vitest 4 + Testcontainers (real Postgres/Redis) + Supertest. **NOT** Jest.
- **Bundler**: Webpack 5 with `swc` compiler (`@nx/webpack` auto-injects `swc-loader` with `legacyDecorator` + `decoratorMetadata`, so NestJS DI metadata stays intact). Type-checking is NOT done at build time — it lives in the separate `typecheck` target (`tsc --build`) that the CI gate runs. **Consequence**: interface/type-only imports MUST use `import type`. swc transpiles per-file with no type graph, so a _value_ import of an interface used as a `@Inject`-decorated constructor param type leaks into `design:paramtypes` and webpack warns `export 'X' was not found`. Keep class imports as value imports — their runtime reference is exactly what DI metadata needs.

## Architecture & boundaries

The monorepo enforces DDD layering via Nx tags + `@nx/enforce-module-boundaries` (see `eslint.config.mjs`):

```
scope:api / scope:worker / scope:scheduler
  → modules, composition, infra, core, shared, contracts

scope:migration
  → empty allow-list (intentional — migration is a thin prisma deploy wrapper)

scope:composition  (cross-cutting modules, e.g. admin)
  → modules, infra, core, shared, contracts

scope:modules  (bounded contexts: users, audit-log, …)
  → infra, core, shared, contracts        # NEVER another scope:modules

scope:infra
  → core, shared, contracts, infra

scope:core / scope:contracts
  → shared
```

**Key invariant**: `scope:modules` cannot depend on another `scope:modules`. Cross-context aggregation goes through `scope:composition` (one-way: composition → modules, never reverse). Do not "fix" lint errors by adding cross-module imports — extract a composition lib instead.

**Composition discipline (anti-bloat)**: `scope:composition` libs are **orchestrators only** — they wire HTTP routes + call CQRS handlers from multiple modules. Business rules MUST live on the domain entity / value object of the owning module. If a composition lib starts growing its own `domain/` or `application/` layer, the rule is being broken: the logic belongs in a module, and what's missing is a domain event from one module that another module listens to (use the outbox / EventEmitter pattern). The boundary lint will not catch this — it's a code-review smell.

Test files (`*.spec.ts`, `*.integration.ts`, `e2e/**`) are exempt from boundary rules.

## Where things live

```
apps/
  api/          HTTP + GraphQL + WebSocket entrypoint
    e2e/        Supertest e2e suite (NOT apps/api/test/)
    src/common/ filters, interceptors, swagger, throttler, health
  worker/       BullMQ consumer
  scheduler/    cron jobs (@nestjs/schedule)
  migration/    one-shot `prisma migrate deploy` + optional seed (Docker CMD bypasses
                main.ts; the file exists only so Nx sees an entry point. Gated by orchestrator
                via `depends_on: service_completed_successfully` before api/worker/scheduler boot)

libs/
  modules/      bounded contexts (DDD per module — see below)
    upload/     multipart handler (see HTTP_BODY_LIMIT_BYTES, UPLOAD_MAX_FILE_BYTES env)
  composition/  cross-cutting libs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chuanghiduoc/nestjs-fastify-nx](https://github.com/chuanghiduoc/nestjs-fastify-nx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
