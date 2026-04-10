---
trigger: always_on
description: Comprehensive React component and application standards
---


# React Component & Application Standards (Bulletproof‑React Enhanced)

> **Purpose**  Unify front‑end architecture, state, logging, testing, and DX across every app in `/apps/*`, combining our existing guide with proven *Bulletproof React* principles.

---

## 1  Project Layout & Dependency Flow

```text
src/
  app/           # React Router entry (Vite CSR), global providers
  features/      # Self‑contained domain slices (Auth, Billing…)
  components/    # Re‑usable UI primitives (Button, Card…)
  hooks/         # Shared hooks not tied to a feature
  lib/           # Framework‑agnostic utilities (date, math…)
  types/         # Global TypeScript contracts
```

* **Unidirectional flow**  `components → features → app` only (enforced via ESLint `import/no-restricted-paths`).
* **Absolute imports**  Configure `@/*` alias in `tsconfig.json` to eliminate `../../../`.

## 2  Code Quality Guard‑Rails

| Tool                                                 | Purpose                  | Invocation                    |
| ---------------------------------------------------- | ------------------------ | ----------------------------- |
| TypeScript `--strict`                                | Static safety            | `pnpm tsc -p tsconfig.json`   |
| ESLint (+ plugin\:import, plugin:@typescript-eslint) | Style & dependency rules | `pnpm lint`                   |
| Prettier                                             | Auto‑formatting          | `pnpm format`                 |
| Husky + lint‑staged                                  | Pre‑commit gate          | runs `lint`, `format`, `test` |

CI workflow → *fail fast* on lint, type, unit, integration, e2e.

## 3  State Management – Four Buckets

1. **Local component state**   `useState`, `useReducer`.
2. **UI / app state**          Context, Zustand, or Redux Toolkit (keep thin slices).
3. **Server‑cache state**      React Query (preferred) or SWR.
4. **Form state**              React‑Hook‑Form.

> ✱ *Rule:* Never mix buckets—e.g. don’t put form fields in global UI state.

## 4  API & Data Layer

* All HTTP calls live in `lib/api/` with a single axios instance.
* Generate typed hooks via React Query (`useGetUser`). UI remains declarative.
* Centralised error mapping → user‑friendly toasters, log raw error once.

## 5  UI Component Guidelines

### 5.0 Core Principles

* **Co‑location**  `Button.tsx`, `Button.test.tsx`, `Button.stories.tsx` live together.
* **Props first**   Expose minimal surface; favour composition over prop drilling.
* **Styled‑components + Mantine**    Theme tokens only; ban inline hex colours.

### 5.1 File Naming & Barrel Exports

| Artifact      | Convention               | Example                       |
| ------------- | ------------------------ | ----------------------------- |
| Component     | `PascalCase.tsx`         | `UserCard.tsx`                |
| Styles        | `PascalCase.styles.ts`   | `UserCard.styles.ts`          |
| Tests         | `PascalCase.test.tsx`    | `UserCard.test.tsx`           |
| Stories       | `PascalCase.stories.tsx` | `UserCard.stories.tsx`        |
| Hook          | `useCamelCase.ts`        | `useAuth.ts`                  |
| Folder barrel | `index.ts` re‑export     | `export * from './UserCard';` |

*Rules*

* Prefer **named exports** everywhere; avoids refactor pain and improves tree‑shaking.
* Keep a single React component per file unless trivial variations (e.g., sub‑components).
* Never import from a sibling file’s relative path; use the folder barrel + absolute alias (`@/features/auth`).

### 5.2 Shared Components & Atomic Design

> We blend *Atomic Design* with our **components → features → app** dependency flow by treating the `/components` layer as an atomic toolbox.

```
src/components/
  atoms/        # Lowest‑level primitives – Button, Text, Icon
  molecules/    # Groupings of atoms – AvatarWithName, InputWithLabel
  organisms/    # Complex, reusable blocks – Header, Sidebar, DataTable
  layouts/      # Page‑level wrappers – DashboardLayout, AuthLayout
```

* **Dependencies**

  * Atoms 🚫 cannot import molecules or organisms.
  * Molecules may import atoms.
  * Organisms may import atoms and molecules.
  * Layouts may import organisms, molecules, atoms but **never features**.
* **Stories**  Every atom/molecule/organism gets a Storybook story; organisms/layouts also get Playwright interaction tests.
* **Theming**  All atoms rely on Mantine’s theme + styled‑components; higher layers compose without overriding base styles.
* **Promotion rule**  If a feature‑level component is reused twice outside its feature, promote it up the atomic ladder (molecule → organism as complexity grows).

---

## 6  Logging & Observability  (@kit/logger / Pino 8)  Logging & Observability  (@kit/logger / Pino 8)

### Levels & Usage

| Level           | Use‑case                                           |
| --------------- | -------------------------------------------------- |
| `trace`         | Opt‑in request/response body, perf micro‑spans     |
| `debug`         | Branch decisions, config dumps                     |
| `info`          | Lifecycles: server start, page mount, route change |
| `warn`          | Recoverable issues, slow queries (> 500 ms)        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drumnation) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
