---
trigger: always_on
description: - Node.js 24, TypeScript strict, npm workspaces
---

# Podverse Monorepo Rules

## Stack

- Node.js 24, TypeScript strict, npm workspaces

## Architecture

Lower tiers cannot depend on higher (see `.cursor/rules/architecture-tier-dependencies.mdc`; full table in `.llm/context/architecture.md`).

### Mobile (`apps/mobile`)

- **Tier 5 consumer** (React Native + Expo), alongside web — imports shared packages downward only; **not** on `build:packages` / `build:apps`.
- **Standalone install** outside the root npm workspace (own `package-lock.json`); see **mobile-expo-monorepo** skill.
- **iOS / Android device targets:** manual — `"iPhone 17 Pro"` / `Pixel_6_Pro_API_33`; E2E/Make —
  `"iPhone 17 Pro E2E"` / `Pixel_6_Pro_API_33_e2e`. Never bare `--device` (picker). See
  [mobile-ios-simulator](/.cursor/rules/mobile-ios-simulator.mdc).
- **Startup and sync:** nothing network-bound blocks first paint; background sync runs one job at a
  time behind a visible indicator, and failures are logged rather than shown — see
  [mobile-sync-orchestration](/.cursor/rules/mobile-sync-orchestration.mdc).
- App-specific rules: [apps/mobile/AGENTS.md](apps/mobile/AGENTS.md), contributor guide [apps/mobile/APPS-MOBILE.md](apps/mobile/APPS-MOBILE.md).
- Master plan phasing: **mobile-master-plan-phasing** skill. The master plan is split into phases under [`docs/proposals/mobile/_master-plan_/`](docs/proposals/mobile/_master-plan_/PHASES.md) — **Phase 1 is closed** (agent-led framework build-out); **Phase 2 is active** and **operator-guided** from legacy-app screenshots, see **mobile-legacy-screenshot-planning**. Mobile E2E uses Maestro/Detox — not `make e2e_*`.
- Commands from repo root: prefer composite scripts (`deps:init`, `deps:init:native`, `mobile:dev`, `mobile:ios` / `mobile:android`, `mobile:e2e:*`) over multi-step install/prebuild recipes — see **mobile-expo-monorepo**. Not `-w apps/mobile`.
- **Expo CLI:** never bare `npx expo` from repo root (pulls wrong SDK). Use **Mobile** tab + `npm --prefix apps/mobile exec -- expo …` or `npm run mobile:install` (**vscode-terminals-commands**, **mobile-expo-monorepo**).

## Code Quality

- No `any` types
- **Shared UI:** Prefer `@podverse/ui` for reusable primitives in web and management-web. When both apps overlap, implement once in `packages/ui` and **prefer the web app’s existing visual baseline** when merging styles unless a11y/product docs say otherwise (see `.cursor/rules/prefer-shared-ui-web-management.mdc`, **`reusable-components`**, and **`ui-component-promotion`** when extracting between apps).
- **Shared logic:** The same habit applies past components. While building a mobile or web feature, look for reuse in **hooks** and **plain functions** too — search for an existing implementation before writing one, and when the logic already exists on the other surface, collapse the two instead of adding a third copy. See **`reuse-beyond-components`**.
- DTOs from `@podverse/helpers`
- Follow `tsconfig.base.json`
- Use ESM formatting: **Tier A** uses `.js` specifiers on relative TS imports (NodeNext); **Tier B** (`apps/web/src`, `apps/management-web/src`) stays extensionless until Turbopack parity — [docs/development/tooling/DOCS-DEVELOPMENT-TOOLING-IMPORT-SPECIFIERS.md](docs/development/tooling/DOCS-DEVELOPMENT-TOOLING-IMPORT-SPECIFIERS.md). Prefer `"type": "module"` where applicable.
- Prefer `import type` for type-only imports instead of value imports
- **Strict equality**: Use `===` and `!==` only (no `==` or `!=`). For "not null or undefined" use `x !== null && x !== undefined` (or optional chaining / truthiness where appropriate).
- **Avoid type assertions (`as`)**: Prefer proper types, optional chaining, type guards, or narrowing so the type system enforces correctness. Use `as` only when there is no better way (e.g. necessary escape hatch); keep such use minimal and documented.
- **Named exports**: Prefer named `export` in TypeScript modules; avoid `export default` when a named export works. Framework-required defaults (e.g. Next.js `page.tsx`) are the exception. See `.cursor/skills/prefer-named-exports/SKILL.md`.
- **CSS `var()`:** Do not use fallback arguments in SCSS/CSS (`var(--token, …)`); see `.cursor/rules/css-custom-properties-no-var-fallbacks.mdc`.
- **Comments are future-forward**: Describe the code as it stands, never what it used to be, what was removed, or which plan/detail/step number asked for it — plans get archived and deleted, leaving a dead reference. Clean these up in files you touch; see `.cursor/rules/comments-future-forward.mdc`.

## Commands / Terminal

- **Commands from repo root**: When giving terminal or npm commands, always give them **relative to the monorepo root**. Use `npm run <script> -w apps/<app> -- [args]` instead of instructing users to `cd apps/workers` (or similar) first. **Root-only scripts** (`test:unit`, `build:packages`, `lint`, …) must not be scoped with `-w`; scoped unit tests use `npm run test -w <workspace>`. See `.cursor/rules/commands-from-monorepo-root.mdc`.
- **Copy button**: Always put runnable commands in a **fenced code block** (e.g. `bash ... `) so the IDE shows a copy button. Do not give only inline commands when the user may want to run them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
