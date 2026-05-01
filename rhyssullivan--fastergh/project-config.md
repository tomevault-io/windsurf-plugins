---
trigger: always_on
description: Prefer Effect primitives over standard JavaScript primitives whenever possible for better type safety, immutability, and functional programming patterns
---


# Effect Primitives Preference

Always prefer Effect's primitives over standard JavaScript primitives whenever possible. This provides better type safety, immutability, error handling, and composability.

## Important: Avoid Shadowing JavaScript Built-in Types

**Never import Effect modules that shadow JavaScript built-in types without aliasing.** Effect provides modules like `Array`, `Number`, `String`, `Boolean`, etc. that would shadow the global JavaScript types if imported directly.

### ❌ Avoid (Shadowing)

```typescript
import { Array, Number, String } from "effect";

// This shadows the global Array, Number, String types!
const items = Array.make(1, 2, 3);
```

### ✅ Prefer (Aliased Import)

```typescript
import { Array as Arr, Number as Num, String as Str } from "effect";

// Use aliased names to avoid shadowing
const items = Arr.make(1, 2, 3);
const sum = Num.sum(1, 2, 3);
```

## Data Structures

### Arrays

- **Prefer `Chunk<A>`** over `Array` when:
  - Performing repeated concatenation operations
  - Working with immutable collections
  - Need structural equality
- **Prefer `Data.array`** for immutable arrays with structural equality
- **Prefer `List<A>`** for linked list operations (efficient prepend/append)
- **When using Effect's `Array` module**, always use aliased imports (e.g., `import { Array as Arr }`) to avoid shadowing the global `Array` type
- **Prefer Effect's `Array` utilities** over standard JavaScript array methods when working with JavaScript arrays:
  - Use `Arr.filter(values, Predicate.isNotNullable)` instead of `values.filter(x => x != null)`
  - Use `Arr.map`, `Arr.filter`, `Arr.find`, etc. with Effect predicates and utilities for better type safety and composability
  - Always import with aliases: `import { Array as Arr, Predicate } from "effect"`
- **When filtering null values after map operations**, **ALWAYS** use `Arr.filter` with `Predicate.isNotNull` instead of manual `.map().filter()` patterns, **even in non-Effect code**:
  - ❌ Avoid: `items.map(x => x?.value ?? null).filter((x): x is NonNullable<typeof x> => x !== null)`
  - ✅ Prefer: `Arr.filter(Arr.map(items, x => x?.value ?? null), Predicate.isNotNull)`
  - This applies to **all code**, not just Effect-based code - use Effect primitives consistently throughout the codebase
  - This provides better type safety and cleaner code without manual type guards

### Maps & Sets

- **Prefer `HashMap<K, V>`** over `Map<K, V>` for immutable key-value pairs
- **Prefer `HashSet<A>`** over `Set<A>` for immutable unique collections
- **Prefer `SortedSet<A>`** when ordering matters
- Use mutable variants (`MutableHashMap`, `MutableHashSet`) only when performance is critical and mutation is localized

### Objects

- **Prefer `Data.struct`** over plain objects for immutable structs with structural equality
- **Prefer `Data.tagged`** for tagged unions/discriminated unions

## Control Flow

### Conditionals

- **Prefer `Effect.if`** for conditional Effect execution
- **Prefer `Match`** module over `switch` statements for pattern matching
- **Prefer `Option.match`, `Either.match`, `Exit.match`** for type-safe pattern matching

### Loops & Iteration

- **Prefer functional methods** (`map`, `filter`, `reduce`, `forEach`) over `for` loops
- **Prefer `Effect.forEach`** for async iteration over collections
- **Prefer `Stream`** over async iterables for lazy async sequences
- **Prefer `Effect.gen`** over `async/await` for Effect-based code

## Error Handling

### Null/Undefined

- **Prefer `Option<A>`** over `null` or `undefined`
- Use `Option.some(value)` and `Option.none` instead of nullable types

### Error Handling

- **Prefer `Either<L, R>`** over throwing errors
- **Prefer `Effect.try`** over try/catch blocks
- **Prefer `Effect.catch`, `Effect.catchAll`, `Effect.catchTag`** for error recovery
- **Prefer `Effect.orElse`, `Effect.orElseFail`** for fallback mechanisms

## Async Operations

### Promises

- **Prefer `Effect`** over `Promise` for async operations
- **Prefer `Effect.promise`** to convert Promises to Effects
- **Prefer `Effect.async`** for callback-based APIs
- **Prefer `Effect.gen`** over `async/await`

### Coordination

- **Prefer `Deferred<A>`** over manual Promise coordination
- **Prefer `Fiber<A>`** for cancellable async operations

## State Management

### Mutable State

- **Prefer `Ref<A>`** over mutable variables in concurrent code
- **Prefer `SynchronizedRef<A>`** for atomic effectful updates
- **Prefer `SubscriptionRef<A>`** for observable state

## Concurrency

### Communication

- **Prefer `Queue<A>`** over manual async queues
- **Prefer `PubSub<A>`** for publish-subscribe patterns
- **Prefer `Channel`** for bidirectional communication

### Coordination

- **Prefer `Semaphore`** for controlling concurrent access
- **Prefer `Latch`** for synchronization barriers
- **Prefer `Effect.all`, `Effect.race`, `Effect.zipPar`** for concurrent operations

## Time & Scheduling

### Time

- **Prefer `Duration`** over raw milliseconds/seconds
- **Prefer `DateTime`** over `Date` for time points
- **Prefer `Effect.delay`** over `setTimeout`

### Scheduling

- **Prefer `Schedule`** for retry/repeat patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RhysSullivan/fastergh](https://github.com/RhysSullivan/fastergh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
