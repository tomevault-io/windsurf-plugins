---
trigger: always_on
description: pnpm 11 + Turborepo monorepo. Requires Node >= 22.13.
---

# Supabase Monorepo

pnpm 11 + Turborepo monorepo. Requires Node >= 22.13.

## Structure

| Directory                | Purpose                                                                                                            |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| `apps/studio`            | Supabase Studio/Dashboard — has its own `apps/studio/AGENTS.md` (see below)                                        |
| `apps/docs`              | Documentation site — Next.js app router, MDX (port 3001, served under `/docs`) — has its own `apps/docs/AGENTS.md` |
| `apps/www`               | Marketing website — Next.js, app + pages (port 3000)                                                               |
| `apps/design-system`     | Component demos — source of truth for Studio UI patterns (port 3003)                                               |
| `apps/ui-library`        | shadcn-style registry site for Supabase UI blocks (port 3004)                                                      |
| `apps/lite-studio`       | Lightweight Studio — different stack: React Router 7 + Vite + Tailwind v4                                          |
| `apps/kb`                | Knowledge base — Astro — has its own `apps/kb/AGENTS.md`                                                           |
| `apps/learn`             | Courses site — Next.js + Contentlayer (port 3007), early stage                                                     |
| `packages/ui`            | Shared UI components (shadcn/ui based) — `import { Button } from 'ui'`                                             |
| `packages/ui-patterns`   | Composite components — subpath imports, e.g. `ui-patterns/AssistantChat`                                           |
| `packages/common`        | Shared utils, telemetry constants, feature flags                                                                   |
| `packages/api-types`     | Generated platform Management API types                                                                            |
| `packages/pg-meta`       | SQL builders for Postgres introspection (`SafeSqlFragment`)                                                        |
| `packages/shared-data`   | Static data: pricing, plans, regions, error codes                                                                  |
| `e2e/studio`, `e2e/docs` | Playwright E2E tests                                                                                               |
| `supabase/`              | Local Supabase project: edge functions, migrations, config.toml                                                    |

## Common Commands

```bash
pnpm dev:studio              # run Studio dev server → http://localhost:8082
pnpm dev:docs                # run docs dev server
pnpm dev:www                 # run www dev server
pnpm dev:kb                  # run knowledge base dev server
pnpm test:studio             # Studio unit tests (vitest)
pnpm e2e                     # Studio E2E tests (playwright)
pnpm build --filter=studio   # build Studio
pnpm lint --filter=studio    # lint Studio
pnpm typecheck               # typecheck all packages
pnpm format                  # Prettier write (check: pnpm test:prettier)
pnpm generate:types          # local DB types → supabase/functions/common/database-types.ts
pnpm api:codegen             # platform Management API types → packages/api-types
```

## CI

Every PR must pass typecheck + lint (one workflow), Prettier, and a typos check. Other checks are path-filtered: Studio unit tests/build and the lint ratchet (ESLint warning count must not increase) run on `apps/studio/**` changes; app-specific test suites run on their own paths.

Never hand-edit generated files: `packages/api-types/types/**`, `**/routeTree.gen.ts`, `**/__generated__/**`, `apps/docs/features/docs/generated/**`, `apps/www/.generated/**`, `supabase/functions/common/database-types.ts`, `apps/docs/content/_partials/access-control/scoped_pat_*.mdx` (run `make -C apps/docs/spec generate.partials.access-control`).

## Conventions

**UI** — import from `'ui'`; primitives are shadcn/ui-based and exported unsuffixed (`Input`, `Select`, `Form`, …). Use `Button` — the in-house component and the standard everywhere (a raw shadcn `Button_Shadcn_` also exists but is rarely the right choice). Check `packages/ui/index.tsx` before creating new primitives. Higher-level patterns live in `packages/ui-patterns`.

**Styling** — Tailwind only, semantic tokens (`bg-muted`, `text-foreground-light`), no hardcoded colors.

**Exports** — named exports only; default exports are allowed only where a framework requires them (`pages/**`, `app/**`, config files — the eslint preset has the exact carve-out list). Lint-enforced across all apps via `eslint-config-supabase` (severity `warn` everywhere; hard-enforced in Studio by the lint ratchet).

**Language** — Use U.S. English everywhere.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [supabase/supabase](https://github.com/supabase/supabase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
