---
trigger: always_on
description: - Write all code, comments, commit messages, and documentation in English.
---

# Agent Guide

## Project Rules

- Write all code, comments, commit messages, and documentation in English.
- Use Bun for dependency management and scripts. Keep `exact = true` in both Bun config files.
- Keep the app TypeScript-first and preserve strict compiler settings.
- Use shadcn-style local UI components in `src/components/ui`; primitives must come from Base UI.
- Use Zod for form, domain, and Evolu schema validation.
- Store persistent application data through Evolu. Avoid direct `localStorage` except for non-critical UI preferences such as language.
- Use Biome for linting and formatting.
- Do not create or use `index.ts` barrel files for re-exporting. Import directly from the owning module file.
- For asynchronous reads from remote or native APIs in React, use TanStack Query's `useQuery` rather than `useEffect` with local state. Use a stable `queryKey` and `enabled` for runtime or input preconditions; keep Evolu subscriptions on `useEvoluQuery`.

## Project Structure

- `src/main.tsx` is the browser entry point. It installs polyfills and renders the React app.
- `src/App.tsx` wires top-level providers (Jotai store, theme, background jobs, toaster) and the TanStack Router provider.
- `src/router.tsx` creates the TanStack Router from `src/routeTree.gen.ts`; route files live in `src/routes`.
- `src/routes/__root.tsx` defines the root layout and error boundary; `src/routes/_terminal.tsx` is the layout route for the terminal pages. Keep route files thin and move substantial page UI into page or feature modules.
- `src/atoms` contains Jotai atoms that bootstrap app singletons: the device Evolu client, the app Evolu client, the active account, console, and run. Evolu clients are created here, not in `main.tsx`.
- `src/hooks` contains the React bindings for those singletons (`useEvolu`, `useEvoluQuery`, `useDeviceEvoluQuery`, `useConsole`, `useTranslation`, `useAppRun`, ...). Access Evolu from React through these hooks.
- `src/features` contains feature modules: page-level UI (forms, hooks, presentational components) for one feature, composed from domain modules and `src/components/ui` primitives. `src/features/settings` is the first tenant. Substantial page UI extracted from routes belongs here, not in `src/routes` or `src/components`.
- `src/components/ui` contains shadcn-style reusable UI primitives built on Base UI, such as `button.tsx`. Keep generic UI here; avoid feature or domain logic in this directory.
- `src/components/theme-provider.tsx` contains theme-level UI infrastructure.
- `src/core/evolu` contains Evolu client setup, the app schema composition, and the device database (`device-client.ts`, `device-account.ts`). Register new Evolu tables and indexes in `src/core/evolu/schema.ts`.
- `src/core/modules` contains domain modules. Each module owns its schema, branded ids/types, actions, queries, and tests for one domain concept.
- `src/core/modules/shared` contains lower-level domain helpers, shared schemas, Evolu dependency helpers, and the `getFirstOr` Result helper.
- `src/core/deps.ts` declares small injectable dependency objects (`FetchDep`, `DateDep`, `EvoluOwnerIdDep`); `src/core/error.ts` provides the `defineError` factory.
- `src/core/background-jobs` contains the background job framework (`BackgroundJobContext`, keyed task queue) and the sync jobs under `jobs/`. Jobs receive all effects — including `lockManager` — through their context; never use ambient globals such as `navigator.locks`.
- `src/core/integrations` contains HTTP clients for external services (FIO, Yadio, LNURL). Clients follow the fio convention: a `createXApiDep` factory, HTTP through `appFetchAsJson` from `src/core/deps.ts`, zod-validated responses, and `defineError` errors carrying `status` and `responseBody`.
- `src/core/spark` wraps the Spark wallet SDK behind `SparkWalletDep`. Do not call `SparkWallet.initialize`/`getOrCreateWallet` directly outside this wrapper; extend the wrapper when a consumer needs more of the SDK surface. Instances are shared per mnemonic and ref-counted (via the internal `createRefCountedResourcePool` in `src/lib/ref-counted-resource-pool.ts`, exposed through `createSharedSparkSyncWallet` and `createDefaultSparkPaymentWallet`): each caller still owns cleanup of its own `sparkWallet.create()` result exactly like any other disposable, but the underlying SDK instance is only actually torn down once every concurrent holder — including the Spark account sync job's long-held reference — has released it, so a warm instance survives back-to-back calls for the same account.
- `src/core/cli` contains CLI-runtime helpers (`cli-env.ts`, the in-process lock manager); CLI entry points live in `bin/`.
- `src/core/native` contains Capacitor/WebView runtime detection and platform plumbing.
- `src/i18n` contains translation resources and the translation hook. `src/i18n/en.ts` is the source of truth for translation keys; `cs.ts` and `sk.ts` must cover every key via `satisfies Record<TranslationKey, string>`, and `resources.ts` only composes the languages.
- `src/lib` contains app-level generic utilities such as `cn`; keep domain code in `src/core/modules` instead.
- `src/assets` contains static frontend assets.
- `src/index.css` contains global Tailwind and theme styles.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [finitoapp/payky](https://github.com/finitoapp/payky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
