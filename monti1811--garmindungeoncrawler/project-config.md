---
trigger: always_on
description: This is a **Garmin Connect IQ watch app** - a turn-based roguelike dungeon crawler for Garmin watches (Venu 2S), written in **Monkey C**. The game features procedurally generated dungeons, combat, inventory management, and persistent save/load functionality.
---

# Dungeon Crawler - AI Coding Agent Instructions

## Project Overview
This is a **Garmin Connect IQ watch app** - a turn-based roguelike dungeon crawler for Garmin watches (Venu 2S), written in **Monkey C**. The game features procedurally generated dungeons, combat, inventory management, and persistent save/load functionality.

## Core Architecture

### App Entry & Navigation Flow
- **Entry point**: `DungeonCrawlerApp.mc` → `getInitialView()` returns main menu by default
- **Navigation pattern**: View/Delegate pairs using `WatchUi.switchToView()` or `WatchUi.pushView()`
- All UI screens follow the pattern: `DC[ScreenName]View` + `DC[ScreenName]Delegate`
- Views extend `WatchUi.View` (or `WatchUi.ViewLoop`), Delegates extend `WatchUi.BehaviorDelegate` (or `Menu2InputDelegate`, `ConfirmationDelegate`)

### Game State Management (Module Pattern)
The codebase uses **global singleton modules** (not classes) for shared state:
- **`$.Game`**: Central game state (depth, player, dungeon, map, turns, time tracking)
- **`$.EntityManager`**: GUID-based entity registry for save/load (all entities register on creation)
- **`$.Items`** / **`$.Enemies`**: Item/enemy factories with player-specific ID generation
- **`$.SaveData`**: Persistent storage wrapper using Garmin's `Storage` API
  - **Compendium tracking**: `discovered_enemies`, `discovered_items` dictionaries for discovered content
- **`$.Settings`**: App settings persistence
- **`$.Quests`**: Quest tracking and progress
- **`$.StepGate`**: Real-world step counter integration for turn gating
- **`$.SimUtil`**, **`$.MathUtil`**, **`$.Pathfinder`**, **`$.Log`**, **`$.ElementUtil`**: Utility modules

Access global modules via `$.ModuleName.function()`. The `$` is a Monkey C global namespace accessor.

### Type System Patterns
```monkey-c
typedef Point2D as [Numeric, Numeric];  // Used throughout for positions
```
Common types: `Point2D`, `ResourceId` (for Rez assets), `PropertyKeyType`/`PropertyValueType` (for Storage API).

### Entity System
- **Base class**: `Entity` (has `guid`, `energy`, save/load methods)
- **Hierarchy**: `Entity` → `Player`, `Enemy`, `NPC`, `Item` subclasses
- All entities **must call `register()`** to be added to `EntityManager` (assigns GUID for persistence)
- Items/Enemies use factory pattern: `Items.init(player_id)` / `Enemies.init(player_id)` create unique instances per save
- **Player Classes**: Moved to `source/Engine/Entities/Player/Classes/` - Warrior, Mage, Archer, Paladin, God, Nameless
  - Each class has unique stats, starting equipment, and leveling progression
  - Paladin: Tank class with high constitution and wisdom, starts with shield
- **Compendium System**: Automatically tracks discovered enemies/items when encountered
  - Enemies tracked on death via `$.SaveData.discovered_enemies[id]`
  - Items tracked on pickup/interaction via `$.SaveData.discovered_items[id]`

### Map & Dungeon Structure
- **Dungeon**: Grid of rooms (`Array<Array<String?>>`), rooms identified by name strings like `"room_2_3"`
- **Room persistence**: Each room saved individually to `Storage` by name, loaded on-demand
- **Room**: Contains `Map`, `_items`, `_enemies`, `_npcs` dictionaries keyed by `Point2D`
- **Map**: 2D array of `Tile` objects, tiles have `type` (enum: `WALL`, `PASSABLE`, `STAIRS`) and `content` (Entity?)
  - Sparse storage: Only non-empty tiles stored in `_tiles` array, `null` for empty/wall tiles
  - Sentinel pattern: `_null_tile` returned for out-of-bounds/missing tiles (treated as walls)
  - Key methods: `getTile(x, y)`, `setContent(pos, entity)`, `isPosFree(pos)`, `isInBound(pos)`
  - Content layer: Each tile can hold one entity reference (player, enemy, item, NPC)
- **Tile**: Minimal data structure with `type`, `x`, `y`, `content`, `player` flag
  - Types: `EMPTY`, `WALL`, `PASSABLE`, `STAIRS`
  - Content is transient - not saved (entities saved separately by Room)
- **Map data**: Stored in `$.Game.map` as 5-element tuples: `[room_name, connections, size, visited, flags]`
  - Flags: Array of `Point2D?` for special locations (stairs, merchant, boss, quest giver)

### Resource Management (Rez System)
- **Drawables**: `$.Rez.Drawables.LauncherIcon`, sprites, UI hints (defined in `resources/drawables/drawables.xml`)
- **Fonts**: `$.Rez.Fonts.small`, `Rez.Fonts.test` (defined in `resources/fonts/fonts.xml`)
- **Strings**: `$.Rez.Strings.AppName` (defined in `resources/strings/strings.xml`)
- Load resources: `WatchUi.loadResource($.Rez.Fonts.small)` or `new WatchUi.Bitmap({:rezId=>$.Rez.Drawables.Player})`
- Python helpers in `helpers/`: `split_images.py` (slice spritesheets), `create_font.py` (bitmap font generation)

### Save/Load Architecture
**Critical**: Save data is **nested and distributed**:
1. Game state stored in `SaveData` module, persisted to `Storage.setValue(chosen_save, data)`
2. Each **room saved individually** to Storage by key (e.g., `"room_2_3"`)
3. Player, EntityManager, Quests, Game all implement `save()` → `Dictionary` and `load(data)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Monti1811) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
