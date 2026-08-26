---
trigger: always_on
description: Grafana panel plugin for inserting/updating data and modifying configuration via forms.
---

# AGENTS.md — Business Forms Plugin (volkovlabs-form-panel)

Grafana panel plugin for inserting/updating data and modifying configuration via forms.
Frontend-only plugin (no Go backend). Node >= 24, TypeScript 5.8, React 18, Webpack 5.

## Build / Dev Commands

```bash
npm run build          # Production webpack build
npm run dev            # Watch-mode dev build
npm run typecheck      # tsc --noEmit
npm run lint           # ESLint (flat config, ESLint 9)
npm run lint:fix       # ESLint autofix
npm run spellcheck     # cspell across all source files (matches CI exactly)
npm run markdownlint  # markdownlint-cli2 on AGENTS.md, CHANGELOG.md, README.md
npm run start          # Docker compose dev environment (Grafana + servers)
npm run stop           # Tear down docker compose
```

## Test Commands

Unit tests use **Jest 30** with `@swc/jest`, `@testing-library/react`, and `jest-environment-jsdom`.
Tests are randomized and mocks are reset between tests.

```bash
npm test                        # Jest watch mode (changed files only)
npm run test:ci                 # Full suite, 4 workers, coverage

# Run a single test file
npx jest src/components/FormPanel/FormPanel.test.tsx

# Run tests matching a name pattern
npx jest --testNamePattern="Should display"

# Single file + name pattern
npx jest src/module.test.ts --testNamePattern="Should be instance"
```

E2E tests use **Playwright** with `@grafana/plugin-e2e`:

```bash
npm run test:e2e                # Playwright headless
npm run test:e2e:dev            # Playwright with UI
npm run test:e2e:docker         # Playwright in Docker container
```

## Critical Rules

- **Never modify anything inside `.config/`** — it is managed by Grafana plugin tooling.
- **Never change `id` or `type` in `src/plugin.json`** — changes to plugin.json require a Grafana server restart.
- When you need Grafana API docs, fetch from `https://grafana.com/developers/plugin-tools/llms.txt`.
- **Always update the PR summary** when pushing new commits to a PR.

## Project Structure

```text
src/
  components/       # PascalCase dirs, each with Component.tsx, .test.tsx, .styles.ts, index.ts
  constants/        # Shared constants including TEST_IDS (tests.ts)
  hooks/            # Custom React hooks (use* prefix)
  types/            # TypeScript types, interfaces, enums
  utils/            # Utility functions
  module.ts         # Plugin entry point
  plugin.json       # Plugin metadata
test/               # Playwright E2E tests
provisioning/       # Grafana provisioning for local dev
server-json/        # Test helper servers
server-pg/
```

## Path Aliases

`@/*` maps to `src/*`. Use it for non-relative imports within `src/`:

```ts
import { TEST_IDS } from '@/constants';
import { FormElementType } from '@/types';
import { toLocalFormElement } from '@/utils';
```

Use relative imports only for siblings/parents within the same component directory:

```ts
import { FormPanel } from './FormPanel';
import { FormElements } from '../FormElements';
```

## Import Ordering

1. External packages (`@emotion/css`, `@grafana/*`, `@volkovlabs/*`, `lodash`, `react`)
2. Path-aliased internal imports (`@/constants`, `@/hooks`, `@/types`, `@/utils`)
3. Relative imports (`./FormPanel`, `../FormElements`)

## Naming Conventions

| What                 | Convention                                            | Example                                 |
| -------------------- | ----------------------------------------------------- | --------------------------------------- |
| Component files/dirs | PascalCase                                            | `FormPanel/FormPanel.tsx`               |
| Non-component files  | kebab-case                                            | `form-element.ts`, `code-parameters.ts` |
| Components           | PascalCase                                            | `FormPanel`, `CustomCodeEditor`         |
| Types/Interfaces     | PascalCase                                            | `PanelOptions`, `FormElement`           |
| Enums                | PascalCase name, UPPER_SNAKE values                   | `FormElementType.BOOLEAN`               |
| Constants            | UPPER_SNAKE_CASE                                      | `TEST_IDS`, `FORM_ELEMENT_DEFAULT`      |
| Hooks                | camelCase, `use` prefix                               | `useAutoSave`, `useFormLayout`          |
| Barrel exports       | `index.ts` with `export * from './...'` per directory |                                         |

## Formatting & TypeScript

- **Prettier**: Print width 120, single quotes, trailing commas (es5), semicolons, 2-space indent, no tabs.
- **Strict mode** enabled. Use `const enum` for enums (see `FormElementType`).
- JSDoc block comments on all exported interfaces, types, functions, and their properties:

```ts
/**
 * Panel Options
 */
export interface PanelOptions {
  /**
   * Sync
   * @type {boolean}
   */
  sync: boolean;
}
```

## Component Pattern

Each component lives in `ComponentName/` with `ComponentName.tsx`, `.test.tsx`, `.styles.ts`, and `index.ts`.

Styles use Grafana's `useStyles2` hook with `@emotion/css`:

```ts
export const getStyles = (theme: GrafanaTheme2) => ({
  wrapper: css`
    display: flex;
  `,
});

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/business-forms](https://github.com/grafana/business-forms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
