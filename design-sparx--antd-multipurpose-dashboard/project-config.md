---
trigger: always_on
description: Single-package React + Vite + Ant Design 6 dashboard app. No monorepo.
---

# AGENTS.md

Single-package React + Vite + Ant Design 6 dashboard app. No monorepo.

## Commands

```bash
pnpm dev                # Vite dev server on http://localhost:5173
pnpm build              # tsc && vite build (fails on TS errors)
pnpm lint               # ESLint --max-warnings 0 (zero warnings required)
pnpm prettier:write     # Prettier format all files
pnpm storybook          # Storybook on port 6006
pnpm build-storybook    # Static Storybook build
pnpm preview            # Preview production build
```

Antd-specific tooling (`@ant-design/cli`, used in CI / manually):

```bash
antd lint src --format json          # JSX/TSX usage linter (deprecated / a11y / usage / performance)
antd doctor                         # ecosystem compat check
antd info <Component> --format json # offline component API lookup
```

## Pre-commit / CI

- **pre-commit**: `pnpm exec lint-staged` (Prettier + ESLint `--fix` on staged files)
- **commit-msg**: `pnpm exec commitlint --edit $1` — conventional commits enforced (`type(scope): message`)
- **CI**: Node 24, `pnpm install --frozen-lockfile` (lockfileVersion 9), Chromatic on push/PR to `main`, Changesets release on push to `main`. The dedicated CI build+lint workflow (`ci.yml`) was removed to save action minutes — validate locally with `pnpm build` / `pnpm lint` too.
- **pnpm version** is pinned in `package.json#packageManager` (currently `pnpm@9.15.4`). Do **not** also pass `version:` to `pnpm/action-setup@v4` — the action errors with `Multiple versions of pnpm specified`. The `packageManager` field is the single source of truth (resolved by corepack).
- **`pnpm-workspace.yaml`** is required: the `pnpm` cache step in `actions/setup-node@v4` calls `pnpm store path`, which in pnpm 9 requires a non-empty `packages:` field. This is a single-package repo (not a monorepo) so the workspace contains just the root (`packages: ['.']`).
- Lint must pass with **0 warnings** before commit.
- **Lint currently fails with pre-existing errors** (`@typescript-eslint/no-explicit-any`, `ban-ts-comment`) — `pnpm lint` exits non-zero and blocks the pre-commit `eslint --fix` step on affected files. **Workaround**: use `git commit --no-verify` for changes that don't introduce new lint errors. Fix the underlying `any` usages as a separate task.

## Data & Auth Quirks

- **Auth is fully dummy/mock.** `src/services/auth/authService.ts` returns hardcoded demo data. No real backend auth flow works. The login flow is exposed via `<LoginModal>` in `src/components/auth/login-modal/` and triggered by the `open-login-modal` window event.
- **`useAuth()` is the single source of truth** for auth state (re-exported from `src/hooks`). It exposes `{ user, isLoading, login, logout, error }` and a local `useState<string | null>`-based error is used in the login modal (`setError(null)` / `setError(err.message)`). There is **no** `dispatch(clearError())` — `authSlice` was removed in batch 3 (#176).
- **Mock data is locked on — no live API path works.** `src/config/api.config.ts` hardcodes `USE_MOCK_DATA: true`; `src/redux/data-mode/dataModeSlice.ts` locks `useMockData` to `true` (the `toggleDataMode`/`setDataMode`/`enableRealData` actions are no-ops that re-lock to `true`). Env vars `VITE_USE_MOCK` / `VITE_USE_MOCK_DATA` (present in `.env` / `.env.example`) are **not** read by any code. Mock JSON lives in `public/mocks/`.
- **API client** (`src/services/api/apiClient.ts`) routes requests to mock files or live API based on Redux `dataMode.useMockData` (always `true` in practice). Mock-only endpoints (e.g., notifications) always use mock data.
- **Dual auth state**: Auth exists in both Redux (`src/redux/auth/authSlice.ts`, dummy) and Context (`src/contexts/AuthContext.tsx`, the source of truth). `ProtectedRoute` guards via `useAuth()`. Always use `useAuth()` (re-exported from `src/hooks`) — never read `authSlice` directly for auth checks.

## Architecture Notes

- **Routing**: `src/routes/routes.tsx` uses `createBrowserRouter`. `ProtectedRoute` guards auth. `PageWrapper` HOC wraps routes for scroll restoration.
- **Breadcrumbs**: Rendered once, in the app header (`HeaderNav` in `src/layouts/app/app.tsx`), auto-generated from `location.pathname` (home icon + capitalized path segments). `PageHeader` no longer renders breadcrumbs — do **not** pass a `breadcrumbs` prop to it; page-level breadcrumbs were removed in favor of the single header breadcrumb.
- **Redux**: `theme`, `dataMode`, `auth`, `designStyle` slices. All persisted via `redux-persist` to `localStorage`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [design-sparx/antd-multipurpose-dashboard](https://github.com/design-sparx/antd-multipurpose-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
