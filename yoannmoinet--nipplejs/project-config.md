---
trigger: always_on
description: NippleJS is a vanilla virtual joystick library for touch-capable interfaces. Zero dependencies, TypeScript-first, supports mouse and touch input.
---

# CLAUDE.md — NippleJS

## Project Overview

NippleJS is a vanilla virtual joystick library for touch-capable interfaces. Zero dependencies, TypeScript-first, supports mouse and touch input.

**Repository:** `github.com/yoannmoinet/nipplejs`
**License:** MIT

## Monorepo Structure

Yarn 4 workspaces (`"packageManager": "yarn@4.6.0"`). Run `yarn install` from root.

```
packages/
├── nipplejs/     # Core library (TypeScript, Rollup build)
├── docs/         # Documentation website (Astro 6, Tailwind 4)
├── tests/        # E2E (Playwright) and unit (Jest) tests
├── tools/        # Build/version management CLI (Clipanion)
└── assets/       # Logo images
```

## Key Commands

```bash
yarn build              # Build nipplejs library (Rollup)
yarn lint               # ESLint (flat config, ESLINT_USE_FLAT_CONFIG=true)
yarn test:unit          # Jest unit tests (196 tests)
yarn test:e2e           # Playwright e2e tests (50 tests)
yarn typecheck          # TypeScript type checking
yarn workspace @nipple/docs dev    # Astro docs dev server (localhost:4321)
yarn workspace @nipple/docs build  # Build static docs site
```

## Architecture — Core Library

### Class Hierarchy

```
Super (event system, DOM binding, logging)
├── Factory (singleton, manages all collections, document-level events)
├── Collection (manages joysticks in a zone, returned by nipplejs.create())
└── Joystick (individual joystick instance, DOM + position + direction)
```

### Event Flow

Joystick → Collection (bubbled) → Factory (bubbled). Events are space/comma-separated strings. Handler signature: **single argument** `(evt) => { evt.type, evt.data }`.

**IMPORTANT:** v1 uses a single `evt` argument, NOT `(evt, data)`. The old v0 two-argument signature is gone.

### Key Files

- `packages/nipplejs/src/index.ts` — Public API: `create()`, `factory`, `setLogLevel()`, `getLogLevel()`
- `packages/nipplejs/src/Collection.ts` — processOnStart/Move/End, reposition(), ResizeObserver
- `packages/nipplejs/src/Joystick.ts` — buildEl(), start(), end(), computeDirectionAndTriggerEvents(), triggerDirectionEvents()
- `packages/nipplejs/src/Super.ts` — on/off/trigger event system, logLevel, bindEvt/unbindEvt
- `packages/nipplejs/src/types.ts` — All TypeScript interfaces
- `packages/nipplejs/src/constants.ts` — Event bindings (pointer/touch/mouse), modes. SSR-safe (`typeof window` guards).
- `packages/nipplejs/src/utils.ts` — processEvents(), distance(), angle(), etc.

### Important Behaviors

- **`move` event fires continuously** on every pointermove, not just on direction changes.
- **`color` option** accepts a string OR `{ front: string, back: string }` object. CSS `background` property is used, so gradients, images, and `url()` work.
- **`baseDelta`** is included in move event data when `follow: true`. It contains the per-frame joystick base displacement.
- **`reposition()`** is a public method on Collection. Called automatically via ResizeObserver on the zone. Also refreshes `factory.scroll`.
- **`logLevel`** defaults to `'warning'`. Set via `nipplejs.setLogLevel()`. `log()` only fires at `debug` level.
- **Zone position warning** — Collection constructor warns if zone has `position: static`.
- **Pointer events** are the primary binding (`pointerdown`, `pointermove`, `pointerup`). Falls back to touch/mouse.
- **`preventDefault()` only on move events** — not on `pointerdown`, to avoid breaking multitouch. The zone sets `touch-action: none` and `user-select: none` instead.
- **SSR-safe** — `window` access in `constants.ts` is guarded with `typeof window !== 'undefined'`.

## Architecture — Docs Website

Astro 6 standalone (no Starlight) with Tailwind CSS 4 via `@tailwindcss/vite`.

### Visual Theme: "Aurora Neon"

- Dark base `#08081a` with 48px grid overlay
- Animated gradient blobs (indigo, cyan, pink) with 80px blur
- Frosted glass surfaces (`.glass` class)
- Cursor-following grid highlight via CSS mask (disabled on mobile)
- Monospace accents (JetBrains Mono)

### Game Demos

5 interactive canvas games, each showcasing a nipplejs option:

| Game | File | Option |
|------|------|--------|
| Neon Snake | `neon-snake.ts` | `mode: 'static'` |
| Asteroid Dodge | `asteroid-dodge.ts` | `lockX: true` |
| Dual-Stick Arena | `dual-stick-arena.ts` | multitouch (two `create()` calls) |
| Space Observatory | `space-observatory.ts` | `follow: true` + `baseDelta` |
| Space Drift | `space-drift.ts` | `restJoystick: false` |

#### Game Architecture

- Games use `offsetWidth`/`offsetHeight` for canvas sizing (NOT `getBoundingClientRect()` which is affected by CSS transforms).
- **Mobile performance:** `shadowBlur` is disabled on mobile (`isMobile` flag). Logo particles and grid highlight are disabled on touch devices.
- **Firefox:** 1.3x `speedScale` multiplier to compensate for slower Canvas2D frame rates.
- **`speedScale`**: computed from canvas diagonal relative to 800px reference, so fullscreen games feel proportional.
- **Particle effects:** Explosion on enemy kill (dual-stick, asteroid-dodge), consumption sparkle on orb/target/waypoint collection (snake, observatory, space-drift). Screen shake and flash on impact.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoannmoinet/nipplejs](https://github.com/yoannmoinet/nipplejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
