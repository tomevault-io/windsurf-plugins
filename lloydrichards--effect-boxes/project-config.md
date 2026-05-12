---
trigger: always_on
description: > **Note:** This file is the authoritative source for coding agent instructions.
---

# AGENTS.md

> **Note:** This file is the authoritative source for coding agent instructions.
> If in doubt, prefer AGENTS.md over README.md.

## 🚦 Quick Reference

- **Install dependencies:** `bun install`
- **Run all tests:** `bun run test`
- **Type check:** `bun run type-check`
- **Lint & format:** `bun run lint` / `bun run format`
- **Validate docs:** `bun run docs:check`
- **Run scratchpad:** `bun run scratch`
- **Test a single file:** `bun run test tests/box.test.ts`
- **Search code:** `rg "pattern"`

---

This file provides comprehensive guidance for coding agents when working with
TypeScript and Effect.js in this Box layout library.

## Core Development Philosophy

### KISS (Keep It Simple, Stupid)

Simplicity should be a key goal in design. Choose straightforward solutions over
complex ones whenever possible. Simple solutions are easier to understand,
maintain, and debug.

### YAGNI (You Aren't Gonna Need It)

Avoid building functionality on speculation. Implement features only when they
are needed, not when you anticipate they might be useful in the future.

### Design Principles

- **Functional Composition**: Build complex behavior by composing simple, pure
  functions
- **Immutability**: All operations return new values, never mutating existing
  ones
- **Type Safety**: Leverage TypeScript's type system for compile-time
  correctness
- **Effect Integration**: Use Effect's utilities (`pipe`, `Equal`, `Hash`) for
  enhanced functionality
- **Single Responsibility**: Each function should have one clear purpose

## 🧱 Project Structure & Library Architecture

This is a **functional library** designed to be either published on NPM or
included in a monorepo. The architecture follows Effect.js patterns with pure
functional composition.

### Directory Structure

```txt
src/
├── Box.ts          # Core Box data type and operations
├── Annotation.ts   # Text annotation system
└── Ansi.ts         # Terminal rendering with ANSI codes
tests/
├── box.test.ts     # Pure function tests (regular vitest)
├── ansi.test.ts    # Integration tests
└── *.test.ts       # Additional test suites
scratchpad/         # Development playground
└── index.ts        # Live examples and experimentation
```

### Library Design Patterns

- **Pure Functions**: All Box operations are pure transformations
- **Functional Composition**: Use `pipe()` for chaining operations
- **Effect Integration**: Implement `Pipeable`, `Equal`, and `Hash` interfaces
- **Immutable Data**: Return new instances, never modify existing ones

## 🎯 Effect.js Patterns for Pure Functional Libraries

This library focuses on **pure functional composition** rather than effectful
operations. However, we leverage Effect's utilities for enhanced type safety and
composition.

### Core Effect Utilities Used

#### 1. Pipeable Interface

All Box types implement `Pipeable` for fluent composition:

```typescript
import { pipe } from "effect";
import * as Box from "./src/Box";

// Fluent composition with pipe
const layout = pipe(
  Box.text("Hello World"),
  Box.moveRight(5),
  Box.moveDown(2),
  Box.alignHoriz(Box.center1, 20)
);

// Alternative method chaining (when Pipeable is implemented)
const layout2 = Box.text("Hello World")
  .pipe(Box.moveRight(5))
  .pipe(Box.moveDown(2))
  .pipe(Box.alignHoriz(Box.center1, 20));
```

#### 2. Equal and Hash Interfaces

For structural equality and efficient comparisons:

```typescript
import { Equal, Hash } from "effect";

// Boxes implement Equal for value-based comparison
const box1 = Box.text("hello");
const box2 = Box.text("hello");
console.log(Equal.equals(box1, box2)); // true

// Hash enables efficient Set/Map operations
const boxSet = new Set([box1, box2]); // Contains only one item
```

#### 3. Dual Functions

For flexible parameter ordering:

```typescript
import { dual } from "effect/Function"

// Supports both data-first and data-last usage
export const moveRight = dual<
  (n: number) => (self: Box) => Box,
  (self: Box, n: number) => Box
>(2, (self, n) => /* implementation */)

// Usage patterns:
Box.moveRight(box, 5)        // data-first
pipe(box, Box.moveRight(5))  // data-last
```

### Pure Function Composition Patterns

#### Function Organization

```typescript
// ✅ Prefer pure, composable functions
export const text = (content: string): Box => /* ... */
export const emptyBox = (rows: number, cols: number): Box => /* ... */
export const hcat = (boxes: Box[], alignment: Alignment): Box => /* ... */

// ✅ Use pipe for complex transformations
const createTable = (data: string[][]) =>
  pipe(
    data,
    Array.map(row => pipe(
      row,
      Array.map(Box.text),
      boxes => Box.hcat(boxes, Box.left)
    )),
    boxes => Box.vcat(boxes, Box.top)
  )
```

#### Error Handling in Pure Functions

```typescript
// ✅ Use Option for potentially missing values
import { Option } from "effect";

export const safeText = (content: string | null): Option.Option<Box> =>
  content === null ? Option.none() : Option.some(Box.text(content));

// ✅ Use Either for validation
import { Either } from "effect";

export const validateDimensions = (
  rows: number,
  cols: number
): Either.Either<Box, string> =>
  rows < 0 || cols < 0
    ? Either.left("Dimensions must be non-negative")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lloydrichards/effect-boxes](https://github.com/lloydrichards/effect-boxes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
