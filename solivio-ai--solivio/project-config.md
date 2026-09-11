---
trigger: always_on
description: This repository is intended to stay easy to launch for contributors evaluating the idea.
---

# Agent Notes

This repository is intended to stay easy to launch for contributors evaluating the idea.

The sections below are the quick reference. They summarize the boundaries; the detailed sections later in this file (Development, Modules, Database, UI) and the docs in `docs/` remain authoritative.

## Always

- Match the task to the **Task Router** below and read the relevant `docs/` guide before researching or coding. A task can match multiple rows — read all of them.
- Run `yarn check` and `yarn test` before handing work back; add `yarn typecheck` whenever TypeScript, API contracts, server code, or React behavior changes. Auto-fix formatting first with `yarn biome check --write .`. For a full repo health check before handoff, run the single command: `yarn validate:all`.
- Run `yarn generate` after changing module files or `solivio.config.ts` — the generated registries and app-router stubs are how module code reaches the app. (`yarn dev` keeps a generator watcher running for you.)
- Keep feature code in modules (`modules/<id>/src/`); modules depend only on `@solivio/sdk` and the shared packages (`@solivio/ui`, `@solivio/theme`, `@solivio/domain`), and reach shared infrastructure only through `@solivio/sdk/runtime` accessors.
- Read each module's own `AGENTS.md` (e.g. `modules/customers/AGENTS.md`) before changing it; `modules/products-sync` is the reference example exercising every module surface.
- Change database schema only through committed Drizzle migrations in the owning journal (`yarn db:generate` for core, `yarn db:generate <moduleId>` for a module) and verify with `yarn db:check`.
- Build UI from the shared shadcn/ui kit in `packages/ui` and Tailwind utilities; check https://ui.shadcn.com/docs/components before writing a new component.
- Use mocks until the data model and integration boundaries are clear, and keep setup commands simple and documented.
- After a correction, record a rule in `docs/lessons.md` (see **Self-improvement**) so the same mistake cannot recur.

## Ask First

- Before changing architecture, the public `@solivio/sdk` contract surface, or the module/SDK boundary (`docs/contracts.md`).
- Before changing the generator (`scripts/generate/`) or the shape of its emitted artifacts (`docs/codegen.md`).
- Before adding a required external service to the default demo path, or adding a production dependency.
- Before applying migrations in any shared or non-local environment.
- Before reducing scope or changing behavior the user or a doc did not explicitly ask to change.

## Never

- Never edit `apps/solivio/src/generated/` or the generated app trees (`apps/solivio/src/app/(protected)/(gen)/`, `(gen-public)/`, `api/(gen)/`) — they are generator-owned, gitignored, and overwritten by `yarn generate`.
- Never import another module at runtime — cross-module calls go through `getService()` and typed events only. The one sanctioned exception is a **type-only** import of a dependency module's `services.ts` (erased at runtime) to pull its `Services` augmentation into the typecheck.
- Never make a module import app internals (`@/...`, `@solivio/app`) — modules depend only on `@solivio/sdk` and the shared packages.
- Never SQL-join another module's tables. Cross-module references are id-only columns (no FK constraints); fetch display data through batch service lookups (`findByIds`-style).
- Never name a module table anything other than `<module_id>` or `<module_id>_*` (snake_case, hyphens in the module id become underscores; see `docs/database.md`).
- Never run module code at import time that touches the runtime (`getDb()`, `getService()`, `getAi()`, …) — instantiate agents and resolve services lazily inside handlers/factories; the runtime boots in `instrumentation.ts` after modules are imported.
- Never import the Drizzle client (`apps/solivio/src/server/database/db.ts`) outside `apps/solivio/src/server/` or `apps/solivio/src/app/api/`; module code uses `getDb()`/`db` from `@solivio/sdk/runtime` with its own `data/schema.ts` tables.
- Never write custom CSS classes or add rules to `globals.css` beyond the allowed blocks, and never hard-code theme colors — use the theme tokens.
- Never add required external services to the default demo path.

## Validation Commands

Choose the smallest relevant set for the change:

```bash
yarn validate:all            # full repo health/handoff suite: env, immutable install, validate, db:check, build, setup, e2e
yarn validate                # the PR gate: generate --check + biome/boundaries + typecheck + Vitest
yarn biome check --write .   # format, sort imports, apply safe lint fixes
yarn generate                # regenerate module wiring (add --check to validate only)
yarn check                   # Biome quality gate + module boundary checker (CI runs this)
yarn test                    # Vitest suite
yarn typecheck               # when TS, API contracts, server code, or React behavior changes
yarn dedupe --check          # verify lockfile deduplication after dependency changes (`yarn dedupe` fixes it)
yarn db:check                # journals match schemas (core + every module journal)
yarn e2e                     # Playwright against http://localhost:3000 (yarn setup first)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solivio-ai/solivio](https://github.com/solivio-ai/solivio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
