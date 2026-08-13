---
trigger: always_on
description: <!-- intent-skills:start -->
---

<!-- intent-skills:start -->

## Skill Loading

Before substantial work:

- Skill check: run `pnpm dlx @tanstack/intent@latest list`, or use skills already listed in context.
- Skill guidance: if one local skill clearly matches the task, run `pnpm dlx @tanstack/intent@latest load <package>#<skill>` and follow the returned `SKILL.md`.
- Monorepos: when working across packages, run the skill check from the workspace root and prefer the local skill for the package being changed.
- Multiple matches: prefer the most specific local skill for the package or concern you are changing; load additional skills only when the task spans multiple packages or concerns.
<!-- intent-skills:end -->

# Agent Guide (Finance Tracker)

TanStack Start (React 19) full-stack finance app with SSR, Nitro v3,
PostgreSQL/Drizzle, Better Auth, Tailwind CSS v4/shadcn.

## Essentials

- **Package manager**: pnpm (not npm).
- **Path aliases**: `@/*` -> `src/*`, `~e2e/*` -> `e2e/*`, `~test/*` -> `test/*`.
- **Validation**: ArkType everywhere (not Zod). Server schema in
  `validators.ts` is the single source of truth — forms import and
  reuse it (or derive via `.pick()` / `.omit()` / `.merge()`).
  - `validators.ts` defines API contracts as independent `type({...})`
    — not derived from Drizzle insert/update schemas.
  - `models.ts` holds ArkType CRUD schemas (select/insert/update/delete)
    and entity type aliases generated via `drizzle-arktype`.
  - Cross-field validation: export `baseSchema` (pickable) +
    `schema` (narrowed). Forms `.pick()` from base schemas.
  - `drizzle-arktype` is used for select/delete schemas only.
  - Import entity types from `@/modules/{mod}/models`, not `db/schema`.
  - Import validators from `@/modules/{mod}/validators`.
- **UI style**: shadcn/ui `base-nova` (Base UI, not Radix). Use
  `render` prop / `mergeProps`, not `asChild`. Icons: Lucide
  via `@/components/icons` (never import `lucide-react` directly).
- **Typography**: Tailwind classes directly on semantic elements. See
  [styling.md](docs/development/styling.md).
- **Restricted imports** (`no-restricted-imports`):
  - `date-fns` / `@date-fns/tz` — use `@/lib/i18n/date`
  - `lucide-react` — use `@/components/icons`
  - `@base-ui/react` — only in `src/components/ui/` and
    `src/components/filters/`
  - `react-day-picker` — only in `src/components/ui/calendar.tsx`
  - `posthog-js/react` — use `@/hooks/use-analytics`

## Commands

| Task              | Command                                               |
| ----------------- | ----------------------------------------------------- |
| Dev server        | `pnpm dev`                                            |
| Build             | `pnpm build`                                          |
| Lint              | `pnpm lint` / `pnpm lint:fix`                         |
| Format            | `pnpm format` / `pnpm format:check`                   |
| Typecheck         | `pnpm paraglide:compile && pnpm typecheck`            |
| All tests         | `pnpm test`                                           |
| Unit tests        | `pnpm test:unit`                                      |
| Integration tests | `pnpm test:integration`                               |
| E2E tests         | `pnpm test:e2e`                                       |
| E2E smoke         | `pnpm test:e2e:smoke`                                 |
| E2E a11y          | `pnpm test:e2e:a11y`                                  |
| E2E stress        | `pnpm test:e2e:stress`                                |
| DB generate       | `pnpm db:generate`                                    |
| DB migrate        | `pnpm db:migrate`                                     |
| DB migrate (test) | `pnpm db:migrate:test`                                |
| DB seed           | `pnpm db:seed` (profiles: `minimal`, `stress`, `e2e`) |
| DB start          | `pnpm docker:up`                                      |
| DB reset          | `pnpm docker:reset`                                   |
| Auth schema       | `pnpm schema:auth`                                    |
| Clean all         | `pnpm clean`                                          |

## Design Workflow

The broader pipeline flows: ideas → ADRs → specs → implementation
(tracked by Trekker). Reviews can happen at any stage.

- `docs/ideas/` — rough concepts before committing to scope
- `docs/adr/` — decisions with context, alternatives, consequences
- `docs/specs/` — scoped feature designs (per-epic or per-feature)
- `docs/research/` — prior art, community findings, library behavior

During planning-phase research, save findings to `docs/research/`
so they persist across conversations. Specs are the shared reference
that keeps tasks in an epic aligned.

## Implementation Workflow

**Never commit proactively.** Complete the full loop, present a
summary, and wait for the user's explicit "commit."

For interactive bug fixes and exploratory work, steps apply as
applicable — skip planning, TDD, and polish when the user is
driving iteration. Checks (step 7) and the commit gate (step 12)
are always required.

1. **Review** — `trekker ready`, check task context, deps, blockers.
   Set task to `in_progress`.
2. **Research** — read relevant files, `docs/` (specs, ADRs,
   research), schemas. `/component-patterns` before any UI work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oakoss/finance-tracker](https://github.com/oakoss/finance-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
