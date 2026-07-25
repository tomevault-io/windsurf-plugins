---
trigger: always_on
description: This monorepo contains **Zero** (real-time sync platform) and **Replicache** (client-side data layer), built as complementary technologies for building reactive, sync-enabled applications.
---

# Rocicorp Monorepo Instructions

## Architecture Overview

This monorepo contains **Zero** (real-time sync platform) and **Replicache** (client-side data layer), built as complementary technologies for building reactive, sync-enabled applications.

### Repo Structure

```
mono/
├── packages/          # 29 core packages (libraries and engines)
│   ├── zero-client    # Main Zero client (uses Replicache)
│   ├── zero-cache     # Server-side cache and sync engine
│   ├── zero-server    # Server-side mutations/queries
│   ├── zero-schema    # Schema definition builder
│   ├── zql            # IVM (Incremental View Maintenance) query engine and language
│   ├── replicache     # Core client-side sync library
│   └── shared         # Shared utilities and testing helpers
├── apps/              # 3 applications
│   ├── zbugs          # Reference app (React + Wouter + Zero + PostgreSQL)
│   ├── otel-proxy     # OpenTelemetry proxy
│   └── zql-viz        # Query visualization tool
├── tools/             # 5 development tools
└── prod/              # Production deployment (SST/Pulumi)
```

### Data Flow Architecture

Zero follows a **sync-first** model: client queries are reactive and automatically update when server data changes. ZQL queries are transformed to SQL on the server and results are incrementally maintained.

## Development Workflow

### Essential Commands

```bash
# Install and build everything
pnpm install && pnpm run build

# Run tests (uses vitest)
pnpm run test              # All tests
pnpm run test:watch        # Watch mode

# Type checking and linting
pnpm run check-types       # TypeScript across all packages
pnpm run lint              # oxlint with type-awareness
pnpm run format            # oxfmt formatting
```

**Always run `lint`, `format` and `check-types` after every change.**

### Package-Level Commands

Prefer package-level commands when possible. Each package supports: `test`, `check-types`, `lint`, `format`, `build`. e.g.:

```bash
pnpm --filter zero-client run format
pnpm --filter zero-cache run lint
pnpm --filter zero-server run check-types

# Run with coverage (prefer using this flag when possible)
pnpm --filter zero-client run test --coverage

# Run specific test file
pnpm --filter zero-client run test zero.test
```

### Zero Cache Development

```bash
# Start Zero cache server for local development
pnpm run start-zero-cache

# In zbugs app - start Zero cache with schema hot-reload
pnpm run zero-cache-dev
```

## Code Conventions

### TypeScript Patterns

- **Optional fields**: Always explicitly typed as `type | undefined` (not just `type?`)

  ```typescript
  // Correct
  interface User {
    name?: string | undefined;
  }

  // Incorrect
  interface User {
    name?: string;
  }
  ```

### Zero Schema Definition

Zero schemas use a builder pattern with method chaining:

```typescript
const user = table('user')
  .columns({
    id: string(),
    name: string().optional(),
    role: enumeration<Role>(),
  })
  .primaryKey('id');
```

### Testing Patterns

- Use **vitest** for all testing
- Tests are co-located with source files using environment-specific naming:
  - `.test.ts` - Standard tests (Node.js environment)
  - `.node.test.ts` - Node-specific tests (Replicache)
  - `.web.test.ts` - Browser tests (Replicache)
  - `.pg.test.ts` - PostgreSQL integration tests
- Multiple vitest configs for different environments (e.g., `vitest.config.pg-16.ts` for PostgreSQL tests)
- Test files automatically discovered by the root vitest config
- Prefer `test` over `it` for consistency
- Coverage is run with `v8` - use the `--coverage` flag to help write tests

### Import Patterns

- **DO NOT import from `mod.ts`**: Use direct relative paths instead

  ```typescript
  // Correct - use relative path
  import {helper} from './helper.ts';

  // Incorrect - don't import from mod.ts
  import {helper} from './mod.ts';
  ```

- **DO NOT use `import()` in type expressions**: Always use `import type` at the top of the file

  ```typescript
  // Correct - import type at the top
  import type {AST} from '../../../zero-protocol/src/ast.ts';
  import type {TTL} from './ttl.ts';

  abstract addServerQuery(ast: AST, ttl: TTL): void;

  // Incorrect - don't use import() in type expressions
  abstract addServerQuery(
    ast: import('../../../zero-protocol/src/ast.ts').AST,
    ttl: import('./ttl.ts').TTL,
  ): void;
  ```

- **DO NOT use dynamic imports (`await import()`) unless necessary**: Use standard static imports

  ```typescript
  // Correct - static import
  import {createBuilder} from '../../../zql/src/query/named.ts';

  // Incorrect - unnecessary dynamic import
  const {createBuilder} = await import('../../../zql/src/query/named.ts');
  ```

  Dynamic imports are only needed for:
  - Lazy-loading heavy modules
  - Conditional imports based on runtime conditions

- **AVOID re-exports that create cycles**: Re-exports can introduce circular dependencies between packages

  ```typescript
  // Incorrect - re-exporting from higher-level package
  // In zero-types/src/schema.ts:
  export type {Schema} from '../zero-schema/src/builder/schema-builder.ts';


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rocicorp/mono](https://github.com/rocicorp/mono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
