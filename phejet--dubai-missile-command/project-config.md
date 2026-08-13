---
trigger: always_on
description: Canvas-based missile defense game built with React + Vite.
---

# Dubai Missile Command

Canvas-based missile defense game built with React + Vite.

## Quick Start

```bash
npm install
npm run dev          # starts dev server (usually http://localhost:5173)
npx vite build       # production build to dist/
```

When doing local verification, if you stop the dev server for testing or debugging, start it again before finishing and confirm the local URL.
After implementing a feature or bug fix, proactively check whether the dev server is already running. If it is not running and browser verification could matter, start `npm run dev` yourself and report the active local URL before finishing.

## Browser Smoke Tests

Use the maintained Playwright smoke suite for browser boot/input/shop-flow checks:

```bash
npx playwright test e2e/smoke.spec.ts
```

For the full browser E2E suite:

```bash
npm run test:e2e
```

These tests boot their own production preview server via `playwright.config.ts`, so they do not require `npm run dev` first.

## Running the Bot

The Playwright bot (`play-bot.ts`) auto-plays the game for testing.

```bash
# 1. Start the dev server first
npm run dev

# 2. Run the bot against the active local URL (opens a visible Chromium window)
GAME_URL=http://localhost:5173/dubai-missile-command/ npx tsx play-bot.ts
```

The bot reads game state via `window.__gameRef`, calculates leading shots, prioritizes threats, buys upgrades in the shop, and avoids hitting friendly F-15s.

## Headless Simulation

Run games headlessly for testing and bot tuning.

```bash
# Run a single headless game with determinism check
npx tsx src/headless/sim-runner.ts [seed]

# Record best game as a replay file
npx tsx src/headless/record.ts [--seed=N] [--tries=1000] [--out=replay.json]

# Play a replay in the browser (requires dev server running)
npx tsx play-replay.ts replay.json
```

### Bot training

Use the `/train-bot` skill to benchmark and tune the bot. It runs batch games via `src/headless/train.ts` and analyzes results to tune `src/headless/bot-config.json`.

### Key files

- `src/game-sim.ts` — extracted game loop (spawning, upgrades, auto-systems)
- `src/game-logic.ts` — constants, collision, injectable seeded RNG
- `src/replay.ts` — replay runner (action-log based deterministic replay)
- `src/headless/sim-runner.ts` — headless game runner
- `src/headless/bot-brain.ts` — parameterized bot targeting/firing logic
- `src/headless/bot-config.json` — tunable bot parameters
- `src/headless/train.ts` — batch training benchmark (multi-worker)
- `src/headless/game-worker.ts` — worker thread for parallel game execution

### Replay system

Replays record bot actions (fire coordinates + shop purchases at tick numbers) with a seeded RNG. Drop a `replay.json` onto the game canvas or use `window.__loadReplay(data)` in the console. During replay, the shop UI shows for 2 seconds between waves and a toast displays what the bot purchased.

## Architecture

Current runtime architecture is split across:

- `src/game-sim.ts` — simulation and gameplay state mutation
- `src/art-render.ts` — shared art primitives and prebaked sprite generation
- `src/pixi-render.ts` — Pixi frame composition and render-time asset caching
- `src/game.ts` — runtime controller that advances sim and calls renderers

Start with [`docs/README.md`](./docs/README.md) for the current documentation map.
Focused breakdowns:

- [`docs/render-split-analysis.md`](./docs/render-split-analysis.md)
- [`docs/runtime-controller.md`](./docs/runtime-controller.md)
- [`docs/game-state-contract.md`](./docs/game-state-contract.md)
- [`docs/spawn-commander-reference.md`](./docs/spawn-commander-reference.md)
- [`docs/upgrades-shop-progression.md`](./docs/upgrades-shop-progression.md)
- [`docs/replay-system.md`](./docs/replay-system.md)
- [`docs/headless-bot-workflow.md`](./docs/headless-bot-workflow.md)

### Key constants

- `CANVAS_W=900`, `CANVAS_H=1600`, `GROUND_Y=1530`
- `BURJ_X=460`, `BURJ_H=340`
- `LAUNCHERS` at x: 60, 560, 860

### Game state (`gameRef.current`)

- `missiles`, `drones`, `interceptors`, `explosions`, `particles`, `planes`
- `defenseSites[]` — physical upgrade structures enemies can destroy
- `launcherHP[3]` — each launcher starts with 1 HP (upgradable to 2 with Launcher Kit L2), destroyed = can't fire
- `upgrades{}` — wildHornets, roadrunner, flare, ironBeam, phalanx, patriot, burjRepair, launcherKit, emp
- `stats{}` — missileKills, droneKills, shotsFired (shown on game over)

### Targeting system (`pickTarget`)

- 30% chance enemies target the Burj directly, aiming at a random spot on the tower trunk (`getBurjBodyAimPoint`)
- Otherwise, targets defense sites and alive launchers; picks closest-to-missile-spawn 70% of the time, second-closest 30%
- Top-spawning missiles within 200px of their target are offset 300-500px for interceptable angles
- The Burj hit area covers the tower body only — the pedestal band at the base (bottom `BURJ_PEDESTAL_FRAC` of `BURJ_H`) is excluded, and ground impacts never damage the Burj

### Upgrade systems

| Upgrade           | What it does                                                                                                 |
| ----------------- | ------------------------------------------------------------------------------------------------------------ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phejet/dubai-missile-command](https://github.com/phejet/dubai-missile-command) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
