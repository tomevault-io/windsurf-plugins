---
trigger: always_on
description: This file is the behavioral contract for AI agents writing applications with
---

# AGENTS.md — Effection agent contract

This file is the behavioral contract for AI agents writing applications with
Effection.

Agents must not invent APIs, must not infer semantics from other ecosystems, and
must ground claims in the public API.

If you are unsure whether something exists, consult the API reference:
https://frontside.com/effection/api/

## Core invariants (do not violate)

### Operations vs Promises

- **Operations** are lazy. They execute only when interpreted (e.g. `yield*`,
  `run()`, `Scope.run()`, `spawn()`).
- **Promises** are eager. Creating a promise (or calling an `async` function)
  starts work; `await` only observes completion.
- You must not claim that a promise is "inert until awaited".
- You must not use `await` inside a generator function (`function*`). Use
  `yield*` with an operation instead (e.g. `yield* until(promise)`).

### Structured concurrency is scope-owned

- Scope hierarchy is created automatically by the interpreter; application code
  should not manage scopes manually.
- "Lexical" in Effection: scope hierarchy follows the lexical structure of
  operation invocation sites (e.g. `yield*`, `spawn`, `Scope.run`), not where
  references are stored or later used.
- Work is owned by **Scopes**.
- When a scope exits, all work created in that scope is halted.
- References do not extend lifetimes. Returning a `Task`, `Scope`, `Stream`, or
  `AbortSignal` does not keep it alive.

### Effects do not escape scopes

- Values may escape scopes.
- Ongoing effects must not escape: tasks, resources, streams/subscriptions, and
  context mutations must remain scope-bound.

## Operations, Futures, Tasks

### Operation

- An `Operation<T>` is a recipe for work. It does nothing by itself.
- Operations are typically created by invoking a generator function
  (`function*`).

### Future

- A `Future<T>` is both:
  - an Effection operation (`yield* future`)
  - a Promise (`await future`)

### Task

- A `Task<T>` is a `Future<T>` representing a concurrently running operation.
- A task does not own lifetime or context; its scope does.

## Entry points and scope creation

### `main()`

- You should prefer `main()` when writing an entire program in Effection.
- Inside `main()`, prefer `yield* exit(status, message?)` for termination; do
  not call `process.exit()` / `Deno.exit()` directly (it bypasses orderly
  shutdown).

### `exit()`

- `exit()` is an operation intended to be used from within `main()` to initiate
  shutdown.

### `run()`

- You may use `run()` to embed Effection into existing async code.
- `run()` starts execution immediately; awaiting the returned task only observes
  completion.

### `createScope()`

- You must not use `createScope()` for normal Effection application code.
- You may use `createScope()` only for **integration** between Effection and
  non-Effection lifecycle management (frameworks/hosts/embedders).
- You must observe `destroy()` (`await` / `yield*`) to complete teardown.
  Calling `destroy()` without observation does not guarantee shutdown
  completion.

### `useScope()`

- Use `yield* useScope()` to capture the current `Scope` for integration (e.g.
  callbacks) and re-enter Effection with `scope.run(() => operation)`.

## `spawn()`

**Shape (canonical)**

```ts
const op = spawn(myOperation); // returns an OPERATION
const task = yield * op; // returns a TASK (Future) and starts it
```

**Rules**

- `spawn()` does not start work by itself. Yielding the spawn operation starts
  work.
- Yielding `spawn()` does not guarantee that the child has reached any
  particular point in its body before the parent continues.
- A spawned task must not outlive its parent scope.

## `Task.halt()`

**Rules**

- `task.halt()` returns a `Future<void>`. You must observe it (`await` /
  `yield*` / `.then()`), or shutdown is not guaranteed to complete.
- `halt()` represents teardown. It can succeed even if the task failed.
- If a task is halted before completion, consuming its value (`yield* task` /
  `await task`) fails with `Error("halted")`.

## Scope vs Task (ownership)

| Concept | Owns lifetime | Owns context |
| ------- | ------------: | -----------: |
| `Scope` |            ✅ |           ✅ |
| `Task`  |            ❌ |           ❌ |

## Context API (strict)

**Valid APIs**

- `createContext<T>(name, defaultValue?)`
- `yield* Context.get()`
- `yield* Context.expect()`
- `yield* Context.set(value)`
- `yield* Context.delete()`
- `yield* Context.with(value, operation)`

**Rules**

- You must treat context as scope-local. Children inherit from parents; children
  may override without mutating ancestors.
- You must not treat context as global mutable state.

## `race()`

**Rules**

- `race()` accepts an array of operations.
- It returns the value of the first operation to complete.
- It halts all losing operations.

## `all()`

**Rules**

- `all()` accepts an array of operations and evaluates them concurrently.
- It returns an array of results in input order.
- If any member errors, `all()` errors and halts the other members.
- If you need all results regardless of success or failure, use `allSettled()`
  instead of wrapping each member in railway-style results.

## `allSettled()`

**Rules**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thefrontside/effection](https://github.com/thefrontside/effection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
