---
trigger: always_on
description: - This is a single-page, vanilla JS P2P shooter (no build step). All source files (`.js`, `.css`, `.html`) are now located in the `src/` folder. The entry point is `src/index.html`, which loads scripts in order: `themes/retro-neon.js` -> `themes/midnight-void.js` -> `themes/sandstorm.js` -> `themes/cyber-sakura.js` -> `themes/index.js` -> `core/ThemeManager.js` -> `constants.js` -> `sound.js` -> `physics.js` -> `renderer.js` -> `network.js` -> `game.js`. Theme files must load before `constants.j
---

# AGENTS Instruction

## Big picture

- This is a single-page, vanilla JS P2P shooter (no build step). All source files (`.js`, `.css`, `.html`) are now located in the `src/` folder. The entry point is `src/index.html`, which loads scripts in order: `themes/retro-neon.js` -> `themes/midnight-void.js` -> `themes/sandstorm.js` -> `themes/cyber-sakura.js` -> `themes/index.js` -> `core/ThemeManager.js` -> `constants.js` -> `sound.js` -> `physics.js` -> `renderer.js` -> `network.js` -> `game.js`. Theme files must load before `constants.js` so theme data is available when the registry is queried.
- Core loop and state live in the `Game` module (IIFE). `Game` owns the tick, input handling, mode switching (local/online host/online guest), and state transitions.
- Networking is host-authoritative. The host runs the canonical physics simulation. **P1 (host's player)**: the host sends its input keys **and authoritative P1 position** to the guest at **60 Hz** (`host_input` message). The guest runs `processRemoteInput(p1)` for responsive bullet creation only; the rendered P1 position is **directly applied** from `hostP1State` every frame (not gated by physics tick). **P2 (guest's player)**: the guest sends its input to the host at **60 Hz** (`input` message); the host processes P2's movement authoritatively and sends back P2's position/direction at **60 Hz** inside the `host_input` message. The guest applies P2 positions directly from `hostP2State` every frame — a "dumb terminal" rendering model that eliminates jitter by simply displaying whatever the host last sent. The guest still handles P2 shooting locally for responsive bullet creation. The host also sends lightweight **authority corrections** at **10 Hz** (`correction` message) containing player health/scores, both player positions, bombs, zombies, health packs, explosions, and game state; correction positions also update `hostP1State`/`hostP2State`. Authority state (health, score, alive, game state) is snapped immediately. Bomb/zombie/health-pack **spawning** only happens on the host; the guest ticks existing entities but receives new ones via corrections. Full state (`broadcastState`) is only sent once after reconnection and also seeds `hostP1State`/`hostP2State`. See `sendHostInput()`/`sendCorrections()`/`applyCorrections()` in `game.js`.
- **Render interpolation** — the game loop uses a fixed 60 Hz physics timestep with `requestAnimationFrame` for rendering. On the **host/local** side, player and bullet positions are interpolated between the previous and current physics state using `alpha = physicsAccumulator / PHYSICS_STEP_MS` to avoid micro-stutter on high-refresh displays. `savePhysicsPositions()` snapshots `prevX/prevY` before each physics step; `interpolateForRender(alpha)` temporarily overwrites entity positions for drawing; `restorePhysicsPositions()` puts them back after rendering. On the **guest** side, player positions are **not** render-interpolated — they are applied directly from host state every frame (the "dumb terminal" model), so save/interpolate/restore only applies to **bullets**. Bombs, explosions, and zombies move slowly enough that jitter is imperceptible without interpolation, so they are not interpolated.

## Architecture and data flow

- All source files are now in the `src/` folder:
  - `constants.js` defines gameplay constants, maze data, and `parseMaze()`. It also owns the global `activeMaze` used by `Physics` and `Renderer`.
  - `physics.js` uses `activeMaze.walls` for collision checks; keep wall rectangles aligned to the maze grid.
  - `renderer.js` renders the maze grid (`activeMaze.grid`) and HUD every frame. It assumes the retro color palette in `COLORS` from `constants.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arifulislamat/p2p-maze-shooter](https://github.com/arifulislamat/p2p-maze-shooter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
