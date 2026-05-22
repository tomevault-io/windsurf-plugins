---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project identity

This workspace ships **one** publishable library under three names. Don't conflate them.

| Surface | Identifier |
|---|---|
| npm | `solidjs-motion` |
| JSR | `@solidjs-motion/motion` |
| GitHub | `solidjs-motion/motion` |
| Internal workspace package | `packages/motion/` |

The repo is named `motion` (not `solidjs-motion`) because the org name is already `solidjs-motion` — repeating it would be redundant. Inside the repo, the term **`motion`** refers to this library. The npm package `motion` (the framework-agnostic animation engine we wrap) is always written out explicitly as "the `motion` npm package" or "upstream `motion`" to avoid collision.

## Architecture

The library is layered:

```
                createMotion(el, opts)        ← imperative primitive
                          ↑
                    useMotion(opts)           ← canonical public API
                          ↑
            ┌─────────────┴─────────────┐
            ↑                           ↑
       <motion.div>                motion(MyButton)
       (proxy, Phase 4)            (HOC, Phase 4)
```

`createMotion` is the imperative primitive that takes an element + reactive options. `useMotion` wraps it and returns a callable that merges user props with motion's (style merge, ref composition, SSR-friendly inline style). Phase 4 lands the JSX-level wrappers; the plan's original `<Motion as="...">` proposal was dropped in favor of the proxy-plus-HOC pattern.

**Reactivity opt-in via function form:**

```tsx
useMotion({ animate: { x: 100 } })             // static
useMotion(() => ({ animate: { x: x() } }))     // reactive — signals tracked
```

`initial` is captured once at construction. `animate`/gesture targets track Solid signals through the inner `createEffect`. MotionValues in target values take a separate subscription path (see "MV-in-target" below).

**SSR pattern:**

- Server: `useMotion` returns props with a deterministic inline `style` from `targetToStyle(initial)` plus `data-motion-hydrated=""`. HTML ships with the initial style.
- Browser: first paint matches server (no flicker).
- Hydration: ref runs, `createMotion` sees `initialAppliedBySSR: true`, skips the initial-style application, runs `animate()` to the target.

Hydration matching requires `targetToStyle` to be **pure and deterministic** — same input must produce byte-identical output on server and client.

**Build pipeline (ship-source pattern):**

- Library is published with both `src/` (TS source) and `dist/` (compiled JS + `.d.ts`).
- The `"solid"` export condition in `packages/motion/package.json` is listed **before** `"types"` so dev-mode TS resolution reads source directly (avoids stale-dist shadowing). External consumers without the `solid` condition fall through to `types`.
- Consumers using `vite-plugin-solid` (anyone in the Solid ecosystem) resolve to raw source and Babel-transform it themselves with `babel-preset-solid`. This is the pattern `@solidjs/router` and `solid-motionone` use; it's correct for SSR/hydration semantics.
- Inside this monorepo, the same condition powers HMR-through-source: edits to `packages/motion/src/*.ts` are picked up live by `examples/basic` without rebuilding the library.

## The `MotionValueAccessor` callable hybrid

Every Phase 1 primitive that produces an animatable value returns a **`MotionValueAccessor<T>` = `MotionValue<T> & (() => T)`** — a Proxy that's callable as a Solid Accessor AND has every upstream MotionValue method.

```tsx
const x = createMotionValue(0)

x()                                  // Solid-tracked read (use in JSX, createEffect)
x.get()                              // sync, untracked read (motion engine uses this)
x.set(100)                           // imperative write (fires the change subscription)
x.jump(50)                           // hard set, no animation
x.getVelocity()                      // upstream MotionValue method
x.on("change", cb)                   // raw subscription
useMotion({ animate: { x: x } })     // motion engine sees .getVelocity → treats as MV
animate(x, 200)                      // motion engine accepts as MV target
```

**Why this works**: motion's `isMotionValue` is duck-typed (`Boolean(v && v.getVelocity)`), and motion never uses `instanceof MotionValue` in its JS source. The Proxy forwards `.getVelocity` to the underlying MV, so `isMotionValue(callable)` returns true. `useMotion`'s `splitTarget` then routes the hybrid down the MotionValue subscription path.

| Primitive | Returns |
|---|---|
| `createMotionValue<T>(initial)` | `MotionValueAccessor<T>` |
| `createTransform<I,O>(input, range, output, opts?)` | `MotionValueAccessor<O>` |
| `createSpring(source, opts?)` | `MotionValueAccessor<number>` |
| `createTime()` | `MotionValueAccessor<number>` (driver) |
| `createVelocity(source)` | `MotionValueAccessor<number>` |
| `createTemplate\`...\`` | `MotionValueAccessor<string>` |
| `createScroll(opts?)` → 4 fields | each `MotionValueAccessor<number>` |
| `createInView(ref, opts?)` | `Accessor<boolean>` — boolean, not a motion value |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solidjs-motion/motion](https://github.com/solidjs-motion/motion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
