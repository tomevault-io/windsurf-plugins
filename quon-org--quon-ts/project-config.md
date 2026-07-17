---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Build**: `pnpm run build` - Compiles TypeScript to JavaScript in `dist/`
- **Test**: `pnpm test` - Runs tests using Node.js test runner
- **Test Watch**: `pnpm run test:watch` - Runs tests in watch mode
- **Examples**: `pnpm run examples` - Runs example code
- **Lint**: `pnpm run lint` - Runs ESLint on TypeScript files
- **Lint Fix**: `pnpm run lint:fix` - Fixes auto-fixable ESLint issues
- **Format**: `pnpm run format` - Formats code with Prettier
- **Format Check**: `pnpm run format:check` - Checks code formatting

## Architecture

This is a reactive programming library built around **Realm**, **Blueprint**, and **Store** - providing a declarative API for managing reactive state and side effects with automatic cleanup.

### Core Concepts

- **Realm**: Represents a space where resources are created and released
- **Blueprint**: A synchronous-style DSL for composing Realms using flatMap chains
- **Store**: Manages multiple values from an Realm with automatic lifecycle management
- **Resource**: Interface for resources that need cleanup, released in proper order
- **Context**: Type-safe dependency injection system for Blueprints

### Key Files

- `src/realm.ts`: Core Realm implementation
- `src/blueprint.ts`: Blueprint DSL implementation
- `src/store.ts`: Store class for managing Realm values with lifecycle
- `src/resource.ts`: Resource interface and composition utilities
- `src/bilink-map.ts`: Bidirectional map for managing Observer-Value relationships
- `src/task-queue.ts`: Task queue for managing async operations
- `src/index.ts`: Main entry point that exports all public APIs + convenience re-exports
- `benchmarks/history-comparison.ts`: Performance comparison (Queue vs Array)

### Reactive System

The library uses an Realm-based execution model where:

1. **Realms** represent streams of values that can be transformed and combined
2. **Blueprint** provides a synchronous-style DSL where `useX` functions chain Realms via flatMap
3. **Store** manages multiple concurrent values from an Realm, each with its own lifecycle
4. **Resources** handle cleanup in reverse order of creation
5. **Context API** provides type-safe dependency injection using symbols

### API Conventions

#### Realm

- **`Realm<T>`**: Base class for reactive value streams
  - `instantiate(observer: (value: T) => Resource): Resource` - Subscribe to value changes
  - `map<U>(f: (value: T) => U): Realm<U>` - Transform values
  - `flatMap<U>(f: (value: T) => Realm<U>): Realm<U>` - Transform and flatten
  - `filter(predicate: (value: T) => boolean): Realm<T>` - Filter values
  - `merge<U>(other: Realm<U>): Realm<T | U>` - Merge two streams
  - `Realm.pure<T>(value: T)` - Create Realm with single value
  - `Realm.never<T>()` - Create Realm that emits nothing

#### Blueprint

Blueprints are synchronous-style functions that compose Realms. All `useX` functions must be called at the top level of a Blueprint (not inside conditionals, loops, or callbacks).

**Core Blueprint APIs:**

- **`Blueprint.toRealm<T>(blueprint: () => T, userCtx?: UserContext): Realm<T>`**
  - Converts a Blueprint function into an Realm

- **`Blueprint.use<T>(realm: Realm<T>): T`** (also exported as `use()`)
  - Uses an Realm within a Blueprint (creates flatMap chain)
  - Throws `BlueprintChainException` internally for control flow

- **`Blueprint.useEffect<T>(maker: (addResource, abortSignal) => T | Promise<T>): T`** (also exported as `useEffect()`)
  - Executes side effects with proper cleanup
  - Use `addResource()` to register cleanup functions
  - `abortSignal` indicates when the effect is being cancelled
  - Should be used for all I/O, timers, console.log, and other side effects

- **`Blueprint.useTimeout(delayMs: number): void`** (also exported as `useTimeout()`)
  - Pauses Blueprint execution for specified milliseconds

- **`Blueprint.useNever(): never`** (also exported as `useNever()`)
  - Stops Blueprint execution (no values emitted)

- **`Blueprint.useGuard(predicate: () => boolean): void`** (also exported as `useGuard()`)
  - Conditionally continues execution (like filter)

- **`Blueprint.useIterable<T>(iterable: Iterable<T>): T`** (also exported as `useIterable()`)
  - Iterates over values, emitting each one

**Store-related Blueprint APIs:**

- **`Blueprint.toStore<T>(blueprint: () => T): Store<T>`** (also exported as `toStore()`)
  - Create a Store from a Blueprint outside of a Blueprint context
  - This is the main entry point for creating root Stores

- **`Blueprint.useStore<T>(blueprint: () => T): Store<T>`** (also exported as `useStore()`)
  - Create a Store from a Blueprint within a Blueprint context
  - The created Store will be a child of the current Blueprint

- **`Blueprint.useCell<T>(initialValue: T): [Store<T>, (newValue: T) => Promise<void>]`** (also exported as `useCell()`)
  - Create a single-value cell within a Blueprint
  - The setter replaces the current value (releases old, creates new)

- **`Blueprint.usePortal<T>(): [Store<T>, (newValue: T) => void]`** (also exported as `usePortal()`)
  - Create a multi-value portal within a Blueprint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quon-org/quon-ts](https://github.com/quon-org/quon-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
