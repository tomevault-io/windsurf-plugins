---
trigger: always_on
description: Standards and patterns used throughout the GPC codebase. Follow these when contributing or building plugins.
---


# Code Conventions

Standards and patterns used throughout the GPC codebase. Follow these when contributing or building plugins.

## TypeScript

### Strict Mode

All packages use TypeScript strict mode (`"strict": true` in tsconfig). This enables `strictNullChecks`, `noImplicitAny`, and all other strict checks.

### ESM-First

All packages use ES modules. No CommonJS `require()` calls.

```typescript
// Correct
import { PlayApiClient } from "@gpc-cli/api";

// Wrong
const { PlayApiClient } = require("@gpc-cli/api");
```

### Named Exports Only

No default exports anywhere in the codebase. Every module uses named exports.

```typescript
// Correct
export { PlayApiClient };
export { ServiceAccountAuth };

// Wrong
export default PlayApiClient;
```

### Explicit Return Types

All exported functions have explicit return type annotations.

```typescript
// Correct
export function createClient(options: ClientOptions): PlayApiClient {
  // ...
}

// Wrong — missing return type
export function createClient(options: ClientOptions) {
  // ...
}
```

### No `any`

Use `unknown` and narrow with type guards instead of `any`.

```typescript
// Correct
function parseResponse(data: unknown): AppInfo {
  if (typeof data !== "object" || data === null) {
    throw new ApiError("Invalid response");
  }
  // narrow and validate
}

// Wrong
function parseResponse(data: any): AppInfo {
  return data as AppInfo;
}
```

### Barrel Exports

Each package has an `index.ts` that re-exports the public API.

```typescript
// packages/api/src/index.ts
export { PlayApiClient } from "./client.js";
export { ReportingApiClient } from "./reporting-client.js";
export type { ClientOptions, ApiResponse } from "./types.js";
```

## Naming Conventions

| Entity       | Convention                          | Example               |
| ------------ | ----------------------------------- | --------------------- |
| Files        | kebab-case                          | `rate-limiter.ts`     |
| Classes      | PascalCase                          | `ApiClient`           |
| Interfaces   | PascalCase (no `I` prefix)          | `AuthStrategy`        |
| Types        | PascalCase                          | `TrackRelease`        |
| Functions    | camelCase                           | `uploadBundle()`      |
| Constants    | UPPER_SNAKE_CASE                    | `MAX_RETRY_COUNT`     |
| Env vars     | UPPER*SNAKE_CASE with `GPC*` prefix | `GPC_SERVICE_ACCOUNT` |
| CLI flags    | kebab-case                          | `--service-account`   |
| npm packages | `@gpc-cli/<name>`                   | `@gpc-cli/core`       |

## Import Order

Sort imports in this order, with a blank line between groups:

1. Node.js built-ins
2. External dependencies
3. Internal packages (`@gpc-cli/*`)
4. Relative imports

```typescript
import { readFileSync } from "node:fs";
import { resolve } from "node:path";

import { Command } from "commander";

import { PlayApiClient } from "@gpc-cli/api";
import { ServiceAccountAuth } from "@gpc-cli/auth";

import { formatOutput } from "./formatters.js";
import type { UploadOptions } from "./types.js";
```

## Git Conventions

### Branch Strategy

Trunk-based development on `main`. Short-lived branches only for risky experiments.

```
main                          # Primary branch (direct commits)
feat/<scope>/<short-desc>     # Feature branches (when needed)
fix/<scope>/<short-desc>      # Bug fixes (when needed)
chore/<scope>/<short-desc>    # Maintenance
docs/<short-desc>             # Documentation
```

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:** `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `ci`, `build`

**Scopes:** `api`, `auth`, `config`, `core`, `cli`, `plugin-sdk`, `ci`, `docs`

**Examples:**

```
feat(cli): add gpc releases upload command
fix(auth): handle expired refresh tokens gracefully
docs(api): add rate limiting section to API reference
chore(deps): update googleapis to v130
refactor(core): extract rollout logic into dedicated module
test(auth): add service account auth integration tests
```

### Pull Requests

- One feature/fix per PR
- Require at least 1 review
- Must pass CI (lint, typecheck, test)
- Squash merge to `main`
- PR title follows commit convention

## Testing Conventions

### Framework

All tests use [Vitest](https://vitest.dev/). Tests are TypeScript-native and ESM-first.

### File Structure

Tests live in a `tests/` directory inside each package:

```
packages/api/
├── src/
│   ├── client.ts
│   └── rate-limiter.ts
└── tests/
    ├── client.test.ts
    ├── rate-limiter.test.ts
    └── fixtures/
        └── mock-responses.json
```

### Coverage Targets

| Package           | Target |
| ----------------- | ------ |
| `@gpc-cli/api`    | 90%    |
| `@gpc-cli/auth`   | 90%    |
| `@gpc-cli/config` | 95%    |
| `@gpc-cli/core`   | 85%    |
| `@gpc-cli/cli`    | 80%    |

### Mock External APIs

Never call real Google APIs in tests. Mock fetch with `vi.stubGlobal`:

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest";

describe("PlayApiClient", () => {
  beforeEach(() => {
    vi.stubGlobal(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yasserstudio/gpc](https://github.com/yasserstudio/gpc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
