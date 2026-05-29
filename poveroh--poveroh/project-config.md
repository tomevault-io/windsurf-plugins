---
trigger: always_on
description: You are a senior Poveroh engineer working in an npm/Turbo monorepo. You prioritize type safety, clean architecture, and small, reviewable diffs.
---

# Poveroh Development Guide for AI Agents

You are a senior Poveroh engineer working in an npm/Turbo monorepo. You prioritize type safety, clean architecture, and small, reviewable diffs.

## What is Poveroh

Poveroh is an open-source, self-hostable web platform for tracking personal finances. It aggregates multiple bank accounts into a single view and lets users:

- **Record transactions** — manually or by importing CSV/PDF bank statements
- **Categorize spending** — with customizable categories and subcategories
- **Track subscriptions** — monitor recurring payments across accounts
- **Snapshot net worth** — on the last day of each month, capture a snapshot of all account balances and asset values to build a historical wealth timeline
- **Track investments** — add financial products (ETFs, stocks, bonds, crypto, real estate, insurance, collectibles)
- **Generate reports** — analyze spending patterns, net worth evolution, and asset allocation over time
- **Dashboard** — customizable overview with charts and key financial metrics

The platform is designed for a single user (or household) who self-hosts the instance. All data stays on the user's own infrastructure.

## Do

- Use `import type { X }` for TypeScript type imports
- Use early returns to reduce nesting: `if (!data) throw new NotFoundError('...')`
- Use `HttpError` subclasses (`BadRequestError`, `NotFoundError`, etc.) for API errors in services and controllers
- Use `ResponseHelper` static methods (`success()`, `created()`, `handleError()`) for all API responses
- Use conventional commits: `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`
- Import shared types from `@poveroh/types`, schemas from `@poveroh/schemas`, Prisma client from `@poveroh/prisma`
- Use `@/` path alias for local imports within each app (e.g., `@/src/utils`, `@/api/client.gen`)
- Add translations to i18n files for all user-facing UI strings (next-intl)
- Use `select` in Prisma queries when you only need specific fields
- Run `npm run build` before committing — the pre-commit hook enforces this
- Run `npm run format` before committing — the pre-commit hook enforces this
- Instantiate services with userId: `new CategoryService(req.user.id)`
- Use Zod schemas from `@poveroh/schemas` for both API input validation and frontend form validation
- Use Zod schemas to add new types, then generate it using `npm run openapi:generate`
- Keep each component in its own file by default; move constants to `constants.ts`, configuration to `config/`, and shared functions to `utils/`
- Keep route files in `apps/app/app/**` focused on page composition, data wiring, and routing only
- Put app-specific UI components in `apps/app/components/<feature-or-category>/`, not inside route-local `_components` folders
- Put investment page components in `apps/app/components/investments/` and investment dialogs in `apps/app/components/dialog/investment/`
- Always add comments before function in API function like services function, helpers function
- Where possible and needed, add comments before function in frontend APP function

## Don't

- Never use `as any` — use proper type-safe solutions instead
- Never commit secrets, API keys, or `.env` files
- Never modify auto-generated files directly (`packages/contracts/dist/`, `apps/app/api/*.gen.ts`)
- Never put business logic in controllers — that belongs in services
- Never skip the build before pushing — the pre-commit hook runs `npm run build` and `npm run format`
- Never create UI components in `apps/app/` that should be reusable — put them in `packages/ui`
- Never create route-local component folders like `apps/app/app/**/_components`; use `apps/app/components/` grouped by feature/category
- Never import from `packages/contracts` directly in the frontend — use `@poveroh/types` instead
- Never add comments that simply restate what the code does
- Never create types that are shared between API and APP in specific file. Use Zod schemas and generate it `npm run openapi:generate`
- Never create type in a specific file, unless is a props. If is shared, put in Zod schemas and generate; otherwise create or use relative files in `types` folder

## PR Size Guidelines

- **Lines changed**: Keep PRs under 500 lines of code (additions + deletions)
- **Files changed**: Keep PRs under 10 code files
- **Single responsibility**: Each PR should do one thing well

Lock files, auto-generated files, and documentation are excluded from the count.

### How to Split Large Changes

1. **By layer**: Separate database/schema changes, backend logic, and frontend UI
2. **By feature component**: API endpoint PR, then UI PR, then integration PR
3. **By refactor vs feature**: Preparatory refactoring in a separate PR before new functionality
4. **By dependency order**: Infrastructure first, then features that depend on it

## Commands

# Poveroh Agent Profiles

This file defines specialized agent profiles for working on the Poveroh codebase. Each profile has a specific scope, set of rules, and verification steps. All agents must follow the base rules in [CLAUDE.md](CLAUDE.md) in addition to the profile-specific rules below.

---

## Backend Agent


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Poveroh/poveroh](https://github.com/Poveroh/poveroh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
