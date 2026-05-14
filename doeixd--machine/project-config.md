---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`@doeixd/machine` is a minimal, type-safe state machine library for TypeScript built on mathematical foundations and centered around **Type-State Programming**.

**Core Philosophy:**
- **Type-State Programming First**: States are represented as TypeScript types, not strings. The compiler catches invalid state transitions at compile-time, making illegal states unrepresentable.
- **TypeScript as Safety Net**: The type system prevents entire categories of bugs—no runtime checks needed for state validity.
- **Minimal Primitives**: Provide only essential building blocks; everything else is built on top.
- **Flexibility Over Prescription**: Immutability by default but not enforced; supports functional, OOP, and factory patterns.
- Every transition is a pure function that returns a new machine instance (though mutability is technically allowed for flexibility)

## Development Commands

```bash
# Install dependencies
npm install

# Build the library (uses pridepack)
npm run build

# Type checking
npm run type-check

# Watch mode for development
npm run watch

# Run tests
npm test

# Clean build artifacts
npm run clean
```

## Common Pitfalls and Testing Patterns

### Creating Test Fixtures

**DON'T** use `createMachine(context, {})` - the empty object causes TypeScript errors:
```typescript
// ❌ WRONG - causes type errors
const machine = createMachine({ status: 'idle' }, {});
```

**DO** use one of these approaches:

1. **Use `createContext()` helper** (recommended for simple test fixtures):
```typescript
// ✅ CORRECT - type-safe and clean
const machine = createContext({ status: 'idle' });
```

2. **Use plain objects** when you need specific structure:
```typescript
// ✅ CORRECT - explicit structure
const machine = { context: { status: 'idle' as const } };
```

3. **Use `createMachine()` with actual transitions**:
```typescript
// ✅ CORRECT - for real machines
const machine = createMachine({ count: 0 }, {
  increment() { return createMachine({ count: this.context.count + 1 }, this); }
});
```

### Why This Matters

The `createMachine` function has multiple overloads that expect either:
- A factory function: `(transition) => Record<string, Function>`
- Transition functions: `Record<string, Function>`
- Another machine to copy transitions from

An empty object `{}` doesn't satisfy any of these overloads, causing type errors. The `createContext()` helper provides a clean, type-safe way to create minimal machine-like objects for testing.

## Architecture

### Core Module Structure

The codebase is organized into focused, single-purpose modules:

1. **`src/index.ts`** - Core library exports
    - `Machine<C>` and `AsyncMachine<C>` types - fundamental machine shapes
    - `createMachine()` / `createAsyncMachine()` - factory functions for creating machines
    - `state()` - smart function that chooses between `createMachine` or `createFunctionalMachine` based on arguments
    - `runMachine()` - the runtime "interpreter" for async machines with event dispatch
    - `setContext()` - immutably update machine context
    - `overrideTransitions()` / `extendTransitions()` - compose/decorate machines
    - `createMachineBuilder()` - create factory functions from template machines
    - `MachineBase` - optional OOP base class
    - Type utilities: `Context<M>`, `Event<M>`, `Transitions<M>`, etc.

2. **`src/primitives.ts`** - Type-level metadata DSL
   - Annotation functions like `transitionTo()`, `guard()`, `guardAsync()`, `invoke()`, `action()`, `describe()`
   - These are runtime no-ops but add type-level metadata via branded types
   - Used by static analysis tools to extract formal statecharts
   - The `META_KEY` symbol brands types with `TransitionMeta`

3. **`src/extract.ts`** - Build-time statechart extraction
   - Uses `ts-morph` to perform static analysis on machine source code
   - Reads type-level metadata from primitives to generate JSON statecharts
   - Compatible with Stately Viz and XState tooling
   - Available as separate entry point: `import { extractMachine } from '@doeixd/machine/extract'`
   - NOT included in main bundle (tree-shakeable, ~8 kB separate chunk)
   - CLI tool: `npx tsx scripts/extract-statechart.ts --config .statechart.config.ts`

4. **`src/utils.ts`** - High-level utilities
   - `isState()` - type-safe state checking guard
   - `createEvent()` - type-safe event factory for `runMachine`
   - `mergeContext()` - shallow merge context updates
   - `pipeTransitions()` - compose multiple transitions sequentially
   - `logState()` - debugging helper ("tap" function)

5. **`src/react.ts`** - React integration (draft)
   - `useMachine()` hook for React integration
   - Connects machine state changes to React re-renders

6. **`src/devtools.ts`** - Browser DevTools integration (draft)
   - `connectToDevTools()` - connects to browser extension
   - Sends state changes and events to visualization tools

7. **`src/generators.ts`** - Generator-based composition
   - `run()` - Execute generator-based state flows
   - `step()` - Yield state and receive next

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doeixd/machine](https://github.com/doeixd/machine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
