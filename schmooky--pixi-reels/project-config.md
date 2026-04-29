---
trigger: always_on
description: Instructions for AI agents (Claude, Codex, Cursor, etc.) and human contributors working in this repo. Read the whole file before touching anything. It is short on purpose.
---

# AGENTS.md

Instructions for AI agents (Claude, Codex, Cursor, etc.) and human contributors working in this repo. Read the whole file before touching anything. It is short on purpose.

---

## 1. What this repo is

`pixi-reels` is a **reel engine for PixiJS v8**. It animates slot symbols moving on reels and emits typed events describing that lifecycle. One monorepo with three surfaces:

```
packages/pixi-reels/   <- the published library (npm: pixi-reels)
apps/site/             <- the docs site (pixi-reels.dev, not published)
examples/              <- three example apps + shared reference code (not published)
```

The library builds to two entry points:
- `pixi-reels` — core (reel set, phases, symbols, events, testing, debug).
- `pixi-reels/spine` — Spine 2D symbols with the Bonbon animation vocabulary.

Everything else — cheats, cascade loop, seeded RNG, mock server, Spine loaders — lives in `examples/shared/`. It is reference code, copy-pasteable, **not library API**.

---

## 2. What this repo is NOT

Read [ADR 007 — Scope](./docs/adr/007-scope.md) for the full list. The short version:

| Out of scope | Why | Where it belongs |
|---|---|---|
| Win detection (lines, clusters, scatters) | Game-specific rules | Consumer code / server |
| Paytable math | Regulated money math | Server |
| Multiplier / RTP math | Same | Server |
| RNG / outcome selection | `setResult(grid)` is the inbound interface | Server, or `CheatEngine` in examples |
| Audio | Every exit path emits events; wire audio to those | Consumer audio layer |
| Bonus state machines (FS, Hold & Win) | The library provides the primitives; the state machine is game code | Consumer code; see `ScatterFsDemo.tsx` for a primitive shape |
| UI / HUD | The library is a `PIXI.Container`. You place it. | Consumer UI |
| Accessibility | Canvas has no ARIA surface; consumer's DOM UI owns a11y | Consumer DOM |
| i18n | No user-facing strings in the library | Consumer |

**If a request, PR, or feature idea lives in the right column, decline it. Point the asker at the corresponding ADR or `docs/BEST_PRACTICES.md`.**

---

## 3. Repo layout (mental model)

```
pixi-reels/
├── packages/pixi-reels/           the library
│   ├── src/
│   │   ├── core/                  ReelSet, Builder, Reel, Viewport, Motion, StopSequencer
│   │   ├── spin/                  SpinController, phases (Start/Spin/Anticipation/Stop), spinning modes
│   │   ├── symbols/               ReelSymbol (abstract) + Sprite / AnimatedSprite / Spine / Headless
│   │   ├── speed/                 SpeedManager, speed profiles
│   │   ├── frame/                 FrameBuilder, middleware pipeline, RNG
│   │   ├── spotlight/             SymbolSpotlight (win animations)
│   │   ├── events/                typed EventEmitter, event type definitions
│   │   ├── utils/                 Disposable, TickerRef
│   │   ├── testing/               FakeTicker, HeadlessSymbol, createTestReelSet, harness
│   │   ├── spine/                 SpineReelSymbol (Bonbon vocabulary) -> subpath export
│   │   ├── debug/                 debugSnapshot, debugGrid, enableDebug
│   │   └── index.ts               the only public barrel
│   └── tests/                     vitest (unit + integration)
├── examples/
│   ├── shared/                    cheats, cascade loop, seeded RNG, spine loaders, mock server
│   ├── classic-spin/              5x3 line pays
│   ├── cascade-tumble/            6x5 cascade
│   ├── hold-and-win/              hold-and-win respin
│   └── assets/                    prototype-symbols sprite atlas
├── apps/site/                     Astro + shadcn docs site
├── docs/
│   ├── adr/                       architecture decision records
│   └── BEST_PRACTICES.md          developer guide
├── scripts/                       release + size + fancy-unicode guard
└── .github/                       workflows + templates
```

If your change would create a file outside the tree above, stop and think. There is probably a better home.

---

## 4. Load-bearing invariants

Break any of these and the build, the tests, or real consumers break. They are enforced by lint guards, tests, or code review.

- **No default exports.** Always named. Tree-shaking depends on this.
- **`.js` extensions in imports.** Even from `.ts`. Node ESM resolution requires it.
- **Only `src/index.ts` is a barrel.** Subdirectories do not re-export. The Spine subpath barrel (`src/spine/index.ts`) is the single exception.
- **`Disposable` on anything that allocates.** `destroy()` + `isDestroyed`. Wired into `ReelSet.destroy()`'s cascade.
- **`TickerRef`, never `ticker.add()` directly.** The wrapper tracks callbacks and auto-removes on destroy.
- **Events use colon namespacing**: `spin:start`, `spin:reelLanded`, `speed:changed`. Never camelCase, never dot-separated.
- **Per-survivor fall distance in cascades.** `tumbleToGrid` must not move a cell whose computed fall distance is zero. Enforced by `tests/integration/cascadeLoop.test.ts`. See [ADR 010](./docs/adr/010-cascade-physics.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schmooky/pixi-reels](https://github.com/schmooky/pixi-reels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
