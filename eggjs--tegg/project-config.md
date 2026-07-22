---
trigger: always_on
description: Provides the fundamental decorators for dependency injection.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

tegg is a TypeScript dependency injection framework and plugin system for [Egg.js](https://eggjs.org/). It provides decorator-based dependency injection, lifecycle management, and modular architecture for building scalable Node.js applications.

## Common Commands

```bash
# Install dependencies
npm install

# Run all tests across workspaces
npm test

# Run tests for a single package
npm test --workspace=core/metadata
npm test --workspace=plugin/tegg

# Run a single test file (from package directory)
cd core/metadata && npm test -- --grep "test name"

# Lint
npm run lint
npm run lint:fix

# Build TypeScript
npm run tsc

# Full CI (prepare + lint + test)
npm run ci

# Publish (requires permissions)
npm run bump    # version bump with lerna
npm run pub     # publish to npm
```

## Architecture

### Monorepo Structure

This is a Lerna monorepo with three workspace categories:

- **core/**: Core packages containing decorators, runtime, and utilities
- **plugin/**: Egg.js plugins that integrate tegg features
- **standalone/**: Standalone runtime without Egg.js dependency

### Key Packages

| Package | Description |
|---------|-------------|
| `@eggjs/tegg` | Main entry point, re-exports all decorator packages |
| `@eggjs/tegg-plugin` | Main Egg.js plugin for tegg integration |
| `@eggjs/core-decorator` | Core decorators: `@ContextProto`, `@SingletonProto`, `@Inject` |
| `@eggjs/tegg-types` | TypeScript type definitions |
| `@eggjs/tegg-metadata` | Metadata storage and graph building |
| `@eggjs/tegg-runtime` | Runtime container and object instantiation |
| `@eggjs/tegg-lifecycle` | Object lifecycle hooks |

### Core Concepts

**Prototype Types (instantiation modes):**
- `@ContextProto`: One instance per request context
- `@SingletonProto`: One instance for entire application lifecycle
- `@MultiInstanceProto`: Multiple instances with different qualifiers

**Access Levels:**
- `AccessLevel.PRIVATE`: Only accessible within the same module
- `AccessLevel.PUBLIC`: Accessible from other modules

**Object Init Types:**
- `ObjectInitType.CONTEXT`: New instance per request
- `ObjectInitType.SINGLETON`: Single instance for app lifetime
- `ObjectInitType.ALWAYS_NEW`: New instance on every injection

### Dependency Injection Flow

1. Decorators (`@ContextProto`, `@SingletonProto`, etc.) register metadata on classes
2. `@eggjs/tegg-loader` scans and loads modules
3. `@eggjs/tegg-metadata` builds a dependency graph (GlobalGraph, ModuleGraph)
4. `@eggjs/tegg-runtime` instantiates objects based on the graph
5. `@Inject` decorator triggers dependency resolution at runtime

### Lifecycle Hooks

Objects can implement `EggObjectLifecycle` interface or use decorators:
- `@LifecyclePostConstruct` / `postConstruct()`
- `@LifecyclePreInject` / `preInject()`
- `@LifecyclePostInject` / `postInject()`
- `@LifecycleInit` / `init()`
- `@LifecyclePreDestroy` / `preDestroy()`
- `@LifecycleDestroy` / `destroy()`

## Import Guidelines

### Application Code (Egg.js apps using tegg)

Always import from `@eggjs/tegg` - it re-exports everything needed:

```typescript
// Core decorators and enums
import {
  ContextProto,
  SingletonProto,
  Inject,
  AccessLevel,
  EggQualifier,
  EggType,
} from '@eggjs/tegg';

// Subpath imports for specific features
import { Advice, Crosscut, Pointcut } from '@eggjs/tegg/aop';
import { DataSource } from '@eggjs/tegg/orm';
import { Schedule } from '@eggjs/tegg/schedule';
import { Transactional } from '@eggjs/tegg/transaction';
```

Available subpaths: `aop`, `orm`, `dal`, `schedule`, `transaction`, `ajv`, `helper`, `standalone`

### Advanced Usage (custom loaders, lifecycle hooks)

Use `@eggjs/tegg/helper` for internal APIs:

```typescript
import {
  ModuleConfigUtil,
  LoaderFactory,
  EggObjectLifeCycleContext,
  EggObject,
} from '@eggjs/tegg/helper';
```

### Framework Internal Code (packages within this monorepo)

**For type-only imports**, use `@eggjs/tegg-types`:

```typescript
import type { EggPrototype, EggObject, EggObjectLifecycle } from '@eggjs/tegg-types';
import { AccessLevel, ObjectInitType } from '@eggjs/tegg-types';
```

**For decorator utilities**, import from `@eggjs/core-decorator`:

```typescript
import { PrototypeUtil, QualifierUtil, MetadataUtil } from '@eggjs/core-decorator';
```

### Package Dependency Rules

| If you're in... | Import decorators from | Import types from |
|-----------------|----------------------|-------------------|
| Application code | `@eggjs/tegg` | `@eggjs/tegg` |
| `plugin/*` packages | `@eggjs/tegg` | `@eggjs/tegg-types` |
| `core/*` packages | `@eggjs/core-decorator` | `@eggjs/tegg-types` |

## Best Practices

### Prefer SingletonProto for Performance

Use `@SingletonProto` by default unless you need request-scoped state. Singleton objects are created once at startup, avoiding repeated instantiation overhead per request.

```typescript
// Good: Stateless service as singleton
@SingletonProto()
export class UserRepository {
  async findById(id: string) { /* ... */ }
}

// Use ContextProto only when you need request-specific state
@ContextProto()
export class RequestTracer {
  @Inject()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eggjs/tegg](https://github.com/eggjs/tegg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
