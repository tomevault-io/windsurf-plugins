---
trigger: always_on
description: Guidelines for agentic coding agents working in this repository.
---

# AGENTS.md

Guidelines for agentic coding agents working in this repository.

## Project Overview

This is a **Bun monorepo** using workspaces. It contains:

- **packages/roon-web-model** - Shared TypeScript types/interfaces
- **packages/roon-web-client** - HTTP client library
- **packages/roon-web-eslint** - Shared ESLint config
- **app/roon-web-api** - Backend API (Bun + Hono)
- **app/roon-web-ng-client** - Angular frontend

**Bun version:** `1.3.9` (enforced in package.json)

---

## Build/Lint/Test Commands

### Root-Level Commands

```bash
bun install                    # Install dependencies
bun run build                  # Build all packages/apps
bun run lint                   # Lint all packages/apps
bun run lint:fix               # Lint and auto-fix
bun run test                   # Run all tests
bun run ci                     # Full CI: install, build, lint, test
bun run backend                # Start backend in dev mode
bun run frontend               # Build client and serve Angular
```

### Package-Specific Commands

Run from package directory or use `--filter`:

```bash
bun run --filter @nihilux/roon-web-api test
bun run --filter @nihilux/roon-web-client lint
```

### Running a Single Test

**Vitest (API and client packages):**

```bash
# Run specific test file
bun run vitest run path/to/specific.test.ts

# Run tests matching pattern
bun run vitest run -t "test name pattern"

# Watch mode
bun run vitest watch path/to/specific.test.ts
```

**Angular/Vitest:**

```bash
cd app/roon-web-ng-client
bun run ng test --include="**/specific.component.spec.ts"
```

---

## Code Style Guidelines

### Prettier Configuration

- **Quotes:** Double quotes (`"`)
- **Semicolons:** Required
- **Arrow parens:** Always
- **Trailing commas:** ES5
- **Bracket spacing:** true

### ESLint Rules

- **Import sorting:** Auto-sorted via `eslint-plugin-simple-import-sort`
- **Unused imports:** Automatically removed
- **Console:** Forbidden (use logger instead)
- **Type checking:** Strict TypeScript rules enabled

### Import Order (Enforced)

Imports are auto-sorted in this order:

1. **Mocks first** - Files matching `@mock*` or `*.mock*`
2. **External packages** - Third-party modules
3. **Internal via aliases** - Path aliases like `@infrastructure`, `@data`, `@service`
4. **Relative imports** - `./` and `../`

Example:

```typescript
import { nanoidMock } from "@mock";
import { clientManagerMock } from "../service/client-manager.mock";

import { Subject } from "rxjs";
import { Hono } from "hono";

import { logger } from "@infrastructure";
import { clientManager } from "@service";
import { Command } from "@nihilux/roon-web-model";

import { handleCommand } from "./command-handler";
```

### Path Aliases

**API (`app/roon-web-api`):**

- `@infrastructure` → `src/infrastructure`
- `@data` → `src/data`
- `@mock` → `src/mock`
- `@service` → `src/service`
- `@roon-kit` → `src/roon-kit`
- `@router` → `src/router`

**Client (`packages/roon-web-client`):**

- `@mock` → `src/mock`

**Angular (`app/roon-web-ng-client`):**

- `@components/*` → `src/app/components/*`
- `@services/*` → `src/app/services/*`
- `@model` → `src/app/model`

### Naming Conventions

| Type                     | Convention         | Example                      |
| ------------------------ | ------------------ | ---------------------------- |
| Variables/functions      | `camelCase`        | `clientId`, `getZone`        |
| Classes/interfaces/types | `PascalCase`       | `ClientManager`, `ZoneState` |
| Constants                | `UPPER_SNAKE_CASE` | `DEFAULT_PORT`               |
| Files                    | `kebab-case`       | `client-manager.ts`          |
| Test files               | `*.test.ts`        | `api-router.test.ts`         |
| Mock files               | `*.mock.ts`        | `client-manager.mock.ts`     |
| Angular selectors        | `nr-prefix`        | `nr-zone-player`             |

### Error Handling

- Use typed catch: `(err: unknown)`
- Log errors via logger: `logger.error(err, "context message")`
- Never use `console.log/error/warn`

### Comments

- **DO NOT add comments** unless explicitly requested
- Code should be self-documenting

---

## Testing Guidelines

### Vitest Configuration (API & Client)

- **Globals:** Enabled (use `describe`, `it`, `expect`, `vi` without imports)
- **Coverage provider:** Istanbul
- **Coverage thresholds:**
  - API: 100% (statements, branches, functions, lines)
  - Client: 100% lines, 98% branches

### Coverage Exclusions

Files excluded from coverage:

- `src/index.ts` - Entry points
- `src/build.ts` - Build scripts
- `src/**/*.mock.ts` - Mock files
- `src/**/*.test.ts` - Test files
- `src/roon-kit/**/*` - Vendored code
- `src/infrastructure/logger.ts` - Logging setup
- `src/infrastructure/host-info.ts` - Host config
- `src/router/app-router.ts` - Bun-specific static serving

### Mock Pattern

```typescript
// src/service/client-manager.mock.ts
import { Mock } from "vitest";

const register: Mock = vi.fn();
const unregister: Mock = vi.fn();

export const clientManagerMock = { register, unregister };

vi.mock("./client-manager", () => ({
  clientManager: clientManagerMock,
}));
```

### Test Pattern

```typescript
// Mock imports FIRST

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nihilux-org/roon-web-stack](https://github.com/nihilux-org/roon-web-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
