---
trigger: always_on
description: This rule defines the logical organization structure for the Don't Starve Together core systems documentation based on the comprehensive analysis in [core-systems-structure.md](mdc:dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/core-systems-structure.md).
---

# DST Core Systems Directory Structure

This rule defines the logical organization structure for the Don't Starve Together core systems documentation based on the comprehensive analysis in [core-systems-structure.md](mdc:dst-api-webdocs/dst-api-webdocs/dst-api-webdocs/core-systems-structure.md).

## Core Systems Organization

The `/docs/game-scripts/core-systems/` directory is organized into 12 main categories to reflect logical learning progression and system dependencies:

### 1. System Core
**Location:** `docs/game-scripts/core-systems/system-core/`
**Purpose:** Engine foundation and runtime systems that power the entire game
**Contents:**
- `engine/` - Game initialization, core functions, physics, map utilities
- `runtime/` - Update loops and runtime execution systems

### 2. Fundamentals
**Location:** `docs/game-scripts/core-systems/fundamentals/`
**Purpose:** Core programming foundation - OOP, entities, utilities, and basic systems
**Contents:**
- `core/` - OOP foundation, entity system, metaclass, standard components
- `utilities/` - Math, string, vector utilities, simulation helpers
- `actions/` - Player interaction and action systems
- `ai-systems/` - AI brains, behavior trees, state machines

### 3. Game Mechanics
**Location:** `docs/game-scripts/core-systems/game-mechanics/`
**Purpose:** Core gameplay systems that define how players interact with the game
**Contents:**
- `cooking/` - Recipe system, food mechanics, spiced variants
- `crafting/` - Recipe definitions, tech tree, filtering systems
- `containers/` - Storage and inventory systems
- `achievements/` - Achievement system and progression tracking
- `special-events/` - Seasonal events and limited-time content

### 4. Character Systems
**Location:** `docs/game-scripts/core-systems/character-systems/`
**Purpose:** Player character management, customization, and progression
**Contents:**
- `core/` - Character utilities, profiles, death tracking
- `customization/` - Skins, clothing, visual customization
- `emotes/` - Player expressions and social features
- `progression/` - Skill trees, character advancement, WX-78 modules
- `speech/` - Character dialogue (Wilson as master template)

### 5. World Systems
**Location:** `docs/game-scripts/core-systems/world-systems/`
**Purpose:** World generation, terrain management, and entity systems
**Contents:**
- `generation/` - World generation, settings, presets, regrowth
- `tiles-terrain/` - Ground tiles, terrain properties, noise functions
- `entities/` - Prefab system, world entities, skin mappings
- `ocean/` - Ocean mechanics and water systems

### 6. Networking Communication
**Location:** `docs/game-scripts/core-systems/networking-communication/`
**Purpose:** Multiplayer networking, chat, and server communication
**Contents:**
- `networking/` - Core networking, RPC, shard management
- `chat-commands/` - Chat system, user commands, voting
- `multiplayer/` - Server preferences, MOTD, popup management

### 7. User Interface
**Location:** `docs/game-scripts/core-systems/user-interface/`
**Purpose:** UI management, input handling, and visual effects
**Contents:**
- `frontend/` - Core UI, data grids, loading tips, split screen
- `input/` - Input handling, controls, haptic feedback
- `graphics/` - Visual effects, lighting, particles, post-processing
- `typography/` - Font system, animation easing

### 8. Game Configuration
**Location:** `docs/game-scripts/core-systems/game-configuration/`
**Purpose:** Game settings, balance, modes, and statistics
**Contents:**
- `settings/` - Configuration, constants, tuning, global overrides
- `modes/` - Game modes, logic, event systems
- `stats/` - Statistics tracking, item blacklists

### 9. Development Tools
**Location:** `docs/game-scripts/core-systems/development-tools/`
**Purpose:** Developer utilities, debugging, and testing tools
**Contents:**
- `debugging/` - Debug commands, tools, menu systems, inspection
- `console/` - Console commands, hot reloading
- `profiling/` - Performance monitoring, code profiling
- `utilities/` - Development utilities, serialization, error handling

### 10. Localization Content
**Location:** `docs/game-scripts/core-systems/localization-content/`
**Purpose:** String management, translation, and game content
**Contents:**
- `strings/` - String management, localization, POT generation
- `translation/` - Translation system, monkey curse utilities
- `content/` - Game content, theatrical systems, guitar tabs

### 11. Data Management
**Location:** `docs/game-scripts/core-systems/data-management/`
**Purpose:** Save systems, asset management, and data utilities
**Contents:**
- `saves/` - Save game system, upgrades, shard management
- `assets/` - Asset loading, JSON support, audio preloading
- `utilities/` - Task scheduling, platform configuration, legacy redirects

### 12. Mod Support
**Location:** `docs/game-scripts/core-systems/mod-support/`
**Purpose:** Mod system and DLC support infrastructure
**Contents:**
- `core/` - Core mod system, registry, utilities, compatibility
- `dlc/` - DLC support, strings, world generation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vietnd69/dst-api-webdocs](https://github.com/vietnd69/dst-api-webdocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
