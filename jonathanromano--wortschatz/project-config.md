---
trigger: always_on
description: Operational notes for AI assistants working in this repo.
---

# CLAUDE.md

Operational notes for AI assistants working in this repo.

## Boundary: which surface gets the new code

**Read this before adding any route, endpoint, or server action.** Two HTTP
surfaces exist and the split is a hard rule, not a preference — full details
in [ARCHITECTURE.md](./ARCHITECTURE.md), deployment context in
[MONOREPO.md](./MONOREPO.md#api-boundary-rules).

- **Express (`apps/api`)** owns anything that **calls an LLM provider**, takes
  **> 5 seconds**, **processes binary data** (images/audio/PDF), or is a
  background job. All three AI endpoints live here: `/ai/review-text`,
  `/ai/evaluate-answer`, `/ai/generate-exercise`.
- **Next.js (`apps/web`)** owns auth/sessions, lightweight DB CRUD, read-only
  queries, session-bound server actions, and UI.
- **No file under `apps/web/src/` may import `@anthropic-ai/sdk` or `openai`.**
  CLI scripts under `apps/web/scripts/` may (offline-fallback generators
  only). A static test (`apps/web/src/__tests__/architecture.test.ts`) fails
  the build if this is violated.
- When Next.js needs an LLM result it calls Express via
  `src/lib/api-client.ts`. Schemas/logic both tiers need live in a shared
  package (`@wortschatz/exercises`), never in `apps/web/src/`.

## Stack

- **pnpm workspaces + Turborepo 2.x** monorepo
- **`apps/web`** — Next.js 15 (App Router, React 19 RC) + MUI v9 + emotion
- **`apps/api`** — Express 4 + helmet (ESM, runs via `tsx`)
- Shared packages: **`@wortschatz/{database,types,config}`**
- Tailwind CSS v4 — layout-utility classes only (see coexistence rule)
- next-intl for i18n (`en`, `pt`, `tr`, `uk`)
- Prisma + PostgreSQL, NextAuth.js v5
- Fonts: `next/font/google` → **Fraunces** (display) + **Inter** (body)
- vitest + jsdom + `@testing-library/react` for tests

All Next.js routes nest under `apps/web/src/app/[locale]/…`. The root
layout is minimal — locale HTML lives in
`apps/web/src/app/[locale]/layout.tsx`.

When this file refers to a path beginning with `src/…` (e.g.
`src/theme/`), read it as `apps/web/src/…` — the rules predate the
Sprint 03 monorepo split. See [MONOREPO.md](./MONOREPO.md) for the full
layout, deployment story, and current limitations.

## Monorepo conventions

- **Prefer the package barrel over the in-tree alias.** Use
  `import { prisma } from "@wortschatz/database"` (not `@/lib/db`),
  `import { MUENZEN_REWARDS, pickLocalized } from "@wortschatz/config"`
  (not `@/config/limits` or `@/lib/exercises/i18n`), and
  `import type { LocalizedText } from "@wortschatz/types"`. The old
  in-tree modules were deleted in Sprint 03; their re-imports won't
  resolve. Prisma enums are re-exported from `@wortschatz/database`,
  so never import directly from `@prisma/client`.
- **Scripts run from the repo root.** `pnpm dev`, `pnpm build`,
  `pnpm test`, `pnpm typecheck`, `pnpm db:generate`, `pnpm db:migrate`.
  Target a single workspace with
  `pnpm --filter @wortschatz/<name> run <script>`.
- **Shared values belong in `@wortschatz/config`** when both apps
  need them (Münzen rules, rate limits, locales, env schemas). Pure
  TS types belong in `@wortschatz/types`. Prisma schema, migrations,
  and the client singleton belong in `@wortschatz/database`. Don't
  duplicate.
- **One language per layer.** `apps/web` writes React + Next + MUI;
  `apps/api` writes Node + Express + zod. They share types but never
  cross-import (e.g. apps/api must never `import "@/something"`).

## Palette System + Material UI

All color, typography, radius, shadow, and shape tokens live in
`src/theme/`. Pages and components receive the theme via the
`<AppThemeProvider>` in `src/app/[locale]/layout.tsx`, which calls
`createAppTheme(mode)` from `@/theme`. **Never hardcode hex anywhere
outside `src/theme/`.**

- **Palettes.** `src/theme/palette.ts` defines both `lightPalette` and
  `darkPalette`. Mode selection is wired (see "Color modes" below).
- **Typography.** Headings use Fraunces via CSS var `--font-fraunces`,
  body uses Inter via `--font-inter`. MUI's `<Typography>` is the only
  way to set type — do not use Tailwind `font-*` classes.
- **Custom palette keys** (already augmented in `src/theme/augmentation.ts`):
  `tertiary`, `accentSoft`, `successSoft`, `dangerSoft`, `surfaceAlt`.
  Use them via `theme.palette.X` or MUI's `color="X"` where supported.
- **Hover behavior.** Buttons lift with `translateY(-1px)` + shadow on
  hover via the `MuiButton` style override in `src/theme/index.ts`. Never
  use `hover:opacity-90`.
- **Shape.** Inputs/buttons use the small radius from `shape`; cards use
  `RADIUS_CARD`; pills/chips use `borderRadius: 999` via MUI defaults.

### Color modes

- The user picks `'light' | 'dark'`. Default is `'light'` on first
  visit; persisted in `localStorage` under `wortschatz:color-mode`
  (exported as `COLOR_MODE_STORAGE_KEY` from
  `src/theme/ColorModeContext.tsx`). The pre-Sprint-04 `'system'`
  option was removed; old stored values map to `'light'` on read.
- Use `useColorMode()` from `@/hooks/useColorMode` to read or change
  the mode. It returns `{ mode, setMode, toggle }`. Don't access
  `localStorage` directly.
- **Hydration contract** (load-bearing — do not loosen):
  - The locale layout injects a blocking inline script in `<head>`
    that reads `localStorage` and writes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JonathanRomano/wortschatz](https://github.com/JonathanRomano/wortschatz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
