---
trigger: always_on
description: Map for AI coding agents working in this repository. Short on purpose — deeper
---

# AGENTS.md

Map for AI coding agents working in this repository. Short on purpose — deeper
sources of truth are linked at the bottom.

**Read the doc for what you are touching before you open code** — then fix the
doc if it was insufficient:

| You are… | Read |
| --- | --- |
| changing backend code | the feature doc in [docs/features/](docs/features/index.md) — table, service, router, procedures, all in one place |
| building UI against an API | **only** the contract header of `apps/api/src/__tests__/<feature>.api.test.ts` — payloads, responses, error codes |
| touching a page | its route doc in [docs/routes/](docs/routes/index.md) |
| building a whole feature | [docs/rules/WORKFLOW.md](docs/rules/WORKFLOW.md) — plan, then API + UI in parallel, then integration |

The rules you must follow live in [docs/rules/](docs/rules/COMMON.md).

## What this is

A Bun + Turborepo TypeScript monorepo template: Next.js web app + NestJS API,
connected end-to-end with tRPC. It keeps one example of each pattern — one
application shell (`components/layout/`), one shared package (`@reclit/ui`) — so
you can copy the pattern to build features. The `spreadsheet` feature is the
worked backend slice, Postgres → Prisma → service → tRPC (plus a REST
controller), fully documented by its contract test, and `/ai-spreadsheet`
consumes it. `/` is the dashboard shell. There is no auth yet.

## Layout

| Path | Package | Purpose |
| --- | --- | --- |
| `apps/api` | `@reclit/api` | NestJS API server (Bun runtime), port **4001**, tRPC mounted at `/trpc`, Prisma + Postgres |
| `apps/dashboard` | `@reclit/dashboard` | Next.js 16 App Router web app, port **4000** |
| `packages/ui` | `@reclit/ui` | The one shared package: `Button`, `Input`, `Label`, `Select`, `Calendar`, `Spinner` + `focusRing` + `cn` + Tailwind preset |

## Commands

```bash
bun install                 # install all workspaces
bun dev                     # run api + dashboard in parallel
bun run dev:api             # api only (http://localhost:4001)
bun run dev:dashboard       # dashboard only (http://localhost:4000)
bunx turbo typecheck        # typecheck all workspaces
bunx turbo lint             # biome lint (bunx turbo lint:fix to autofix)
bun run format              # biome format --write
bun run build:dashboard     # production build of the dashboard (.next/standalone)
bun run start:dashboard     # serve the built dashboard
bun run start:api           # run the api in production mode (bun runs TS directly; no build step)
bunx turbo test             # run tests (bun test; api smoke + feature contracts)
bun run --filter=@reclit/api db:generate   # regenerate the Prisma client
bun run --filter=@reclit/api db:migrate    # create + apply a migration
```

Filter to one workspace: `bunx turbo typecheck --filter=@reclit/api`.

## Conventions

- **Formatting/linting is Biome** (`biome.json`), not ESLint/Prettier. Run `bun run format` before committing.
- **Dependency versions**: shared deps are pinned in the root `package.json` `"catalog"`
  field; workspace packages reference them as `"react": "catalog:"`. Workspace-internal
  deps use `"@reclit/x": "workspace:*"`.
- **Path aliases**: `@api/*` → `apps/api/src/*` (inside the api), `@/*` → `src/*` (inside dashboard).
- **Every dashboard string is a message key** in `apps/dashboard/src/messages/en.json`,
  and every text size is a named step in the `fontSize` scale in
  `packages/ui/tailwind.config.ts`. Literal copy and raw `text-sm`/`text-2xl` in
  a component are both bugs — see [docs/rules/FRONTEND.md](docs/rules/FRONTEND.md).
- **Skills**: `backend-feature`, `frontend-feature`, `api-testing`,
  `feature-workflow`, `new-package`. Load the matching one before writing code.
- **Feature layout**: one folder per feature in `apps/api/src/modules/<feature>/`
  (`schema` + `service`) — see [docs/rules/BACKEND.md](docs/rules/BACKEND.md).
  Frontend components live in `apps/dashboard/src/components/{layout,common,<feature>}/`,
  shared primitives in `packages/ui` — see [docs/rules/FRONTEND.md](docs/rules/FRONTEND.md).
- **Every API is documented and tested in one file**:
  `apps/api/src/__tests__/<feature>.api.test.ts` — see [docs/rules/TESTING.md](docs/rules/TESTING.md).
- **Test files never live in frontend source.** No `*.test.ts`, stub, fixture or
  other test-only file goes under `apps/dashboard/src/`. Frontend tests live in
  one central place, `apps/dashboard/tests/` (`support/` for shared helpers, one
  directory per feature), and reach the code through the `@/` alias — see
  [docs/rules/TESTING.md](docs/rules/TESTING.md).
- **Plans**: anything larger than a one-file edit gets `docs/plans/NNN-<slug>.md`
  written before the code and an `Outcome` after — see [docs/rules/COMMON.md](docs/rules/COMMON.md).
- **Each workspace carries its own `tsconfig.json`** — there is no shared tsconfig package.
- **Database**: Prisma, schema at `apps/api/prisma/schema.prisma`, single client
  at `apps/api/src/db/prisma.ts`. `DATABASE_URL` lives in `apps/api/.env`.

## Hard invariants (breaking these causes confusing failures)

1. `apps/api/package.json` must keep exporting `"./trpc/routers/_app"` — it is the
   only type bridge to the dashboard.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theharoonali/Reclit](https://github.com/theharoonali/Reclit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
