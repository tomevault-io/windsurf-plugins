---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

《圈地运动》(Enclosure Movement) — a local 2-player territory-capture game built with vanilla HTML5 Canvas + ES6 classes. No frameworks, no bundlers, no package.json at root. Open `index.html` directly in a browser to run local multiplayer.

Online multiplayer (room-based, central server) is **partially implemented** — client-side networking (`NetworkManager`, `RemotePlayer`) and the Node.js server skeleton exist, but the full state-sync pipeline is still being wired up.

Recent performance work replaced most per-frame vector drawing with **pre-rendered sprite images** (`/image/*.png`) loaded by `ImageLoader`, plus distance-squared collision math and batch trail trimming. See `plans/optimization-images.md` for the sprite list and generation prompts.

## How to Run

```bash
# Local game — just open index.html in a browser, no build step.

# Online multiplayer server (also serves static game files on port 3000):
cd server
npm install
npm start        # production — visit http://localhost:3000
npm run dev      # development (--watch auto-restart)
# → WebSocket: ws://localhost:3000
# → Health check: http://localhost:3000/health
# → The server doubles as a static file server — no separate npx serve needed.
```

There are no tests, no linting, no build pipeline.

## Script Loading Order (Critical)

`index.html` loads scripts in a specific sequence. **Load order matters** — each file depends on ones loaded before it:

1. `js/Config.js` — game constants (depends on nothing)
2. `js/modes/Flag.js`, `js/modes/Survive.js`, `js/modes/DayNight.js` — mode logic
3. `js/objects/Player.js`, `js/objects/RemotePlayer.js`, `js/objects/Territory.js`, `js/objects/UIManager.js`, `js/objects/ItemManager.js`, `js/objects/Color.js`
4. `msgpack-lite` CDN (external, for binary WebSocket messages)
5. `js/core/InputHandler.js`, `js/core/ImageLoader.js`, `js/core/Renderer.js`, `js/core/NetworkManager.js`, `js/core/GameEngine.js`, `js/core/main.js`

**`ImageLoader.js` must load before `Renderer.js`** — the Renderer reads `window.imageLoader` to pick sprite vs. procedural fallback. If adding a new JS file, insert it at the correct position in this chain. A file cannot reference classes defined in files loaded after it.

## Architecture

### SPA Page Navigation

The game uses a single-page app pattern: all "pages" are `<div>` elements in `index.html` with class `page`. Navigation toggles via `page.classList.add('active')` / `remove('active')`. Page navigation functions (`showMainMenu()`, `showOnlineLobby()`, `startGame()`, etc.) are **global functions** defined in `js/core/main.js` and called via `onclick` attributes in the HTML. There is no router — this is direct DOM manipulation.

### CSS Organization

`css/main.css` imports all modules via `@import`:

- `css/bases/` — base, background, components, buttons, forms
- `css/pages/` — color-picker, rules, games, game-ui, overlays, online-lobby

Each page/overlay has its own CSS file. When adding a new UI page, add its CSS in `css/pages/` and import it in `main.css`.

### Game Loop (GameEngine.gameLoop)

Uses a **fixed timestep** pattern:
- Physics runs at 60 FPS (`PHYSICS_TIMESTEP = 16.67ms`), max 5 frame skips
- Rendering is decoupled — variable frame rate with an interpolation factor
- `fixedUpdate(deltaTime)` is the authoritative tick; `render(interpolation)` is visual only

### State Machine

`gameState` transitions: `menu` → `countdown` (3s) → `playing` → `paused` / `gameOver`

### GameEngine Dual Init Pattern

`GameEngine` has two initialization paths:
- **`init(canvas)`** — local 2-player mode. Creates renderer, item manager, territory canvas, static/dynamic offscreen canvases, and sets `networkMode = 'local'`.
- **`initOnline(canvas, networkManager, localPlayerId)`** — online mode. Sets `networkMode = 'online'`, `isServerAuthoritative = true`. The game loop switches to `updateOnline()` which sends inputs to the server and interpolates remote players instead of running local collision detection.

### InputHandler Dual Mode

- **Local mode**: Processes two sets of key bindings (`Config.KEY_BINDINGS.player1` = WASD, `player2` = Arrow keys) simultaneously. Both players' inputs update both Player objects directly.
- **Online mode**: `setLocalBindings(bindings)` locks to a single binding set (typically player1/WASD). The local player's input is sent to the server; remote players are driven by server state, not keyboard.

### Canvas Compositing (3 Layers)

The Renderer uses three offscreen canvases composited onto the main canvas each frame, **bottom → top**:

1. **Territory canvas** — filled enclosed polygons; updated only when `territoryChanged === true`
2. **Static canvas** — spawn points, items, obstacles, barriers, flags; redrawn every 3 frames or when `staticNeedsUpdate === true`
3. **Dynamic canvas** — trails + player tanks + AI enemies; redrawn **every frame** when `dynamicNeedsUpdate === true`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yao1145/quandiyundong](https://github.com/yao1145/quandiyundong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
