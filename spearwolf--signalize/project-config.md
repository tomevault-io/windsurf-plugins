---
trigger: always_on
description: Architecture and conventions reference for AI coding agents. For Claude Code-specific operational guidance see `CLAUDE.md`. For human contributor docs see `CONTRIBUTING.md`.
---

# AGENTS.md — `@spearwolf/signalize`

Architecture and conventions reference for AI coding agents. For Claude Code-specific operational guidance see `CLAUDE.md`. For human contributor docs see `CONTRIBUTING.md`.

## What it is

Framework-agnostic signal/effect/memo/link library. Synchronous reactivity. Built on `@spearwolf/eventize` for all internal pub/sub.

- Runtime: ESM-only, Node `>=24.13`, targets ES2023, `sideEffects: false`
- TypeScript v5, `strict: true` **but `strictNullChecks: false`** (intentional — don't "fix" it)
- Peer dep: `@spearwolf/eventize ^5.0.0`
- Two entry points: `.` (`src/index.ts`) and `./decorators` (`src/decorators.ts`)

## Core concepts

| Concept | Created via | Purpose |
| --- | --- | --- |
| Signal | `createSignal()` | Reactive value; reads inside an effect register a dependency |
| Effect | `createEffect()` | Function that auto-reruns when tracked signals change |
| Memo | `createMemo()` | Cached derived signal — internally a signal driven by a high-priority effect |
| Link | `link()` | Explicit one-way data flow between signals (or signal → callback) |
| SignalGroup | `new SignalGroup()` | Lifecycle bundle — destroy group → destroys all attached signals/effects/links |
| Object Signal | `@signal` decorator + `findObjectSignal*()` | Signal stored on a host object, retrievable by property name |

## Architecture

### Eventize pub/sub

`@spearwolf/eventize` is **synchronous** (unlike Node's EventEmitter): `emit()` calls all subscribers inline before returning. All internal communication uses it.

| Function | Use here |
| --- | --- |
| `eventize(obj)` | Make object event-capable (used on global queues, `EffectImpl`, `SignalGroup`) |
| `emit(obj, event, ...args)` | Dispatch — signal value changes, lifecycle events |
| `on(obj, event, [priority,] callback)` | Subscribe; higher priority runs first |
| `once(obj, event, callback)` | One-shot subscription (cleanup hooks) |
| `off(obj, [listener])` | Unsubscribe (used in destroy paths) |

### Global event buses (`src/global-queues.ts`)

| Queue | Carries |
| --- | --- |
| `globalSignalQueue` | Signal value changes: `emit(queue, signalId, newValue)` |
| `globalEffectQueue` | Effect lifecycle: `$createEffect`, `$destroyEffect`, `RECALL` |
| `globalDestroySignalQueue` | Signal destruction (cleanup signal) |
| `globalEffectCalledQueue` | Batch deduplication tracking |

### Key symbols (`src/constants.ts`)

| Symbol | Use |
| --- | --- |
| `$signal` | Get internal `SignalImpl` from `Signal` wrapper |
| `$effect` | Get internal `EffectImpl` from `Effect` wrapper |
| `RECALL` | Event triggering effect re-execution |
| `$createEffect`, `$destroyEffect` | Effect lifecycle events |
| `$destroySignal` | Signal destruction event |

### Priorities

Effects subscribe to signals with a numeric priority — **higher runs first**.

- Memos: `Priority.C` = 1000 (`createMemo.ts:59`)
- Effects: `0` (`EffectImpl.ts:112`, `options.priority ?? 0`)

### Dependency tracking flow

```
1. effect.run()
2. push effect onto globalEffectStack
3. callback executes
4. signal.get() inside callback
5. signal calls getCurrentEffect()?.whenSignalIsRead(signalId)
6. effect subscribes: on(globalSignalQueue, signalId, priority, RECALL, this)
7. ...later, signal value changes:
8. emit(globalSignalQueue, signalId, newValue)
9. RECALL handler fires → step 1
```

Subscribe-on-read happens inside `EffectImpl.whenSignalIsRead` (single subscription per signalId per run); cleanup happens before each rerun and on destroy.

### Batching

`batch(callback)`:

1. Creates `Batch` instance, sets it as current context
2. `effect.run()` enqueues into a priority-ordered queue instead of running
3. Batch end → drains queue, each effect runs at most once

### Other context modes

| Mode | Function | Effect |
| --- | --- | --- |
| Quiet | `beQuiet(fn)` | Inside `fn`, signal `set()` does **not** notify dependents |
| Hibernate | `hibernate(fn)` | Suspends all reactive context state during `fn` |
| Untracked read | `value(signal)` or `signal.value` | Read without registering as dependency |
| Forced notify | `touch(signal)` | Emit change without value change |

## Source file map

| File | Responsibility |
| --- | --- |
| `index.ts` | Public API exports for `.` |
| `decorators.ts` | `@signal`, `@memo` (TC39 standard decorators) — separate `./decorators` entry |
| `constants.ts` | Symbols (`$signal`, `$effect`, `RECALL`, `$createEffect`, `$destroyEffect`, `$destroySignal`) |
| `types.ts` | Public TypeScript interfaces |
| `Signal.ts` | `Signal<T>` class — thin wrapper around `SignalImpl` |
| `createSignal.ts` | `SignalImpl`, `createSignal`, `destroySignal`, `isSignal`, `muteSignal`, `unmuteSignal`, `getSignalsCount`, internal `writeSignal`, `signalImpl` |
| `Effect.ts` | `Effect` class — wrapper around `EffectImpl` |
| `EffectImpl.ts` | Core dependency tracking + rerun logic; `EffectOptions` interface |
| `effects.ts` | `createEffect`, `getEffectsCount`, `onCreateEffect`, `onDestroyEffect` |
| `createMemo.ts` | `createMemo` — wraps signal + high-priority effect |
| `link.ts` | `link`, `unlink`, `getLinksCount` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spearwolf/signalize](https://github.com/spearwolf/signalize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
