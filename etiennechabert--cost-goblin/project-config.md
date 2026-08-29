---
trigger: always_on
description: > Read `SPEC.md` for the full product specification. This file tells you HOW to work. The spec tells you WHAT to build.
---

# CostGoblin — Development Instructions

> Read `SPEC.md` for the full product specification. This file tells you HOW to work. The spec tells you WHAT to build.

## Project Overview

CostGoblin is a desktop app (Electron + TypeScript + DuckDB) for cloud cost visibility. It syncs billing data from S3, stores it locally as Parquet, and queries it with DuckDB. See `SPEC.md` for full architecture and features.

## Monorepo Structure

```
costgoblin/
  data/                     # .gitignore'd — real Parquet data for development only
  packages/
    core/                   # @costgoblin/core — pure TypeScript, no framework deps
    desktop/                # Electron shell — imports @costgoblin/core
    ui/                     # @costgoblin/ui — shared React components
    web-backend/            # (Future) server — imports @costgoblin/core
```

**Package dependency rules:**
- `core` has ZERO dependency on `desktop`, `ui`, or any framework
- `ui` depends on `core` (for types only — never imports query/sync logic directly)
- `desktop` depends on `core` and `ui`
- `web-backend` (future) depends on `core` only

## Commands

```bash
# Verify everything — run after EVERY change
npm run check              # tsc --noEmit + eslint + vitest run (~10 seconds)

# Per-package work
cd packages/core && npm run check     # core only
cd packages/ui && npm run check       # UI only (uses mock CostApi)
cd packages/desktop && npm run check  # desktop only
cd packages/desktop && npm run dev    # launch Electron in dev mode with hot reload

# Tests
npx vitest run                        # all tests
npx vitest run packages/core          # core tests only
npx vitest run <specific-file>        # single test file (fastest iteration)

# Fixtures
npx tsx packages/core/src/__fixtures__/generate.ts --profile    # profile real data
npx tsx packages/core/src/__fixtures__/generate.ts --generate   # create synthetic fixtures
npx tsx packages/core/src/__fixtures__/focus-1-2/write-samples.ts  # regenerate the committed AWS/Azure/GCP FOCUS 1.2 samples
```

## Development Workflow

Follow this sequence for EVERY feature:

```
1. Read the relevant SPEC.md section
2. Write types first (interfaces, branded types, discriminated unions)
3. Run: tsc --noEmit → fix type errors
4. Write tests for the expected behavior
5. Run: vitest run <test-file> → see tests fail (red)
6. Write the implementation
7. Run: vitest run <test-file> → see tests pass (green)
8. Run: npm run check → full verification (MUST pass before moving on)
9. If working on UI: npm run dev in desktop/ to visually verify
10. After pushing & opening the PR — close the review loop (see "After pushing"):
    - Run `/code-review max --fix` yourself (via the `Skill` tool), review the fixes it applies, and re-run npm run check
    - Address every sentry[bot] review comment on the PR
```

## TypeScript Rules — STRICTLY ENFORCED

**tsconfig base:**
- `strict: true`
- `noUncheckedIndexedAccess: true`
- `exactOptionalPropertyTypes: true`
- `noImplicitReturns: true`
- `noFallthroughCasesInSwitch: true`

**BANNED — never use these, fix the design instead:**
- `any` type → use `unknown` and narrow, or define a proper type
- `@ts-ignore` and `@ts-expect-error` → fix the type error
- `as` type assertions → use type guards and discriminated unions
- Non-null assertions (`!`) → handle the null case explicitly
- `eslint-disable` comments → no exceptions, no line-level overrides
- `console.log` → use the structured logger

**REQUIRED patterns:**

Branded types for domain concepts:
```typescript
type Brand<T, B extends string> = T & { readonly __brand: B };
type DimensionId = Brand<string, 'DimensionId'>;
type EntityRef = Brand<string, 'EntityRef'>;
type TagValue = Brand<string, 'TagValue'>;
type Dollars = Brand<number, 'Dollars'>;
```

Discriminated unions for all state (no impossible states):
```typescript
// WRONG:
{ isLoading: boolean; error?: Error; data?: CostResult }

// RIGHT:
type QueryState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }
```

## Linting Rules

- `@typescript-eslint/strict-type-checked` ruleset
- `@typescript-eslint/no-explicit-any`: error
- `@typescript-eslint/no-unsafe-assignment`: error
- `@typescript-eslint/no-non-null-assertion`: error
- `no-console`: error
- Import sorting enforced
- Formatting: Biome (configured once, never debated)

## Testing Approach

### Layer 1: Core Logic (Vitest)
Pure functions: config loader, tag normalizer, alias resolver, org tree traversal, cost math.
No I/O, no DuckDB. Tests run in milliseconds.

### Layer 2: DuckDB Queries (Vitest)
Real DuckDB queries against synthetic fixture Parquet files in `packages/core/src/__fixtures__/synthetic/`.
Shared DuckDB instance created once per test suite. Fixtures are small (~1000 rows), queries complete in milliseconds.
Provider-shape coverage uses the committed FOCUS 1.2 samples in `packages/core/src/__fixtures__/focus-1-2/` (AWS/Azure/GCP native export shapes, materialized to Parquet at test time).

### Layer 3: React Components (Vitest + React Testing Library)
Components tested against `MockCostApi` (implements `CostApi` interface with fixture data).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [etiennechabert/cost-goblin](https://github.com/etiennechabert/cost-goblin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
