---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm test                                          # run all tests
npx mocha test/all.js --grep "pattern"            # run a single test by name
```

No build or lint step — the library ships as plain ES modules with no transpilation.

## Architecture

**pure-effect** is a zero-dependency effect system for JavaScript implementing the "Functional Core, Imperative Shell" pattern. Business logic returns plain data structures instead of executing side effects, enabling testing without mocks.

### Core abstractions (all in `index.js`)

| Export                                    | Shape                                      | Purpose                                                                                                                                                  |
| ----------------------------------------- | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Success(value)`                          | `{ type: 'Success', value }`               | Wraps a successful result                                                                                                                                |
| `Failure(error, initialInput)`            | `{ type: 'Failure', error, initialInput }` | Short-circuits the pipeline                                                                                                                              |
| `Command(cmdFn, nextFn, meta)`            | `{ type: 'Command', cmd, next, meta }`     | Defers a side effect for the interpreter                                                                                                                 |
| `Ask(nextFn)`                             | `{ type: 'Ask', next }`                    | Reads the `context` passed to `runEffect`; passes it to `nextFn`                                                                                         |
| `Retry(effect, options)`                  | `{ type: 'Retry', effect, options, next }` | Wraps any Effect tree with retry-on-failure semantics; handled natively by the interpreter                                                               |
| `Parallel(effects, next)`                 | `{ type: 'Parallel', effects, next }`      | Runs multiple Effect trees concurrently via `Promise.all`; returns the first Failure by array order if any effect fails; context flows into all branches |
| `effectPipe(...fns)`                      | —                                          | Composes functions into a sequential pipeline via `chain`                                                                                                |
| `runEffect(effect, context, callConfig?)` | async                                      | Interpreter: traverses the effect tree, executes Commands; resolves `Ask` and `Retry`; per-call `callConfig` overrides global defaults                   |
| `configureEffect(options)`                | —                                          | Sets process-wide telemetry hooks (`onStep`, `onRun`, `onBeforeCommand`) and global `retry` defaults; overridden by per-call `callConfig`                |

### Data flow

```
effectPipe(f1, f2, f3)(input)
  → returns tree of Success / Failure / Command / Ask / Retry values
  → f1 runs eagerly here; initialInput is threaded through chain so every
    node in the tree carries it without post-construction mutation

runEffect(tree, context, callConfig?)
  → per-call callConfig merges over global configureEffect defaults
  → executes Commands async, passes results into next(result), repeats
  → resolves Ask by calling next(context), continues
  → resolves Retry via an inner execute() loop (not recursive runEffect),
    so onRun fires exactly once per runEffect call regardless of attempts;
    on exhaustion returns Failure({ retryExhausted: true, lastError, attempts })
  → resolves Parallel by running all effects via Promise.all with the same
    context; if any effect returns Failure, returns the first Failure by
    array index and skips next; otherwise calls next with the array of
    unwrapped success values
  → resolves to final Success or Failure
```

The `chain` combinator (internal) drives composition: `Success` passes its value to the next function, `Failure` short-circuits, `Command` defers execution, `Ask` wraps its continuation so the chain propagates through it, `Retry` wraps its continuation via object spread (same pattern as `Ask`). `chain` accepts an optional `initialInput` parameter; `effectPipe` passes the pipeline's starting value through every `chain` call so that `initialInput` is stamped on all nodes — including mid-pipeline `Failure`s — without mutation. `runEffect` loops through the tree with a `while` loop rather than recursion.

`configureEffect` options (all overridable per-call via `runEffect`'s third argument):

- `onStep` — fires on every Command execution; wraps the `cmd` call (use for per-command tracing)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aycangulez/pure-effect](https://github.com/aycangulez/pure-effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
