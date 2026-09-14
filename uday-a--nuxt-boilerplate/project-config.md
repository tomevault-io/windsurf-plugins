---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project skills

This repo ships agent skills at `.claude/skills/` that enforce the boilerplate's conventions. They activate automatically based on the trigger phrases in their `description:` frontmatter — invoke them when the trigger matches:

- **`uipkge-first`** — before adding any UI primitive. Routes to `npx shadcn-vue add @uipkge/<name>` over hand-rolling.
- **`add-page`** — when creating any `app/pages/**/*.vue`. Picks layout, middleware, useHead correctly.
- **`auth-gating-check`** — when touching `server/api/**`, `server/routes/**`, `app/pages/**`, or `app/middleware/**`. Verifies auth posture.
- **`secret-exposure-check`** — before commit when env / runtimeConfig / logs changed. Catches the four canonical Nuxt leak vectors.
- **`logger-conventions`** — any log statement in `server/**`. Enforces dot-namespaced event names, structured fields, no PII.
- **`response-envelope`** — any new/modified `server/api/**` handler. Forces `apiHandler` + `ok()` / `apiError()` over raw responses.
- **`error-handling`** — any try/catch, throw, or rethrow. Enforces typed errors, no silent swallow, no leaked stack traces.
- **`i18n-keys`** — any `t('…')` / `$t('…')` call added/modified. Enforces key parity across en.json and es.json.
- **`db-migration`** — when `server/db/schema.ts` changes. Forces `drizzle-kit generate` and surfaces destructive ops.
- **`shipping-check`** — when the user signals "done" / "ready to commit". Runs lint + typecheck + knip + jscpd + boundary check.

Plus three external skills pulled from [skills.sh](https://skills.sh) and pinned in `skills-lock.json`:

- **`nuxt`** — `antfu/skills@nuxt`. Generated from the Nuxt docs.
- **`vue`** — `antfu/skills@vue`. Vue 3 Composition API reference.
- **`reka-ui`** — `onmax/nuxt-skills@reka-ui`. Headless Vue primitives that shadcn-vue is built on.

See `.claude/skills/README.md` for the rationale and how the skills relate to the tool-side enforcement (lefthook, commitlint, knip, jscpd, zod env).

## Commands

- `npm run dev` — Nuxt dev server on http://localhost:3000 (reuse if already running; do not kill).
- `npm run build` — production build.
- `npm run preview` — preview production build locally.
- `npm run generate` — static generate.
- `npm run lint` / `npm run lint:fix` — ESLint flat config via `@nuxt/eslint`.
- `npm run typecheck` — `vue-tsc --noEmit` via `nuxi typecheck`.
- `npm run knip` — unused files/exports/deps.
- `npm run duplicates` — `jscpd` copy-paste detection. Threshold 2.5%.
- `npm install` runs `nuxt prepare` + `lefthook install` via postinstall.
- Git hooks (lefthook): pre-commit runs `eslint --fix` on staged files; commit-msg runs commitlint (Conventional Commits).
- Drizzle (no npm-script wrappers; invoke directly):
  - `npx drizzle-kit generate` — emit SQL into `server/db/migrations/`.
  - `npx drizzle-kit migrate` — apply migrations against `DATABASE_URL`.
  - `npx drizzle-kit studio` — local schema browser.

## Architecture

**Nuxt 4** with the `app/` directory split. `compatibilityDate: 2025-07-15`. TypeScript via project references — root `tsconfig.json` points at the four generated `.nuxt/tsconfig.*.json` projects (app, server, shared, node). Do not put `compilerOptions` in the root `tsconfig.json`.

### Frontend (`app/`)

- `app/components/` is registered with `pathPrefix: false` — component filenames map directly to PascalCase auto-imports with no directory prefix (e.g. `blocks/AuthSignIn.vue` → `<AuthSignIn />`, not `<BlocksAuthSignIn />`). Rename collisions accordingly.
- Three component buckets:
  - `components/ui/` — shadcn-vue (style `new-york`, base color `neutral`) consumed via the `@uipkge` registry declared in `components.json`. Add components with `npx shadcn-vue add @uipkge/<name>`. Each ui dir exports via `index.ts`.
  - `components/blocks/` — composed page sections (auth forms, dashboard layout, sidebar, hero/cta/pricing variants, kanban, command palette, etc.).
  - `components/kanban/` — kanban-specific pieces.
- `app/layouts/dashboard.vue` wraps content in `<DashboardLayout>` and derives breadcrumbs from `route.path`. It reads `useUserSession()` and dispatches profile-menu / command-palette intents to the router.
- `app/pages/` — file-based routing. Top-level public: `index.vue`, `pricing.vue`, `terms.vue`, `privacy.vue`, auth entry pages (`login`, `sign-up`, `forgot-password`, `mfa`). Top-level authenticated: `onboarding/`, `invite/[token].vue`. Nested under `definePageMeta({ middleware: 'auth' })`: `dashboard/`, `settings/`, `projects/`, `feedback/`, `support/`.
- `app/middleware/auth.ts` is a **page-level** middleware (opt in via `definePageMeta({ middleware: 'auth' })`). Enforces real auth: unauthenticated requests redirect to `/login?next=…`. Demo mode (see graceful-degradation matrix below) keeps the boilerplate clickable without configuring OAuth.
- `app/lib/utils.ts` exports `cn()` (clsx + tailwind-merge) — the only shared frontend util.
- `app/composables/`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uday-a/nuxt-boilerplate](https://github.com/uday-a/nuxt-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
