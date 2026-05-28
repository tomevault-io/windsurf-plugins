---
trigger: always_on
description: Generic monorepo boilerplate. Clean Architecture + DDD. No business features.
---

# CLAUDE.md

Generic monorepo boilerplate. Clean Architecture + DDD. No business features.

> **Detailed rules live in sub-CLAUDE.md files — auto-loaded by Claude Code via recursive lookup the moment you read/edit a file under the path.**
> - `apps/api/CLAUDE.md` → high-level server (CQRS, DI inwire, Hono RPC, BetterAuth server, storage, org scoping, logging). Read before touching `packages/{ddd-kit,drizzle,access-control}/**` too.
>   - `apps/api/src/modules/CLAUDE.md` → per-module: layers, DDD primitives, domain events, testing, code patterns
>   - `apps/api/src/shared/CLAUDE.md` → shared kernel: port placement decisor, transaction.ts exception
> - `apps/app/CLAUDE.md` → high-level client (layout, import direction, naming, theme). Read before touching `packages/{ui,access-control}/**` too.
>   - `apps/app/src/features/CLAUDE.md` → per-feature: anatomy, routing 2-file pattern, queries/mutations, composition patterns, form/schema contracts
>   - `apps/app/src/shared/CLAUDE.md` → shared front: api-client, auth-client, route gates, authorization, org-scoping front
>
> **For conceptual / cross-cutting questions** ("how should I structure a new module?", "where does this rule live?"), read the relevant sub-CLAUDE.md before answering — that's where the rules are.

## Philosophy

Lean Startup — **Build → Measure → Learn**. Stack ships SaaS plumbing (auth, billing, multi-tenant, email, storage) and isolates the domain so pivots don't trash the foundation. "Done > perfect" applies to features; the rules in the sub-CLAUDE.md are non-negotiable — what *makes* shipping fast sustainable.

## Working method

Library API/config/SOTA unclear → **check docs first**. Outdated patterns are a frequent failure mode. Primary: Context7 MCP via `explore-docs`. Fallback: `websearch`/`WebFetch`.

## Stack

- **Runtime**: Bun 1.3+ (api+scripts), Node 24.15+ (tooling) · **API**: Hono on native `Bun.serve()` — `bun build` (prod), `bun --hot` (dev)
- **App**: Vite 8 + React 19 + TanStack Router/Query + Tailwind 4 · **UI**: shadcn/ui (`@packages/ui`) + `sonner` + `next-themes`
- **DB**: Drizzle + Postgres 17 (Docker, port `5433`) · **DI**: `inwire` (type-inference container)
- **Auth**: BetterAuth (Drizzle adapter + `twoFactor`, `passkey`, `magicLink`, `bearer`) — module-level singleton, never wrapped in DI
- **Observability**: `pino` + `hono-pino` · **Contract**: Hono RPC (`hc<AppType>`)
- **Primitives**: `@packages/ddd-kit` (`Result`, `Option`, `Entity`, `Aggregate`, `ValueObject`, `UUID`, `DomainEvent`, `BaseRepository`, `ScopedRepository`, `IUnitOfWork`)
- **Tooling**: pnpm 10 + Turborepo + Biome + Husky + commitlint + semantic-release + knip + jscpd · **Testing**: `bun test` (api) + `vitest` (packages, app)

## Cross-cutting rules (always apply)

1. **Adding a rule — omnipotent or it doesn't belong.** A rule states a *principle* tied to an architectural property and survives swapping any library/version/path it references. Phrase library-agnostic; only name a tool when it *is* the property (Zod = "validate at boundary"). Always include the **why**. Promote on 2nd occurrence. Rewrite or delete a rule the moment its property changes.
2. **Reusability-first — promote, don't duplicate.** 2nd occurrence is the trigger. Once promoted, call site has zero cosmetics.
3. **Zero warnings, zero errors before push.** Husky/lint-staged/commitlint/pre-push/CI green (Biome, knip, jscpd, type-check). No `--no-verify`. Intentional warning → `/* biome-ignore <rule>: <why> */`. Contract: green `pnpm ci:check`.
4. **Internal packages ship source, not build artifacts.** Private workspace packages (consumed only in-monorepo) point `exports` directly at `src/`; no `main`/`types` pointing to emitted output, no `build` script, no generated `dist/`. **Why**: file-watchers in dev (container sync, IDE, native watchers) almost always exclude generated directories to avoid host↔runtime collision — a build step at startup then makes every source change produce a stale artifact until manual rebuild. Source types are also more accurate than emitted `.d.ts`, and modern bundlers inline workspace deps from source at app build time. **Exception**: a package explicitly designed for npm publishing keeps its build pipeline, but its top-level `main`/`types`/`exports` still point at `src/` — the rewrite to `dist/` happens only at publish time via `publishConfig.{main,types,exports}` (pnpm/npm swap these in during `pnpm publish`). **Why**: pointing top-level fields at `dist/` re-creates the stale-artifact trap for every monorepo consumer (fresh clone has no `dist/` → `Cannot find module` until someone runs `pnpm build`). The moment the answer to "is this published?" is no, the build (and the `publishConfig` block) goes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axelhamil/clean-stack](https://github.com/axelhamil/clean-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
