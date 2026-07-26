---
trigger: always_on
description: **Analysis Date:** 2026-03-03
---

# Coding Conventions

**Analysis Date:** 2026-03-03

## Naming Patterns

**Files:**

- Use kebab-case for most modules and components in `src/actions/auth-actions.ts`, `src/lib/auth/require-auth.ts`, `src/components/organisms/login-form.tsx`, and `src/components/ui/button.tsx`.
- Use framework-conventional names for route handlers and app pages in `src/app/api/push/subscribe/route.ts`, `src/app/api/chat/route.ts`, and `src/app/layout.tsx`.
- Keep duplicate domain files aligned by suffix when parallel implementations exist (for example `src/actions/account-actions.ts` and `src/actions/accounts-actions.ts`, plus `src/actions/expense-actions.ts` and `src/actions/expenses-actions.ts`).

**Functions:**

- Use camelCase for functions and handlers (`loginAction`, `requireAuth`, `sanitizeForPrompt`, `handleDailySubmit`) in `src/actions/auth-actions.ts`, `src/lib/auth/require-auth.ts`, `src/app/api/chat/route.ts`, and `src/components/organisms/expense-form.tsx`.
- Use PascalCase only for React components (`LoginForm`, `SubmitButton`, `ExpensesClient`, `Button`) in `src/components/organisms/login-form.tsx`, `src/app/(dashboard)/expenses/client.tsx`, and `src/components/ui/button.tsx`.

**Variables:**

- Use camelCase for local variables and state (`shouldRedirect`, `initialCurrency`, `createdExpenseId`, `isSubmitting`) in `src/actions/auth-actions.ts`, `src/app/layout.tsx`, and `src/components/organisms/expense-form.tsx`.
- Use UPPER_SNAKE_CASE for module constants (`MAX_MESSAGES`, `ALLOWED_FILE_TYPES`, `MAX_FILE_SIZE`) in `src/app/api/chat/route.ts` and `src/components/organisms/expense-form.tsx`.

**Types:**

- Use PascalCase with semantic suffixes for domain and payload types (`ApiResponse`, `AuthActionState`, `ExpenseInput`, `AuthResult`) in `src/types/database.ts`, `src/actions/auth-actions.ts`, `src/lib/validations/transaction.ts`, and `src/lib/auth/require-auth.ts`.
- Co-locate inferred Zod types directly under schema definitions (`RegisterInput`, `ExpenseInput`, `DailyExpenseInput`) in `src/lib/validations/auth.ts` and `src/lib/validations/transaction.ts`.

## Code Style

**Formatting:**

- Use Prettier configured in `.prettierrc` with semicolons, double quotes, `tabWidth: 2`, `printWidth: 100`, and trailing commas (`es5`).
- Keep Tailwind class order auto-sorted through `prettier-plugin-tailwindcss` in `.prettierrc`.
- Run `prettier --write .` or `prettier --check .` from scripts in `package.json`.
- Normalize quote style to double quotes for new files to match `.prettierrc`, while accounting for existing single-quote drift in files like `src/app/api/chat/route.ts`, `src/components/organisms/expense-form.tsx`, and `src/lib/db/index.ts`.

**Linting:**

- Use ESLint flat config from `eslint.config.mjs` with `eslint-config-next/core-web-vitals`, `eslint-config-next/typescript`, and `eslint-config-prettier`.
- Run lint checks through `npm run lint` and autofix with `npm run lint:fix` from `package.json`.
- Keep generated build outputs excluded by default (`.next/**`, `out/**`, `build/**`, `next-env.d.ts`) in `eslint.config.mjs`.
- Enforce pre-commit quality gates with `lint-staged` in `package.json` triggered by `.husky/pre-commit`.

## Import Organization

**Order:**

1. Framework/runtime imports first (`next/*`, `react`, third-party SDKs) in `src/actions/auth-actions.ts`, `src/app/layout.tsx`, and `src/components/organisms/login-form.tsx`.
2. Internal alias imports (`@/...`) second in `src/actions/account-actions.ts`, `src/app/api/push/subscribe/route.ts`, and `src/components/organisms/expense-form.tsx`.
3. Type-only imports inline with domain imports (`import type ...`) as needed in `src/actions/account-actions.ts`, `src/app/layout.tsx`, and `src/app/(dashboard)/expenses/client.tsx`.

**Path Aliases:**

- Use `@/* -> ./src/*` and `@/auth -> ./auth.ts` from `tsconfig.json`.
- Prefer alias imports in app and server modules (`@/lib/db`, `@/components/ui/button`, `@/types/database`) across `src/actions/*`, `src/app/*`, and `src/components/*`.

## Error Handling

**Patterns:**

- Wrap server actions in `try/catch` and return typed failure objects instead of throwing raw errors in `src/actions/account-actions.ts` and similar files under `src/actions/`.
- Use discriminated action results (`{ success: true, data } | { success: false, error }`) from `src/types/database.ts` for action-level error transport.
- Gate auth at the top of server actions with `requireAuth()` from `src/lib/auth/require-auth.ts`; return `Unauthorized` early before DB access (pattern in `src/actions/account-actions.ts`).
- Validate untrusted input using Zod `safeParse` before business logic in `src/actions/auth-actions.ts` and `src/app/api/push/subscribe/route.ts`.
- In API routes, return explicit HTTP JSON responses with status codes (`400`, `401`, `429`, `500`) in `src/app/api/chat/route.ts` and `src/app/api/push/subscribe/route.ts`.

## Logging

**Framework:** Custom logger wrapper with console backend in `src/lib/logger.ts`.

**Patterns:**

- Use `logger.error(message, context, error)` for failed operations in server code (`src/actions/account-actions.ts`, `src/actions/auth-actions.ts`, `src/app/api/push/subscribe/route.ts`, `src/app/api/chat/route.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaronjoeldev/cashlytics-ai](https://github.com/aaronjoeldev/cashlytics-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
