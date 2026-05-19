---
trigger: always_on
description: - Core application lives under `app/`, with App Router layouts in `app/(auth)/`, `app/(dashboard)/`.
---

# Repository Guidelines

## Project Structure & Module Organization

- Core application lives under `app/`, with App Router layouts in `app/(auth)/`, `app/(dashboard)/`.
- Supporting UI atoms live in `components/`; shared hooks in `hooks/`, shared contexts in `contexts/`.
- Configuration lives in `next.config.ts`, `app.config.ts` (if present), and `config/`.
- Shared utilities and lib code are in `lib/`; type definitions in `types/`.
- i18n locale files live under `i18n/locales/` (structure must match the old project).
- Static assets belong in `public/` or `assets/`.
- Tests belong in `tests/` (mirror source structure when tests exist).
- **UI vs feedback**: `components/ui/` holds presentational, declarative UI primitives (e.g. Button, Dialog). `lib/feedback/` holds global imperative APIs for toast and confirm dialogs (MessageProvider/useMessage, DialogProvider/useDialog). Use `@/lib/feedback/message` and `@/lib/feedback/dialog` for imperative feedback; use `@/components/ui/*` for declarative UI.

---

## Build, Test, and Development Commands

- **Node version requirement**: Before running `pnpm` commands (especially checks/tests), run `nvm use v22` in this repository.

- `pnpm dev` – start the Next.js development server with hot reload.
- `pnpm build` – create a production build.
- `pnpm start` – run the production bundle locally.
- `pnpm lint` – run ESLint.
- `pnpm test:run` – run the test suite (when configured).
- `pnpm tsc --noEmit` – perform a strict TypeScript type check (or rely on `next build` for type-checking).

---

## Mandatory Code Quality Checks

**⚠️ CRITICAL: These checks MUST pass before every commit.**

Before committing any code changes, you MUST run and pass:

1. **Lockfile Sync Check**: `pnpm install --frozen-lockfile`
   - Ensures `pnpm-lock.yaml` is in sync with `package.json`
   - **MUST run `pnpm install` after modifying `package.json` and commit the updated `pnpm-lock.yaml`**
   - CI will fail if the lockfile is out of sync.

2. **TypeScript Type Check**: `pnpm tsc --noEmit` (or `pnpm build`)
   - Ensures all TypeScript types are correct.
   - Must have zero errors before committing.

3. **Lint Check**: `pnpm lint`
   - Ensures code follows ESLint rules.
   - Fix issues before committing.

4. **Format Check** (if Prettier is configured): `pnpm prettier --check .`
   - Ensures consistent formatting.
   - If it fails, run `pnpm lint:fix` or `pnpm format` (when available) to auto-fix.

5. **Test Coverage Check** (when tests exist): Review and update tests for code changes
   - **MUST review test cases** when modifying code: add tests for new features, update tests for changed behavior, remove tests for removed features.
   - Run `pnpm test:run` to ensure all tests pass.
   - Ensure test cases accurately reflect the current implementation.

**Automated Enforcement**: If a pre-commit hook exists, it will run these checks. If any check fails, the commit will be blocked.

**Quick Fix**: If checks fail:

1. Run `pnpm install` to sync lockfile (if `package.json` changed).
2. Fix ESLint/Prettier issues.
3. Address TypeScript errors manually.
4. Review and update test cases as needed, then run `pnpm test:run` to verify.

---

## Coding Style & Naming Conventions

- Use Prettier defaults when configured; run `pnpm lint:fix` or `pnpm format` after making changes.
- React components use functional components with TypeScript; prefer hooks and custom hooks for shared logic.
- Component files use **kebab-case** (e.g. `bucket-selector.tsx`); reference them with **PascalCase** in JSX (e.g. `<BucketSelector />`).
- Override shadcn primitives **outside** `components/ui/`; never edit files in that directory directly.
- Render tabular data with the shared `DataTable` + `useDataTable` utilities unless a specific requirement makes them unsuitable.
- Language pack files must follow the structure used in the old project; do not alter i18n layout or keys arbitrarily.

### Component structure and naming

- **Directories**: Group by **domain/feature**; use plural for domain folders (e.g. `buckets/`, `user/`, `object/`).
- **File names**: kebab-case; **do not repeat the directory name** in the filename (e.g. under `buckets/` use `info.tsx`, `new-form.tsx`, `selector.tsx` instead of `bucket-info.tsx`, `bucket-new-form.tsx`). The path already provides context.
- **Component names**: PascalCase, aligned with the domain and purpose (e.g. `BucketInfo`, `UserDropdown`); component names may still include the domain when used in JSX for clarity.
- **Forms**: Use consistent patterns per domain: `XxxNewForm` / `XxxEditForm` or `XxxForm`; files can be `new-form.tsx`, `edit-form.tsx`, `form.tsx` under the domain folder.
- **Placement**: Components used only by one domain live in that domain folder; components reused by 3+ different domain pages may stay at root or under `components/shared/` (document if so).

---

## Testing Guidelines

- When tests are configured, add new suites under `tests/`, mirroring source structure.
- Name files `*.spec.ts` or `*.test.ts`.
- Keep tests deterministic; mock network calls through provided hooks or context.
- **⚠️ CRITICAL: Every code change MUST include corresponding test updates** when tests exist:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustfs/console](https://github.com/rustfs/console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
