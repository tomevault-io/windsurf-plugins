---
trigger: always_on
description: AwavePuzz is a multiplayer Roblox zombie survival game featuring wave-based combat, cure-crafting puzzles, and alliance systems. This is a Lua-based game designed for the Roblox platform with up to 8 players per server.
---

# GitHub Copilot Instructions for AwavePuzz

## Project Overview

AwavePuzz is a multiplayer Roblox zombie survival game featuring wave-based combat, cure-crafting puzzles, and alliance systems. This is a Lua-based game designed for the Roblox platform with up to 8 players per server.

## Technology Stack

- **Language**: Lua (Roblox Luau)
- **Platform**: Roblox
- **Architecture**: Modular server-client system with server-authoritative design
- **Design Pattern**: Object-oriented with manager classes

## Core Game Systems

1. **Multiplayer Support**: Up to 8 players, real-time cooperative gameplay
2. **Wave-Based Combat**: Progressive difficulty scaling, zombie AI with pathfinding
3. **Cure-Crafting Puzzle**: 5 unique components, each requiring 5 pieces
4. **Alliance System**: Team-up mechanics with betrayal options
5. **Weapon & Upgrade System**: Server-authoritative raycast weapons, shop system
6. **Inventory & Resource Loop**: Server-tracked inventory, resource spawns
7. **Dynamic Map Support**: MapManager with zombie/resource spawn points

## Project Structure

```
AwavePuzz/
├── ServerScriptService/  # Server-side game logic
│   ├── AI/               # Zombie AI and controllers
│   └── *.lua             # Server services and managers
├── ReplicatedStorage/    # Roblox shared resources
│   └── Shared/           # Configuration and utility modules
├── StarterPlayer/        # Player initialization
│   └── StarterPlayerScripts/  # Client-side controllers
│       ├── Modules/      # Client modules
│       └── FPS/          # FPS camera system
├── StarterGui/           # UI LocalScripts
├── ServerStorage/        # Server-only assets
│   ├── Maps/             # Map models
│   ├── Models/           # Weapon/object models
│   └── ZombieModels/     # Zombie models
├── Archive/Legacy/Code/  # Archived legacy code (old src/ structure)
└── docs/                 # Project documentation
```

> **Note**: The repository structure matches Roblox Studio exactly. The old `src/` structure has been archived in `Archive/Legacy/Code/`.

## Coding Standards

### General Principles

1. **Server Authority**: All game logic MUST be server-authoritative for multiplayer safety
   - Never trust client for damage, currency, cure progress, or alliances
   - Validate all client inputs on the server
   - Use RemoteEvents/RemoteFunctions for client-server communication

2. **Modular Design**: Use ModuleScripts for configuration and reusable logic
   - Keep scripts self-contained and independent
   - Create folders/instances at runtime if they don't exist
   - Use clear manager classes for each system

3. **Modern Lua Practices**:
   - Use `task.wait()` instead of `wait()`
   - Use attributes (`Instance:SetAttribute`) for tagging and simple flags
   - Use `task.spawn()` for concurrent operations
   - Prefer descriptive variable names

4. **Documentation**:
   - Add clear comments explaining what each script does
   - Document how systems interact with each other
   - Reference related systems in comments

### Code Style

```lua
-- Use PascalCase for classes/modules
local PlayerManager = {}

-- Use camelCase for functions and variables
function PlayerManager:getPlayerHealth(player)
    local health = player.Character.Humanoid.Health
    return health
end

-- Use UPPER_CASE for constants
local MAX_PLAYERS = 8
local BASE_HEALTH = 1000

-- Always validate inputs
function PlayerManager:dealDamage(player, amount)
    assert(typeof(player) == "Instance", "Invalid player")
    assert(typeof(amount) == "number" and amount > 0, "Invalid damage amount")
    
    -- Implementation...
end
```

### Remote Event Naming

Use descriptive names that indicate direction and purpose:
- `DealDamage` (client → server)
- `WaveAnnounce` (server → client)
- `RequestAlliance` (client → server)
- `UpdateCureProgress` (server → client)

## Key Configuration Files

- `src/shared/GameConfig.lua` - Game tuning parameters
- `src/shared/ZombieTypes.lua` - Zombie definitions and stats
- `src/shared/WaveConfig.lua` - Wave progression settings

## Important Documentation

Before making changes, consult these files:
- `INSTALLATION.md` - Complete setup guide for Roblox Studio
- `GAME_DESIGN.md` - Game design document and mechanics
- `API_DOCUMENTATION.md` - API reference and system interactions
- `IMPLEMENTATION_SUMMARY.md` - Implementation details and progress

## Development Workflow

### Phase-Based Development

The project follows a phased development approach:
1. **Phase 1**: Core Loop (GameManager, Spawner, ZombieBrain, Waves)
2. **Phase 2**: Weapons & Damage (Raycast weapons, kill rewards)
3. **Phase 3**: Cure System (CureService, puzzle interactions)
4. **Phase 4**: Alliances (AllianceService, UI interactions)
5. **Phase 5**: Polish & Balancing (Tuning, sounds, UI improvements)

### Testing in Roblox

- Changes must be tested in Roblox Studio
- Test in multiplayer mode when possible
- Verify server-client synchronization
- Check for exploits and validation gaps

## Common Patterns

### Creating a Manager Script

```lua
local ManagerName = {}
ManagerName.__index = ManagerName

-- Dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Carnage-Joker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
