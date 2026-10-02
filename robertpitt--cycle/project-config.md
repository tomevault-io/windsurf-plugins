---
trigger: always_on
description: These instructions apply to the whole repository. A more deeply nested `AGENTS.md` may add to or
---

# Cycle agent guidance

These instructions apply to the whole repository. A more deeply nested `AGENTS.md` may add to or
override them for its subtree.

## Shared project context

- This project can contain solo operations, multi-agent rooms, Kanban boards, and project memory.
  Treat them as shared context, but do not assume work is complete without evidence in project
  history or the working tree.
- Preserve unrelated user changes. Keep comments and progress narration concise.

## Effect v4 source of truth

- This repository uses Effect v4 prereleases. Before using an unfamiliar API, check the version in
  the affected package, the installed declarations/source, and the official
  [v4 documentation](https://effect.website/docs/v4). Do not copy v3 APIs or examples from a
  different v4 beta/RC without verifying them against the installed version.
- API names may change during the v4 prerelease. Use the spelling exported by the repository's pin.
  For example, this repository currently exposes `Schema.TaggedErrorClass`; newer v4 documentation
  calls the corresponding API `Schema.TaggedError`. Do not opportunistically migrate between them.
- Treat the compiler and Effect language-service diagnostics as design feedback. Resolve error,
  requirement, and unhandled-Effect diagnostics instead of casting around them.

## Core Effect model and composition

- Read `Effect<Success, Error, Requirements>` literally: the success channel is the result, the
  error channel contains expected recoverable failures, and `Requirements` lists services still
  needed. Preserve all three channels accurately.
- Keep deterministic calculations pure. Use `Effect` for side effects, failure, asynchronous work,
  resource lifetimes, concurrency, scheduling, and dependency access; do not wrap ordinary value
  transformations in `Effect.sync` merely to make them look effectful.
- Prefer Effect ecosystem utilities (`Array`, `Record`, `String`, `Option`, `Match`, `Predicate`,
  `DateTime`, and similar modules) over hand-rolled helpers when they already model the operation.
- Use `Effect.gen` for multi-step workflows. Use ordinary `if`, loops, and local variables inside the
  generator when that is clearer than deeply nested combinators.
- Define functions returning an Effect with `Effect.fn("qualifiedName")`. The name must match the
  function or service method (for example, `"TicketStore.findById"`) so traces and stack frames are
  useful. Use `Effect.fn.Return` when an explicit return type materially improves the contract.
- Pass cross-cutting operators such as `Effect.catch`, `Effect.annotateLogs`, and tracing as extra
  arguments to `Effect.fn`; do not pipe the function returned by `Effect.fn`.
- When a yielded error ends a generator, write `return yield* error` (or
  `return yield* Effect.fail(error)`) so TypeScript knows execution cannot continue.
- For short transformations, use dual APIs in whichever form is clearest: data-first for one
  operation and data-last in a pipeline. Do not use tacit/point-free calls such as `Effect.map(fn)`
  when an explicit `(value) => fn(value)` preserves generics, overloads, stack traces, or intent.
  Avoid `flow` for the same kind of opaque point-free composition.
- Effects are lazy and immutable. Do not introduce a zero-argument `() => Effect` merely for
  laziness. Use `Effect.suspend` only when effect construction itself must occur on every execution,
  to make recursive/circular definitions stack-safe, or to help TypeScript unify a deferred return
  type.
- Combine independent effects with `Effect.all`, `Effect.forEach`, `Effect.zip`, or related
  operators. Execution is sequential unless concurrency is requested; set concurrency deliberately
  and preserve input/result shape.
- Keep `Effect.run*` calls at application or integration edges. Do not run Effects inside services or
  domain logic. Prefer asynchronous execution; reserve `runSync` for effects proven to be fully
  synchronous and use an `Exit` variant when the complete outcome must be inspected.
- Start Node/Bun/browser applications with the platform `runMain` so signals, exit status, fiber
  interruption, and finalizers are handled. Represent a long-running application as a layer and use
  `Layer.launch` when the application lifecycle is naturally layer-shaped.

## Creating Effects and wrapping boundaries

- Use `Effect.succeed` for an already available value and `Effect.fail` for an already available
  expected error. Use `Effect.failSync` only when constructing the error must be deferred.
- Use `Effect.sync` only for non-throwing synchronous side effects. A throw from `sync` becomes a
  defect.
- Use `Effect.try` for synchronous APIs that can throw and `Effect.tryPromise` for Promise APIs that
  can reject. Map `unknown` into a precise domain error at the boundary. Use `Effect.promise` only
  when rejection is genuinely impossible.
- Wrap callback APIs with `Effect.callback`, annotate its success/error types, resume exactly once,
  and return cancellation cleanup or use the supplied `AbortSignal` when the underlying API can be
  interrupted.
- Prefer Effect platform abstractions over wrapping raw Node APIs repeatedly. Boundary adapters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertpitt/cycle](https://github.com/robertpitt/cycle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
