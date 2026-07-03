---
trigger: always_on
description: Frontend-only Grafana panel plugin (`volkovlabs-table-panel`).
---

# AGENTS.md — Business Table (Grafana Panel Plugin)

Frontend-only Grafana panel plugin (`volkovlabs-table-panel`).
Node >= 24, npm, webpack (via `.config/`), TypeScript 5.9+, React 18 + React 19.

## Build & Dev Commands

```bash
npm run build          # Production build (webpack)
npm run dev            # Dev build with watch mode + live reload
npm run typecheck      # tsc --noEmit
npm run lint           # ESLint (ts/tsx/js/jsx)
npm run lint:fix       # ESLint with auto-fix
npm run test           # Jest in watch mode (changed files only)
npm run test:ci        # Jest full run (CI, 4 workers)
npm run test:e2e       # Playwright E2E tests
npm run test:e2e:dev   # Playwright interactive UI mode
npm run test:e2e:docker # Full Docker Compose (Grafana + tests)
npm run markdownlint  # markdownlint-cli2 on AGENTS.md, CHANGELOG.md, README.md
npm run spellcheck   # cspell on all source files
npm run start          # Docker compose: Grafana + plugin (dev profile)
npm run stop           # Docker compose down
```

### Running a Single Test

```bash
npx jest src/components/Table/Table.test.tsx           # Single file
npx jest src/components/Table/Table.test.tsx -t "Should render"  # Single test by name
npx jest --testPathPattern="useTable"                  # Pattern match on path
```

Jest config: `resetMocks: true` (mocks reset between tests),
`randomize: true` (random test order), timezone forced to UTC.

## Project Structure

```text
src/
  components/       # React components (editors/, Table/, TablePanel/, ui/)
  hooks/            # Custom React hooks (useTable, useAutoSave, usePagination, etc.)
  types/            # TypeScript type definitions with barrel exports
  utils/            # Utility functions (actions, editor, export, table, test, etc.)
  constants.ts      # TEST_IDS, default configs, page sizes, regex patterns
  migration.ts      # Panel options migration handler
  module.ts         # PanelPlugin entry point
  plugin.json       # Plugin manifest
  __mocks__/        # Jest manual mocks for @grafana/*, @hello-pangea/dnd, etc.
test/               # Playwright E2E tests + helpers
provisioning/       # Grafana dashboard/datasource provisioning JSON
```

Path alias: `@/` maps to `src/` (webpack, tsconfig, and jest all resolve it).

## Code Style

### Import Ordering

Imports follow this strict order with blank lines between groups:

1. `@grafana/*` packages (data, runtime, ui, schema, e2e-selectors)
2. Third-party packages (`@tanstack/*`, `@emotion/css`, `react`, `semver`, etc.)
3. Path-aliased imports (`@/constants`, `@/hooks`, `@/types`, `@/utils`)
4. Relative imports (`./`, `../`)

```typescript
import { PanelProps } from '@grafana/data';
import { config } from '@grafana/runtime';
import { Alert, Button, useStyles2 } from '@grafana/ui';

import { Table as TableInstance } from '@tanstack/react-table';
import React, { useCallback, useMemo, useState } from 'react';

import { TEST_IDS } from '@/constants';
import { useContentSizes, useTable } from '@/hooks';
import { PanelOptions } from '@/types';

import { Table } from '../Table';
import { getStyles } from './TablePanel.styles';
```

### Naming Conventions

| Element          | Convention                       | Example                                          |
| ---------------- | -------------------------------- | ------------------------------------------------ |
| Components       | PascalCase files + named exports | `TablePanel.tsx`, `export const TablePanel`      |
| Hooks            | `use` prefix, camelCase          | `useAutoSave.ts`, `useTable.ts`                  |
| Type files       | kebab-case                       | `column-editor.ts`, `nested-object.ts`           |
| Interfaces/Types | PascalCase                       | `PanelOptions`, `ColumnConfig`                   |
| Enums            | PascalCase name, SCREAMING_SNAKE | `CellType.COLORED_TEXT`, `ColumnAlignment.START` |
| Constants        | SCREAMING_SNAKE_CASE             | `TEST_IDS`, `AUTO_SAVE_TIMEOUT`                  |
| Styles files     | `Component.styles.ts`            | `TablePanel.styles.ts`                           |

### JSDoc Comments

This codebase uses **pervasive JSDoc comments**. Add `/** ... */` blocks above:

- Every interface and each of its properties (include `@type` tags on properties)
- Every function and constant declaration
- Logical sections within function bodies (state, theme, callbacks, return)

```typescript
/**
 * Query Options Mapper
 */
export interface QueryOptionsMapper {
  /**
   * Source
   *
   * @type {number}
   */
  source: string | number;
}
```

### TypeScript Patterns

- **Enums over string unions** for configuration values.
- **Named exports only** — no default exports in source files.
- **Barrel exports** via `index.ts` in every directory using `export *`.
- **`Props` type** defined at component scope or derived via
  `React.ComponentProps<typeof X>`.

### React Components

- **Functional components only** using `React.FC<Props>` with arrow functions.
- Props destructured in the function signature, not inside the body.
- Styles via `@emotion/css` + Grafana's `useStyles2(getStyles)` pattern.
- Style functions: `(theme: GrafanaTheme2) => ({ className: css\`...\` })`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/business-table](https://github.com/grafana/business-table) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
