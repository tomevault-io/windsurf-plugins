---
trigger: always_on
description: - **Purpose**: Zero-config motion for DOM list changes. Given a parent element, automatically animates its immediate children when they are added, removed, or moved.
---

### Project overview
- **Purpose**: Zero-config motion for DOM list changes. Given a parent element, automatically animates its immediate children when they are added, removed, or moved.
- **Core entry**: `src/index.ts` exports `autoAnimate(el, configOrPlugin)` and types, plus a Vue directive `vAutoAnimate`.
- **Framework adapters**: Thin wrappers providing hooks/directives for Vue, React, Solid, Preact, Angular, Qwik under `src/<framework>/index.ts`.
- **Build**: TS → ESM via Rollup; built artifacts land in `dist/`. Package `exports` map serves `index.mjs` and framework subpaths.

### Mental model (how it works)
- **Observers**
  - MutationObserver on the parent watches childList changes. For each mutation, `getElements` collects the parent and child elements that changed (including removed ones), tagging each with a non-enumerable target marker `__aa_tgt`.
  - ResizeObserver on `document.documentElement` (root) and on each observed element updates cached positions when sizes change.
  - IntersectionObserver is created per element to detect position changes while scrolling; a dynamic `rootMargin` derived from the last known coordinates triggers recalculation. Occasional low-priority polling backs this up.
- **State tracking** (all with WeakMap/WeakSet to avoid leaks)
  - `coords`: last known absolute coordinates adjusted for nearest scrolling ancestor.
  - `siblings`: prev/next references for elements removed from the DOM so they can be temporarily reinserted for exit animations.
  - `animations`: current Web Animations API `Animation` per element.
  - `intersections`, `intervals`, `debounces`: observers/intervals/timers per element.
  - `options`: the per-parent config or plugin; `enabled`: parents with animations enabled.
- **Deciding what to animate**
  - `animate(el)` selects one of: `add`, `remain`, or `remove` based on whether the element has cached coordinates and whether it’s still connected.
  - Flags: `__aa_new` marks newly reinserted nodes to ensure their next cycle is treated as an entry; `__aa_del` marks nodes currently animating out.
- **Effects**
  - Default behavior uses Web Animations API: `add` scales/ fades in; `remain` uses FLIP-style transforms (translate from old to new), also animates width/height deltas respecting `box-sizing`; `remove` temporarily re-inserts the node near its original position, absolutely positions it, and scales/fades out.
  - Plugins: if `config` is a function, it’s treated as an `AutoAnimationPlugin`. Plugin functions return a `KeyframeEffect` or `[KeyframeEffect, { styleReset | false }]`. The library will construct and play an `Animation` from the effect and handle optional style resets.
- **Scroll handling**
  - Coordinates are absolute plus the nearest scrollable ancestor’s offset.
  - During removal at page bottom, the library compensates for scroll jumps by snapshotting pre/post window scroll and performing a manual scroll animation to match the configured duration/easing.
- **Options & accessibility**
  - Defaults: `{ duration: 250, easing: 'ease-in-out' }`.
  - Respects `prefers-reduced-motion: reduce` unless `disrespectUserMotionPreference: true` or a plugin is used.
  - `AnimationController` controls enable/disable per parent.

### Public API surface
- **Default export**: `autoAnimate(el: HTMLElement, config?: Partial<AutoAnimateOptions> | AutoAnimationPlugin): AnimationController`
- **Types**: `AutoAnimateOptions`, `AutoAnimationPlugin`, `AutoAnimationPluginOptions`, `AnimationController`.
- **Vue**: `vAutoAnimate` directive, `autoAnimatePlugin`, `useAutoAnimate` hook.
- **React/Preact**: `useAutoAnimate` hook returns `[refCallback | refObject, setEnabled]`.
- **Solid**: `createAutoAnimate` and `createAutoAnimateDirective`.
- **Angular**: `AutoAnimateDirective` (`[auto-animate]`).
- **Qwik**: `useAutoAnimate` returns `[Signal<HTMLElement>, setEnabledQRL]`.
- **Nuxt**: module that registers the Vue directive and auto-imports `autoAnimate` and `useAutoAnimate`.

### Key invariants and conventions
- Do not mutate children beyond adding style resets for exit animations; remove those styles in `cleanUp`.
- All per-node state is in WeakMaps/Sets to prevent memory leaks; never store strong references to DOM nodes outside these registries.
- Only immediate children of the target parent are animated; traversal utilities (`forEach`) visit the parent first, then its children.
- Always schedule position updates after animations finish (`animation.addEventListener('finish', updatePos)`), and debounce updates using the current duration (or 500ms for plugins).
- When computing size deltas, use `getTransitionSizes` to respect `box-sizing: content-box` padding/borders.
- Tagging properties `__aa_tgt`, `__aa_del`, `__aa_new` are non-enumerable and must be defined via `Object.defineProperty`.
- Core must remain framework-agnostic; adapters import from `../index` only.

### Files and what lives where
- `src/index.ts`: All core logic and types; also exports a Vue directive for convenience.
- `src/debug-utils.ts`: Optional dev-time helpers (e.g., `drawMargins`) to visualize IntersectionObserver margins.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [formkit/auto-animate](https://github.com/formkit/auto-animate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
