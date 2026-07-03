---
trigger: always_on
description: A small, focused TypeScript library for **explicit errors as values**, with a
---

# unthrown

A small, focused TypeScript library for **explicit errors as values**, with a
separate **defect channel** for the unexpected.

It exists because the alternatives fall short: `boxed` and `neverthrow` don't
model unexpected errors through a defect channel and don't enforce error
qualification when crossing async boundaries; `effect` is too heavy and conflates
error handling with context, runtime, etc. `unthrown` does one thing.

The name states the concern: ordinary errors are _unthrown_ — returned as values,
not flung up the stack. Only a true defect ever throws (at `unwrap`).

This file is the authoritative spec — the rules _and_ the reasoning behind them.
Keep it in sync with the code as the library evolves (describe what _is_, not what
was planned).

## Thesis (do not drift from these)

1. **`Result<T, E>` where `E` is only the _anticipated_ domain failures.** A
   defect (an unmodeled failure) is the third variant of the `Result` union
   (`{ tag: "Defect" }`), but it **never appears in `E`**. If a failure mode
   appears in `E`, it is by definition modeled and is no longer a defect. The
   defect is matchable like any variant, but you never thread it through your
   domain error type.
2. **No `Option` type.** Absence is expressed with the type system we already
   trust: `T | undefined`, `T | null`, or `Result<T, NotFound>`. Interop with
   nullable third-party APIs goes through `fromNullable`. Do not add `Option`.
3. **Qualification is enforced at every boundary.** `fromPromise` / `fromThrowable`
   take a mandatory `qualify: (cause: unknown, defect) => E | Defect`, where
   `defect` is a helper the boundary **injects** as the second argument (domain
   code never imports it — the qualify-time marker is not a public value). There
   is no path that produces `unknown` in `E`. The boundary forces a triage
   decision. The
   modeled error type is inferred as **`Exclude<R, Defect>`** (where `R` is
   `qualify`'s return type): the `Defect` arm is _subtracted_ from `E`, never
   inferred into it — a defect-only `qualify` yields `E = never`, not
   `E = Defect` (sound because `Defect` is `unique symbol`-branded, so no domain
   error is assignable to it). This is also why **`AsyncResult` combinator
   callbacks are synchronous** — a raw `Promise` may never enter an `AsyncResult`
   method (its rejection would silently become a defect, skipping the triage).
   Async work re-enters only through `fromPromise` / `fromSafePromise` and
   composes via `flatMap`.
4. **`TaggedError` is the error convention** (à la Effect's `Data.TaggedError`):
   a `_tag` discriminant on a class extending `Error`. Core `Result<T, E>` stays
   **generic in `E`** (unconstrained); only the tag-aware utilities require
   `E extends { _tag: string }`.

## Load-bearing runtime invariants (tests must guard these)

- **Throw → defect.** Any value thrown by a callback inside a combinator
  (`map`, `flatMap`, `flatTap`, `bind`, `let`, `mapErr`, `orElse`, `recover`,
  `tap*`, `flatTapErr`, `recoverDefect`) is caught and converted to a `Defect`. Nothing
  escapes a pipeline as a raw throw.
  This is what lets an HTTP adapter do a single `match({ ok, err, defect })`
  with **no surrounding `try/catch`**.
- **A `Defect` flows through every method untouched EXCEPT `match()` and
  `recoverDefect()`.** Therefore `unwrapOr`, `unwrapOrElse`, `getOrNull`,
  `getOrUndefined` still **throw** on a `Defect` — they recover the modeled `Err`,
  never an unmodeled defect (a defect is a bug, not an absent value).
- **`unwrap()` is asymmetric.** On `Err` it throws a `UnwrapError` carrying `E`
  (on both the typed `.error` property and the standard `Error.cause`, so an
  `Error`-typed `E` chains its original stack under "caused by").
  On a `Defect` it **rethrows the original `cause`** (it _panics_) with its
  original stack, so an unhandled defect hits the global handler looking like the
  real failure.
- **`recover` returns `Result<T | U, never>`, and `never` means only the _error_
  channel is empty — a `Defect` can still be present at runtime.** This is the one
  place the type intentionally under-describes the runtime. Do not read `never`
  as "total".
- **An `AsyncResult`'s internal promise NEVER rejects.** Every rejection or
  thrown value is captured as `Err` (via `qualify`) or `Defect`. `await`-ing an
  `AsyncResult` always yields a `Result` and never throws.

## Public surface (implemented in packages/core/src/, split into focused modules)

`Result<T, E>` is a **discriminated union** — `{ tag: "Ok"; value } | { tag:
"Err"; error } | { tag: "Defect"; cause }`, each intersected with the shared
method surface — so it matches **natively** (a `switch` on `tag`, or
`ts-pattern`'s `match(r).with({ tag: "Ok" }, …).exhaustive()`) **and** chains
fluently. The payload is reachable only after narrowing, so "check before you
access" still holds.

`AsyncResult<T, E>` shares that method surface as an awaitable wrapper typed
`Awaitable<Result<T, E>>` — a **success-only thenable**, not a full `PromiseLike`
(its internal promise never rejects, so there is no rejection channel to model).
Its **combinator callbacks are synchronous** (no raw `Promise` — see Thesis #3);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [btravstack/unthrown](https://github.com/btravstack/unthrown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
