---
trigger: always_on
description: Use when you need parameterized tests without a describe wrapper:
---

# Project Overview

## What

- Polen is a framework for building delightful GraphQL developer portals.
- It generates interactive documentation for GraphQL APIs including schema reference docs, changelogs, and custom pages.

# CRITICAL

- always leverage these installed mcps: ref, serena, effect-docs
- all Effect Schema answers MUST be given with full awareness of:
  - https://effect-ts.github.io/effect/effect/Schema.ts.html
  - https://effect-ts.github.io/effect/effect/SchemaAST.ts.html
  - https://effect.website/docs/schema/* (starting with introduction/)
  - https://effect-ts.github.io/effect/effect/Match.ts.html
- always use ref MCP for documentation searches BEFORE using WebFetch or generic web search
- always use exa MCP for current information instead of generic web search
- when researching complex topics, use exa's deep_researcher instead of multiple separate searches
- never use child process exec to execute a script when you could ESM import it instead
- never use ESM dynamic import when you could ESM statically import it instead
- always use tsx to execute TypeScript files
- always use `tsconfig.json` when running tsc to ensure correct configuration
- always use `.js` extension on relative imports (ESM requirement with nodenext module resolution)
- function contracts (public APIs) must be properly typed, but NEVER complicate internal implementations for type safety - use simple types or cast to `any` internally if needed
- **CRITICAL: MUST CAPTURE FAILING TESTS BEFORE FIXING** - Before implementing any fix for a bug or issue, you MUST first create a failing unit test that reproduces the problem, confirm it fails, then implement the fix. This applies to ALL bug fixes except very complex integration scenarios like massive deep state in Playwright browser tests. No exceptions - TDD is mandatory for bug fixes.

# Project Layout

## Root

```
src/
├── cli/         # Command-line interface
├── api/         # Core configuration and build system (defineConfig, schema handling, Vite plugins)
├── template/    # React-based UI components and routes
├── lib/         # Shared utilities (grafaid for GraphQL, file router, helpers)
└── dep/         # Wrapped external dependencies
```

## Local Libraries

### Structure

```
src/lib/
  ├── <NAME: kebab case>/
  │   ├── $.ts                     (namespace export)
  │   ├── $.test.ts                (optional test file)
  │   ├── $$.ts                    (barrel export)
  │   └── <...kebab case>.ts       (code modules)
```

### File Roles

- **$.ts**: Namespace export file
  - When $$.ts exists: `export * as <NAME: Pascal case> from './$$.js'`
  - When only single module: `export * as <NAME: Pascal case> from './<module>.js'`
  - **CRITICAL**: Always points to $$.ts when it exists, NEVER to individual modules

- **$$.ts**: Barrel export file
  - Exports all public APIs: `export * from './<module>.js'`
  - For ADTs: Also exports member namespaces: `export * as Member from './member.js'`
  - **NEVER** reaches into subdirectories - each directory manages its own exports
    - **Exception**: Parent data types CAN re-export nested data type namespaces when they form a hierarchical data model
    - Example: `lifecycles` barrel can export `lifecycle-event/$.js` and `lifecycle/$.js` because they are integral sub-types

- **$.test.ts**: Test file
  - Imports namespace: `import { NameSpace } from './$.js'`
  - Tests public API only

### Import Patterns

**External imports (from other libraries):**

```typescript
import { LibName } from '#lib/lib-name/$' // Namespace
import { specific } from '#lib/lib-name/$$' // From barrel
import { specific } from '#lib/lib-name/module' // From specific module
```

**Internal imports (within same library):**

```typescript
import { specific } from './$$.js' // From barrel
import { LibName } from './$.js' // Namespace
import { specific } from './module.js' // From specific module
```

**ADT-specific imports:**

**CRITICAL RULE**: For ADT unions, ALWAYS import ONLY from $.js (namespace), NEVER from $$.js (barrel)

```typescript
// ✅ CORRECT: Import ONLY from namespace
import { LifecycleEvent } from './lifecycle-event/$.js'
import { Lifecycle } from './lifecycle/$.js'

// ❌ WRONG: NEVER do this
import { Added, LifecycleEvent, Removed } from './lifecycle-event/$$.js'
import { ObjectType, InterfaceType, Lifecycle } from './lifecycle/$$.js'

// To access members, use the namespace pattern:
const added: LifecycleEvent.Added.Added = LifecycleEvent.Added.make({...})
const objectType: Lifecycle.ObjectType.ObjectType = Lifecycle.ObjectType.make({...})
```

# Effect

## General Principles

- Use the effect-docs MCP and content under https://effect.website/docs/schema/introduction/ to only give valid answers
- We want to maximally leverage Effect in an idiomatic way

## Schema

- There should be a 1:1 between data types and modules
- When defining data types use the module pattern (const object with methods) for better Effect ecosystem alignment and flexibility. Only use classes when you specifically need inheritance or instance methods
- Use pascal case when naming schemas
- Prefer `Schema.Struct` over class-based schemas

### Data Type Modules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonkuhrt/polen](https://github.com/jasonkuhrt/polen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
