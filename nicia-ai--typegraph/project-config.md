---
trigger: always_on
description: - Follow requirements carefully and to the letter
---


# Philosophy

- Follow requirements carefully and to the letter
- Think step-by-step, describe your plan, then implement
- Fully implement all functionality - no TODOs, placeholders, or missing pieces
- Prioritize readability and maintainability over performance optimization
- If uncertain, say so rather than guessing

# Project Structure

Monorepo using Turbo with pnpm workspaces.

```
typegraph/
├── packages/
│   ├── typegraph/          # Core library (@nicia-ai/typegraph)
│   │   ├── src/
│   │   │   ├── backend/    # Database backends (SQLite, PostgreSQL, Drizzle)
│   │   │   ├── core/       # Node/edge definitions, graph DSL
│   │   │   ├── query/      # Query builder, predicates, SQL compilation
│   │   │   ├── store/      # Runtime store, collections, operations
│   │   │   ├── ontology/   # Semantic relationships (subClassOf, etc.)
│   │   │   ├── schema/     # Serialization, migration, versioning
│   │   │   ├── errors/     # Error types
│   │   │   └── utils/      # Shared utilities (Result, date, id)
│   │   ├── tests/          # Unit and integration tests
│   │   │   ├── property/   # Property-based tests (fast-check)
│   │   │   └── backends/   # Backend-specific tests
│   │   └── examples/       # Runnable examples
│   ├── eslint-config/      # Shared ESLint configuration
│   └── benchmarks/         # Performance benchmarks
├── apps/
│   └── docs/               # Documentation site (Astro/Starlight)
└── package.json            # Monorepo root (pnpm + turbo)
```

# Tech Stack

- **Zod** - Schema validation and TypeScript type inference
- **Drizzle ORM** - Database abstraction for SQLite and PostgreSQL
- **Vitest** - Test runner
- **fast-check** - Property-based testing
- **Stryker** - Mutation testing
- **tsup** - Build tool

# Common Commands

```bash
# From repository root
pnpm install              # Install all dependencies
pnpm build                # Build all packages
pnpm test                 # Run all tests (SQLite only, postgres tests are skipped)
pnpm test:postgres        # Run PostgreSQL tests (starts Docker automatically)
pnpm lint                 # Run ESLint
pnpm typecheck            # TypeScript type checking
pnpm fix                  # Auto-fix lint and formatting (prettier + eslint --fix + markdownlint)
pnpm test:unused          # Run knip (unused exports, deps, files)

# From packages/typegraph
pnpm test                 # Run unit tests
pnpm test:unit            # Run unit tests only
pnpm test:property        # Run property-based tests
pnpm test:postgres        # Run PostgreSQL tests (starts Docker automatically)
pnpm test:coverage        # Run tests with coverage
pnpm test:mutation        # Run mutation testing
```

# Before Committing

Running `pnpm typecheck` and `pnpm lint` separately is *not* enough —
prettier rules live outside eslint, and those commands won't surface
formatting drift. The canonical pre-commit sequence is:

```bash
pnpm fix && pnpm typecheck && pnpm test
```

`pnpm fix` chains prettier, eslint `--fix` (so a separate `pnpm lint`
is redundant), and markdownlint, exiting non-zero on any unfixable
violation. If it modifies files, fold the changes into the same
commit — they aren't a separate concern.

**Important:** `pnpm test` runs only SQLite-backed tests. The PostgreSQL
backend tests are **skipped** unless `POSTGRES_URL` is set. Always run
`pnpm test:postgres` (from the repo root or `packages/typegraph`) to verify
changes that touch backend, store, or collection code. The script handles
Docker lifecycle automatically — no manual setup required.

# Core Principles

- **TypeScript strict mode** with readonly types by default
- **Functional programming** over classes
- **Immutable data** patterns
- **Explicit error handling** with Result/Either patterns
- **Single responsibility** - one concern per file/function

# Type Safety

- MUST avoid `any` - use strict types
- SHOULD use `as const` for literal types
- SHOULD prefer type predicates over type assertions
- SHOULD use discriminated unions for state
- SHOULD use `satisfies` operator for type-safe object literals
- SHOULD use `NoInfer<T>` to prevent unwanted inference
- MUST export types from their defining modules
- MUST use `type` imports for type-only imports

# Code Style

## Formatting

- MUST use `function` keyword for pure functions (not arrow functions at top level)
- MUST use braces around switch case statements
- SHOULD avoid unnecessary braces in conditionals for simple statements
- MUST use descriptive names that reveal intent
- MUST use descriptive variable names (ex: `event`, not `e`)

## TypeScript Patterns

- SHOULD avoid `let` unless it adds substantial clarity
- MUST use nullish coalescing (`??`) over logical or (`||`) when appropriate
- MUST avoid `null` - always prefer `undefined`
- MUST use `Readonly<{...}>` type syntax over marking individual members readonly
- SHOULD avoid mutation unless absolutely necessary
- SHOULD use `structuredClone()` for deep copying
- SHOULD prefer spreading over `Object.assign`

## Array Operations

- MUST NOT pass function references directly to array methods

  ```typescript
  // ❌ Avoid - harder to debug, less explicit about arguments
  array.map(transform);

  // ✅ Prefer - explicit arguments, easier debugging, better type inference

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicia-ai/typegraph](https://github.com/nicia-ai/typegraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
