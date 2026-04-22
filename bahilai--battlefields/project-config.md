---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BattleFields is a hex-based tactical game built with Phaser 3. It features turn-based combat between green and red armies on a hex grid battlefield with terrain, static weapons, dice mechanics, and a general/company system.

## Running the Game

This is a client-side only Phaser game. To run:
1. Serve the project with any local web server from the root directory
2. Open `index.html` in a browser
3. The game uses ES6 modules, so a web server is required (not `file://`)

No build process, package.json, or npm commands exist - this is vanilla JavaScript.

## Architecture

### Core Systems (src/core/)

**HexGrid** - Converts between world coordinates and hex grid (q,r) coordinates. Uses offset coordinates with odd-row stagger. Key methods:
- `toWorld(q, r)` - hex coords to pixel position
- `fromWorld(x, y)` - pixel position to hex coords
- `neighbors(q, r)` - returns array of valid neighbor hex coordinates
- `hexDistance(q1, r1, q2, r2)` - cube distance between hexes

**TerrainMap** - Extracts terrain types from the Tiled hex layer. Terrain types:
- `void` - impassable
- `water` - impassable (GIDs: 1,7,8,9,12,14,15)
- `forest` - movement cost 2 (GID: 11)
- `trench` - movement cost 1, tanks cannot stop here (GIDs: 3221225489-91, 17-19)
- `grass` - movement cost 1 (GID: 16)

Static weapons on separate layers: `machine-gun` and `antitank-gun`

**Pathfinder** - BFS-based movement calculation. Takes units array as `getOccupiedSet` callback. Special rules:
- Tanks crossing trenches need >1 movement remaining and cannot end on trenches
- Considers terrain costs and occupied hexes
- Returns Map of reachable hexes with `costLeft` for each

**CombatSystem** - Handles attacks and damage. Damage model:
- First hit: `ready` → `wounded` (plays damaged animations, no tint)
- Second hit on infantry: `wounded` → `dead` (plays death animation)
- Second hit on tank: `wounded` → `dead` (shows static dead texture)
- Dead units have `attacks=0`, `movesLeft=0`, sprite non-interactive

**Unit** - Base unit class with:
- `side`: 'green' or 'red'
- `kind`: 'infantry' or 'tank'
- `state`: 'ready', 'wounded', or 'dead'
- Movement with tweened animation based on direction
- Directional animations: infantry has left/right, tanks have left/right/up/down
- Separate animation sets for normal and wounded states

**UnitFactory** - Creates Unit instances with scene and grid context

**Hud** - Bottom UI bar (280px high) showing:
- General portrait (156x156px, left side)
- Two company icons (artillery, supply) - 156x156px each
- Info panel (turn, unit stats, event log)
- Help button (yellow '?' under info panel)

**Dice** - Physics-based dice rolling system:
- Draggable dice with swipe-to-throw mechanic
- Real-time physics simulation: gravity, velocity, friction, bounce
- Drag strength determines throw velocity (max 2000 units/sec)
- Collision detection with screen boundaries
- Random value changes during roll, settles on final value
- Auto-returns to home position after 1 second
- Roll determines active company (1-3: artillery, 4-6: supply)
- Each round: roll selects general/colonel (values 1-6 determine different colonel types)
- Physics updates in `_updatePhysics(time, delta)` via scene events
- Callbacks: `onDiceResolved(value)`, `onCompanyUsed(kind)`

### Scene Structure (src/scenes/)

**BattleScene** - Main game scene. Initialization order critical:
1. Load assets (background, tilemap JSON, tilesets, sprite frames)
2. Create tilemap with layers: hexes, bases, flags, machine-gun, antitank-gun
3. Calculate dynamic scale to fit screen (720x1280 portrait, 280px HUD reserved)
4. Apply tileoffset compensation for red bases layer (+16x, -7y scaled)
5. Initialize HexGrid, UnitFactory, TerrainMap
6. Create Hud and Dice (before pathfinder/combat)
7. Wire callbacks between Dice ↔ Hud
8. Initialize Pathfinder and CombatSystem with units array
9. Spawn initial units
10. Register input handlers

Turn system:

**Round phases (each round):**
1. **Initiative** (only Round 1) - Dice roll determines who goes first for entire game
2. **Commander Hiring** (every round) - Both players roll to get colonel with bonuses
3. **Company Activation** (skipped in first day of Round 1) - Roll for artillery or supply
4. **Unit Actions** - Players alternate 5 actions each (one unit per action)
5. **Night Phase** - Roll determines Rest (company activation) or Night Battle (repeat day)

- **Initiative**: Determined only once in Round 1. Winner gets first turn for the entire game.
- **Commander Hiring**: Happens EVERY round. Each player rolls for new colonel (1-6 = different bonuses)
- **Manual turn switching**: Players click End Turn button ("▶") to finish their turn (no auto-switching)
- `currentPlayer`: 'green' or 'red'
- Units of current player can be selected and moved/attacked
- **Unit actions per turn**:
  - Can move ONCE (remaining movement points are lost after first move, tracked by `hasMoved` flag)
  - Can attack multiple times (based on unit type: infantry=2, tank=1, machinegun=4)
  - Movement and attacks can be combined in same turn

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bahilai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
