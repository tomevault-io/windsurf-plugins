---
trigger: always_on
description: **COMMUNICATION STYLE**: Be aggressively concise. Prioritize brevity over grammar. Examples:
---

# React Query Builder Development Guide

**COMMUNICATION STYLE**: Be aggressively concise. Prioritize brevity over grammar. Examples:

- "Build failed" not "The build has failed"
- "Fixed type error" not "I have fixed the type error"
- "Run tests" not "I will run the tests for you"

This guide covers React Query Builder development: code style, workflow, and other patterns.

## Project Overview

React Query Builder monorepo contains:

- **Core package**: `@react-querybuilder/core` - Non-React utilities, parsers, formatters
- **Main package**: `react-querybuilder` - React components and hooks
- **UI integrations**: Ant Design, Bootstrap, Bulma, Chakra UI, Fluent UI, Mantine, MUI, Tremor
- **Extensions**: Drag-and-drop (`@react-querybuilder/dnd`), date/time processing (`@react-querybuilder/datetime`), React Native (`@react-querybuilder/native`)
- **Documentation**: Docusaurus website

## Development Workflow

### Setup

```bash
bun install
bun run build
```

### Commands

**Development:**

- `bun start` - Hot-reload dev server (all packages, Bun server)
- `bun start:rqb` - Main package dev (Vite server)
- `bun start:antd`, `bun start:material`, etc. - UI packages (Vite server)

**Quality:**

- `bun test ...` when React and DOM not involved (much faster than Vitest)
- `bunx vitest run --coverage` - Run Vitest tests with 100% coverage check
- `bun typecheck` - TypeScript check
- `bun lint` - Type-aware linting
- `bun fmt` - Format (run after changes)
- `bun checkall` - Full CI check (run before submitting a PR)

**Documentation:**

- `bun web` - Serve documentation website locally
- `bun web:skiptypedoc` - Skip TypeDoc generation for faster startup

### Build

- `bun run build` - All packages (concurrent via Bun CLI filter)
- `bun run build:concurrent` - All packages (concurrent via `concurrently`)
- `bun run build:sequential` - Sequential (better for debugging)
- Individual packages: `bun build:rqb`, `bun build:antd`, etc.

## Code Style

### Structure

```
packages/core/src/           # Non-React utilities
packages/react-querybuilder/src/
├── components/        # React components (PascalCase.tsx)
├── hooks/             # Hooks (useHookName.ts)
├── types/             # TypeScript defs
├── utils/             # Utilities (camelCase.ts)
├── styles/            # SCSS
├── redux/             # Redux
└── barrel.ts          # Export aggregator
```

### Naming

- **Components**: PascalCase (`QueryBuilder.tsx`)
- **Hooks**: camelCase with `use` (`useHookName.ts`)
- **Utilities**: camelCase (`generateID.ts`)
- **Types**: camelCase (`basic.ts`)
- **Debug versions**: `*.debug.ts`
- **Tests**: `*.test.ts[x]`

### TypeScript

- Heavy use of generics with constraints
- Conditional types for API flexibility
- Branded types
- React/non-React type separation (core package enables server usage)

```typescript
// Generic component with constraints
export interface QueryBuilderProps<
  RG extends RuleGroupTypeAny,
  F extends FullField,
  O extends FullOperator,
  C extends FullCombinator,
> {
  // Component props
}

// Type-only imports
import type { RuleGroupType } from '../types';
```

### Components

- Composition over inheritance
- Heavy memoization (`React.memo()`)
- Custom hooks for logic
- Context for state

```typescript
export const ComponentName = React.memo(function ComponentName(props: PropsType) {
  const hookResult = useCustomHook(props);
  const memoizedValue = useMemo(() => computation, [dependencies]);

  return <div className={clsx(baseClassNames.component, customClass)} />;
});
```

### Imports/Exports

- Use `index.ts` for aggregation
- `barrel.ts` for exports that don't have a "debug" version
- React/non-React separation

```typescript
import * as React from 'react';
import type { ComponentProps } from '../types';
import { generateID, isRuleGroup } from '../utils';
```

### Styling

- SCSS + CSS custom properties
- BEM-like (`.queryBuilder-rule`)
- SCSS variables for tokens
- Custom `clsx` utility for conditional classes

### State Management

- Immer for immutable updates
- Path-based updates `[0, 1, 2]`
- Custom Redux context to avoid prop drilling

## Bun APIs

This project runs on Bun. Prefer Bun-native APIs over Node.js equivalents in scripts and utilities:

- `Bun.file(path).text()` / `.json()` instead of `fs.readFileSync`
- `Bun.write(path, content)` instead of `fs.writeFileSync`
- `Bun.spawnSync(...)` / `Bun.spawn(...)` instead of `child_process.execSync` / `exec`
- `Bun.serve(...)` instead of `http.createServer`

Only fall back to `node:*` APIs when no Bun equivalent exists.

## Testing

- Vitest + Testing Library
- Helpers in `utils/testing/`
- 100% coverage required - use `bunx vitest run --coverage` to test for coverage

- Test files: `ComponentName.test.tsx`
- Describe blocks: component/function name
- Test cases: Descriptive behavior

## Generated Files

**Never edit:**

- `packages/core/src/utils/parseCEL/celParser.js`
- `packages/core/src/utils/parseSQL/sqlParser.js`
- Examples (except `_template`)

**Regeneration commands:**

- `bun generate-parsers` - Regenerate CEL and SQL parsers
- `bun generate-examples` - Regenerate example projects

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [react-querybuilder/react-querybuilder](https://github.com/react-querybuilder/react-querybuilder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
