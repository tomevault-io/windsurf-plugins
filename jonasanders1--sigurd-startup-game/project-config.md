---
trigger: always_on
description: You are working on **Sigurd Startup**, a 2D arcade platformer shipped as an npm package (a Web Component + React) and embedded in a separate React landing page with Stripe + Firebase. This document is the contract for how you operate in this repo. Read it every session.
---

# CLAUDE.md — Sigurd Startup

You are working on **Sigurd Startup**, a 2D arcade platformer shipped as an npm package (a Web Component + React) and embedded in a separate React landing page with Stripe + Firebase. This document is the contract for how you operate in this repo. Read it every session.

---

## 1. Project overview

- **Game:** 2D arcade platformer (Bomb Jack–inspired), 800×600 fixed playfield, 60 FPS target
- **Genre mechanics:** bomb collection in sequence + coin power-ups + monster avoidance
- **Engine:** a **custom canvas 2D engine** — a hand-rolled game loop (`GameLoopManager`) rendering to a `<canvas>` via `RenderManager`. **There is no Phaser and no Matter.js.** Do not add them.
- **Language:** TypeScript, `strict: true`
- **Distribution:** published to npm as `sigurd-startup`, consumed by a separate React landing page
- **Delivery form:** a Web Component `<sigurd-startup>` (registered in `src/game-wrapper.tsx`) that mounts a React tree; also importable in React
- **Host integration:** the landing page provides a `window.sigurdGame` bridge (balance, credits, Stripe, Firebase, leaderboard)
- **Authoritative design source:** `game-specs.md` at the repo root — the source of truth for all mechanics, constants, timings, and behaviors. Code and commits cite it as `game-specs §X`.

> ⚠️ There is a second, **divergent** spec copy at `specs/game-spec.md.md` (note the doubled extension). It is stale. Treat `game-specs.md` (root) as canonical until the two are reconciled.

## 2. Before you write code

**Always, in this order:**

1. Read the relevant section of `game-specs.md` before implementing any gameplay system. If the spec is ambiguous, **ask** — do not guess.
2. Check `DECISIONS.md` for architectural choices already made. Do not re-litigate them. (Note: its oldest entries describe an abandoned Phaser design — see the correcting entry at the top.)
3. Propose a plan before writing code for any task that touches more than two files. Wait for approval.
4. When implementing, reference the spec section by number in commit messages (e.g., "implement coin spawn conditions per game-specs §7").

If you find yourself guessing at a constant, a timing, or a behavior, stop and check the spec.

## 3. Repo layout

```
src/
  managers/         # cross-entity coordinators — the heart of the game loop
    GameLoopManager.ts    # requestAnimationFrame loop, fixed-step orchestration
    RenderManager.ts      # draws the playfield to <canvas>
    CollisionManager.ts   # custom AABB collision + resolution
    GameManager.ts / GameStateManager.ts   # top-level orchestration + state
    bombManager.ts, coinManager.ts, coinPhysics.ts
    ScalingManager.ts, PowerUpManager.ts, ScoreManager.ts, AudioManager.ts
    MonsterFactory.ts, MonsterBehaviorManager.ts, monster-movements/
    Optimized{Spawn,Respawn}Manager.ts, InputManager.ts, LevelManager.ts, ...
  stores/           # Zustand stores — the single source of truth for state
    game/           # scoreStore, stateStore, levelStore
    entities/       # playerStore, coinStore, monsterStore
    systems/        # audioStore, inputStore, renderStore, balanceStore, tuningStore
    gameStore.ts    # root store composition
  entities/         # plain game-object classes (Player, Bomb, monster subclasses)
  lib/              # pure logic + helpers (gravityLUT, scoringUtils, gameBridge, logger, ...)
  components/       # React UI: menus/HUD/overlays positioned over the canvas
    menu/menus/     # StartMenu, PauseMenu, BonusScreen, GameOverScreen, ...
  config/           # tunable constants, grouped by concern (see §4)
  types/            # interfaces.ts, enums.ts, constants.ts (deprecated shim)
  editor/           # in-repo level/tuning editor (React)
  maps/             # map definitions
  tutorials/        # tutorial mission definitions
  game-wrapper.tsx  # Web Component registration (customElements.define)
  index.ts          # npm package entry point (non-React exports + side-effect registration)
game-specs.md       # design spec (READ THIS) — canonical
DECISIONS.md        # architectural log (append-only)
CLAUDE.md           # this file
```

- **The canvas renders the playfield only.** Menus, HUD chrome, settings, countdown, bonus screen are React components in `src/components/` positioned over the canvas.
- **Game logic lives in managers and pure `lib/` functions, not React.** Managers own the loop and coordinate; `lib/` holds pure helpers; React owns chrome.

## 4. Naming and style rules

- Files: `kebab-case.ts` for logic (e.g., `coin-physics`), but note much of the existing tree uses `camelCase.ts` (`bombManager.ts`) and `PascalCase.ts` for classes (`ScalingManager.ts`). **Match the convention of the directory you're editing.**
- Classes: `PascalCase`. Methods/variables: `camelCase`.
- Constants: `UPPER_SNAKE_CASE`, grouped by concern in **`src/config/*.ts`** (e.g., `config/coins.ts`, `config/scoring.ts`, `config/game.ts`). `src/types/constants.ts` is a **deprecated** backwards-compat shim (`GAME_CONFIG`) — new code should import from `@/config`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonasanders1/sigurd-startup-game](https://github.com/jonasanders1/sigurd-startup-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
