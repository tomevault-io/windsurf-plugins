---
trigger: always_on
description: Reactive signals library with first-class async support and a React integration layer. Monorepo with the core package plus a docs site.
---

# Signalium

Reactive signals library with first-class async support and a React integration layer. Monorepo with the core package plus a docs site.

## Repository structure

```
packages/
  signalium/          Core signals library (the main package)
docs/                 Documentation site (out of scope for typical work)
```

Package manager: **npm** workspaces. Task runner: **turbo**.

## Commands

```sh
# From repo root
npm run test              # all tests via turbo
npm run build             # all packages via turbo
npm run check-types       # tsc --noEmit for all packages
npm run lint              # eslint + prettier

# From packages/signalium
npm test                  # all vitest projects (unit + transform + react)
npm run test:unit         # non-React tests only (node env, fast)
npm run test:react        # browser tests (Playwright via @vitest/browser)
npm run dev:unit          # watch mode for unit tests
npm run check-types       # tsc --noEmit
```

React tests run in a **real browser** (Playwright/Chromium, headless). Unit tests run in Node. Both use vitest.

## Performance-sensitive code

This is a performance-critical library. Follow these guidelines:

- **Prefer class instances over plain objects or arrays** — they have better hidden class optimization in V8.
- **Respect object shaping** — once a class instance is created, do not add or remove properties dynamically. All properties must be declared in the constructor or as class fields so V8 assigns a stable hidden class.
- **Minimize object allocations** — reuse objects where possible, avoid creating temporary objects in hot paths (e.g. inside `runSignal`, `checkSignal`, `dirtySignal`).
- **Bitwise flags over booleans** — `ReactiveSignal.flags` packs state + boolean properties into a single number using `ReactiveFnFlags`. Follow this pattern.

## Global compile-time constant

`IS_DEV` is a global boolean replaced at build time:

- In tests and dev builds: `true`
- In production builds: `false`, all `if (IS_DEV)` blocks are tree-shaken

Declared in `packages/signalium/src/globals.d.ts`. Do NOT import it — it's a bare global.

---

## Package: signalium

### Entry points

| Import path           | Source                   |
| --------------------- | ------------------------ |
| `signalium`           | `src/index.ts`           |
| `signalium/react`     | `src/react/index.ts`     |
| `signalium/config`    | `src/config.ts`          |
| `signalium/utils`     | `src/utils.ts`           |
| `signalium/debug`     | `src/debug.ts`           |
| `signalium/transform` | `src/transform/index.ts` |

In tests, vitest aliases resolve bare `signalium` and `signalium/*` imports directly to the source `.ts` files (see `vitest.config.ts`).

### Core internals (`src/internals/`)

The reactive graph engine. Key files:

| File              | Purpose                                                                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `reactive.ts`     | `ReactiveSignal` class (the computed/derived signal), `ReactiveDefinition`, `ListenerMeta`, `createReactiveSignal`, `createReactiveDefinition` |
| `signal.ts`       | `StateSignal` class (mutable state signal), `notifier()`                                                                                       |
| `async.ts`        | `ReactivePromiseImpl` (async signals), `createRelay`, `createTask`, `createPromise`                                                            |
| `get.ts`          | `getSignal` (reads a signal, establishes dependencies), `checkSignal` (validates dirty state), `runSignal` (recomputes)                        |
| `dirty.ts`        | Dirty propagation: `dirtySignal`, `propagateDirty`                                                                                             |
| `watch.ts`        | Watch/unwatch lifecycle: `watchSignal`, `unwatchSignal`, `activateSignal`, `deactivateSignal`, relay activation                                |
| `scheduling.ts`   | Flush system: `schedulePull`, `scheduleFlush`, `flushWatchers`, `settled()`                                                                    |
| `edge.ts`         | Dependency edges between signals (`Edge`, `PromiseEdge`)                                                                                       |
| `consumer.ts`     | Thread-local `CURRENT_CONSUMER` tracking for automatic dependency registration                                                                 |
| `contexts.ts`     | `SignalScope` (DI-like context system), `context()`, `getContext()`                                                                            |
| `consume-deep.ts` | `CONSUME_DEEP` protocol for deep dependency tracking across the React boundary                                                                 |
| `core-api.ts`     | Public API wrappers: `reactive()`, `reactiveMethod()`, `reactiveSignal()`, `relay()`, `task()`, `watcher()`                                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Signalium/signalium](https://github.com/Signalium/signalium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
