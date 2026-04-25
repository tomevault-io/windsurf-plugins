---
trigger: always_on
description: Browser-based management interface for [LibreChat](https://github.com/danny-avila/LibreChat).
---

# LibreChat Admin Panel

## Overview

Browser-based management interface for [LibreChat](https://github.com/danny-avila/LibreChat).
Connects to the same database as the main application and provides a GUI for
configuration, user/group/role management, and capability grants.

## Tech Stack

- **Framework:** TanStack Start (React 19 + TanStack Router + React Query)
- **UI:** ClickHouse click-ui component library + Tailwind CSS 4
- **Language:** TypeScript (strict mode, `verbatimModuleSyntax`)
- **Build:** Vite 8
- **Testing:** Vitest (unit), Playwright (e2e)
- **Linting:** ESLint
- **Package manager:** Bun (preferred), pnpm, or npm all work

## Project Structure

```
src/
├── components/
│   ├── access/            # Roles, groups, members management
│   ├── configuration/     # Config editor (schema-driven forms)
│   │   └── fields/        # Individual field type renderers
│   ├── grants/            # Capability grants and audit log
│   ├── shared/            # Reusable UI components
│   └── users/             # User management
├── contexts/              # React contexts (theme)
├── hooks/                 # Custom hooks
├── locales/               # i18n translation files
├── routes/                # TanStack Router file-based routes
├── server/                # Server functions (TanStack Start createServerFn)
├── test/                  # Test fixtures and setup
├── types/                 # TypeScript type definitions
└── utils/                 # Pure utility functions
```

---

## Code Style

### Naming and File Organization

- **Single-word file names** whenever possible (e.g., `permissions.ts`, `capabilities.ts`, `service.ts`).
- When multiple words are needed, prefer grouping related modules under a **single-word directory** rather than using multi-word file names (e.g., `admin/capabilities.ts` not `adminCapabilities.ts`).
- The directory already provides context — `app/service.ts` not `app/appConfigService.ts`.

### Structure and Clarity

- **Never-nesting**: early returns, flat code, minimal indentation. Break complex operations into well-named helpers.
- **Functional first**: pure functions, immutable data, `map`/`filter`/`reduce` over imperative loops. Only reach for OOP when it clearly improves domain modeling or state encapsulation.
- **No dynamic imports** unless absolutely necessary.

### DRY

- Extract repeated logic into utility functions.
- Reusable hooks / higher-order components for UI patterns.
- Parameterized helpers instead of near-duplicate functions.
- Constants for repeated values; configuration objects over duplicated init code.
- Shared validators, centralized error handling, single source of truth for business rules.
- Shared typing system with interfaces/types extending common base definitions.
- Abstraction layers for external API interactions.

### Iteration and Performance

- **Minimize looping** — every additional pass adds up at scale.
- Consolidate sequential O(n) operations into a single pass whenever possible; never loop over the same collection twice if the work can be combined.
- Choose data structures that reduce the need to iterate (e.g., `Map`/`Set` for lookups instead of `Array.find`/`Array.includes`).
- Avoid unnecessary object creation; consider space-time tradeoffs.
- Prevent memory leaks: careful with closures, dispose resources/event listeners, no circular references.

### Type Safety

- **Never use `any`**. Explicit types for all parameters, return values, and variables.
- **Limit `unknown`** — avoid `unknown`, `Record<string, unknown>`, and `as unknown as T` assertions. A `Record<string, unknown>` almost always signals a missing explicit type definition.
- **Don't duplicate types** — before defining a new type, check whether it already exists in the project. Reuse and extend existing types rather than creating redundant definitions.
- Use union types, generics, and interfaces appropriately.
- All TypeScript and ESLint warnings/errors must be addressed — do not leave unresolved diagnostics.

### Comments and Documentation

- Write self-documenting code; no inline comments narrating what code does.
- JSDoc only for complex/non-obvious logic or intellisense on public APIs.
- Single-line JSDoc for brief docs, multi-line for complex cases.
- Avoid standalone `//` comments unless absolutely necessary.

### JS/TS Loop Preferences

- **Limit looping as much as possible.** Prefer single-pass transformations and avoid re-iterating the same data.
- `for (let i = 0; ...)` for performance-critical or index-dependent operations.
- `for...of` for simple array iteration.
- `for...in` only for object property enumeration.

---

## Import Conventions

### Order

Imports form a single block with no blank lines, sorted in four groups:

1. **External packages** — shortest line to longest
2. **`import type` from packages** — longest line to shortest
3. **`import type` from local modules** — longest line to shortest
4. **Local modules** — longest line to shortest

```typescript
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Button, Icon } from '@clickhouse/click-ui';
import type { DynamicSettingProps } from 'librechat-data-provider';
import type { AdminGroup } from '@librechat/data-schemas';

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClickHouse/librechat-admin-panel](https://github.com/ClickHouse/librechat-admin-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
