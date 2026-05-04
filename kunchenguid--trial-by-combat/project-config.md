---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Project

Trial by Combat is a turn-based deterministic 1v1 LLM duel, livestream-ready. The current (and only) mode is **Capture the Relic** on a 9x9 grid. Players are LLM agents that interact via a plain-text HTTP API; spectator and admin views remain browser-based. The server implementation and tests are the source of truth for the player surface.

## Commands

```sh
npm install
npm start                  # node src/server.js, default PORT=4178
npm test                   # node --test, runs all test/*.test.js
npm run test:engine        # engine tests only
node --test test/server.test.js   # single file
node --test --test-name-pattern="creates Center Choke" test/engine.test.js   # single test
npm run build:atlas        # regenerate sprite atlas (see "Sprite atlas" below)
```

There is no bundler, transpiler, linter, or TypeScript. Pure ESM Node + vanilla browser JS.

## URL routes

Browser (spectator and admin only):

- `/?player=spectate` - spectator view
- `/?player=admin` - admin (series length, pause/resume, restart, next game)
- `/?player=1` and `/?player=2` return 404 (player slots are API-only)

WebSocket endpoint `/ws` is for spectator and admin only; a player WS upgrade is rejected.

Player HTTP API:

- `GET /player1` and `GET /player2` - long-poll text view + briefing + DO NEXT block
- `POST /player1/{join,ready,action,leave}` (and `/player2/...`) - JSON bodies, plain-text replies

## Architecture

Two-process boundary: a pure synchronous **game engine** and a thin **HTTP + WebSocket harness** that drives it.

### `src/engine.js` - pure game logic

- Exports: `createGame`, `createSeries`, `resolveTurn`, `validateAction`, `getLegalActions`, `getPlayerView`, `getSpectatorView`, plus `ACTIONS`, `SIDES`, `BOARD_SIZE`, `RULESET_VERSION`.
- All state mutations go through `resolveTurn(game, { blue, red })`, which clones the game (`cloneGame`) and returns `{ game, events, actions, droppedByDamage }`. Never mutate a game object in place outside `resolveTurn` - everything is built around treating game state as immutable from the harness's perspective.
- Resolution order in `resolveTurn` matters and is tested: invalid-action coercion to WAIT → respawn stunned → HEAL → SCAN → dash inventory decrement → PLACE_TRAP (early so opponents stepping into the target this turn trigger it) → 2-step movement (with collision detection between sides) → ATTACK → damage application (GUARD reduces by 2) → forced relic drops on >=3 damage → voluntary DROP_RELIC → knockouts → PLACE_WALL (late so walls can't retroactively block in-flight moves) → auto pickup → win check → turn cap.
- Wall placement runs a path invariant check (`allPathInvariantsHold`) on a cloned trial game so a player can't seal off the relic or either base. BFS via `shortestPath`.
- Sides are fixed for the series: `slotSidesForGame` always returns `{ player_1: 'blue', player_2: 'red' }` regardless of game number. The function exists as a hook in case we re-enable swapping, but today neither slots nor sides flip between games.
- Map constants (`CENTER_CHOKE`) and starting inventory are frozen module locals. To add a new map, parameterize `createGame` rather than mutating these.

### `src/server.js` - HTTP + WebSocket harness

- `createAppServer({ turnSeconds })` returns `{ app, server, listen, close, port }`. Tests use this with a randomly-assigned port and short `turnSeconds`.
- Holds a single in-memory `state` (no DB, no persistence). Phases: `pre_lobby` → `lobby` → `match` → (`game_end` | `series_end`).
- Player slots are HTTP-only. State stores `{ name, ready }` per slot - no socket. The slot is "held" while a name is set; `POST /playerN/leave` clears it (and pauses an active match).
- Long-polling for `GET /playerN` is driven by a per-state `EventEmitter` - `notifyChange(state)` both wakes the long-pollers and broadcasts to spectator/admin WS.
- Turn timer is a single `setTimeout`; on expiry, any side without a `pendingActions` entry is auto-WAIT'd. Pause/resume preserves remaining seconds in `remainingWhenPaused`.
- Validation has two strikes per turn: first invalid action returns 400 with a retry hint; second invalid action this turn locks the side as WAIT.
- Action body translation: HTTP uses uniform `{action, target, intent}`. The harness translates `MOVE`/`DASH` + `target` coord into the engine's directional `MOVE_NORTH` / `DASH_EAST` shape; `PLACE_WALL`/`PLACE_TRAP` keep their target coord; `ATTACK` is untargeted; intent maps to `intent_summary`.
- Spectator/admin still receive `{ type: 'state', role, state }` over `/ws`:
  - Spectator view: `getSpectatorView` with optional X-ray (`set_xray` toggle on the WS).
  - Admin view: full payload + spectator view forced to xray.
- Player view rendering for the API is implemented as text (grid + metadata + DO NEXT) directly in `server.js`.

### `public/` - browser client

- No build step. `index.html` loads Pixi.js from CDN and `app.js` as a module. Asset modules are imported with `?v=...` query strings as cache-busters; bump the version when changing the asset or its consumer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunchenguid/trial-by-combat](https://github.com/kunchenguid/trial-by-combat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
