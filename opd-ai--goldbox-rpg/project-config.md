---
trigger: always_on
description: GoldBox RPG Engine is a modern Go-based framework for creating turn-based RPG games inspired by the classic SSI Gold Box series. This engine provides comprehensive character management, combat systems, and world interactions through a JSON-RPC API with WebSocket support for real-time communication. The project targets game developers building web-based RPG experiences with classical tabletop RPG mechanics including D&D-inspired attribute systems, turn-based combat, spell casting, and character p
---

# Project Overview

GoldBox RPG Engine is a modern Go-based framework for creating turn-based RPG games inspired by the classic SSI Gold Box series. This engine provides comprehensive character management, combat systems, and world interactions through a JSON-RPC API with WebSocket support for real-time communication. The project targets game developers building web-based RPG experiences with classical tabletop RPG mechanics including D&D-inspired attribute systems, turn-based combat, spell casting, and character progression focused on tactical gameplay with grid-based movement and positioning.

The engine features a complete character system with six core attributes (Strength, Dexterity, Constitution, Intelligence, Wisdom, Charisma), multiple character classes (Fighter, Mage, Cleric, Thief, Ranger, Paladin), and an advanced effect system for combat conditions (Stun, Root, Burning, Bleeding, Poison) and status modifications. The architecture emphasizes thread-safe concurrent operations, event-driven gameplay mechanics, and spatial indexing for efficient world queries through an R-tree-like structure. The project includes comprehensive procedural content generation for terrain, items, quests, and NPCs, along with robust system resilience patterns (circuit breakers, retry mechanisms, input validation).

The frontend is an Ebitengine/WASM client compiled from Go. Browser-based visual editors for map creation (`/editor.html`) and quest building (`/quest-builder.html`) enable content authoring. The project includes 10 embedded adventure packs with 100 maps and 37 quests providing 30+ hours of gameplay content.

## Technical Stack

- **Primary Language**: Go 1.25.6 with toolchain 1.25.8
- **Web Framework**: Native Go HTTP server with JSON-RPC 2.0 protocol
- **Real-time Communication**: Coder WebSocket v1.8.14 (nhooyr.io/websocket fork) for production; gorilla/websocket v1.5.3 for E2E test client only
- **Data Format**: YAML v3.0.1 for game data configuration (spells, items, PCG templates)
- **Logging**: Sirupsen Logrus v1.9.4 for structured logging with caller context
- **Utilities**: Google UUID v1.6.0 for entity identification, golang.org/x/exp for extended functionality
- **Testing**: Go built-in testing framework with Testify v1.11.1 for assertions, test coverage analysis scripts, chromedp v0.14.2 for browser playtests
- **Build System**: Makefile with gofumpt formatting, Docker support with health checks, asset generation pipeline
- **Frontend**: Ebitengine v2.9.9/WASM (Go compiled to WebAssembly), launched via splash-screen HTML page
- **Development Tools**: gofumpt for formatting, godocdown for documentation
- **Monitoring**: Prometheus client v1.23.2 for metrics collection, health check endpoints (`/health`, `/ready`, `/live`, `/metrics`)
- **Rate Limiting**: golang.org/x/time v0.15.0 for API throttling
- **Markov Chains**: mb-14/gomarkov for procedural text generation

## Code Assistance Guidelines

1. **Thread Safety First**: All Character and game state modifications must use proper mutex locking (`mu.Lock()` for writes, `mu.RLock()` for reads). Follow the established pattern in `pkg/game/character.go` where concurrent access is protected with `sync.RWMutex`. Example: Character struct uses `mu sync.RWMutex yaml:"-"` and all field modifications require proper locking.

2. **YAML-First Configuration**: Game data (spells, items, character classes) should be defined in YAML files under `/data/` directory. Use struct tags like `yaml:"spell_id"` for proper serialization. Reference `data/spells/cantrips.yaml` for structure examples with fields like `spell_level: 0`, `spell_school: 5`, `damage_type: ""`.

3. **Event-Driven Architecture**: Implement game actions through the event system in `pkg/game/events.go`. Create GameEvent structs with EventType enums and emit events using the EventSystem pattern. Events must include Type, SourceID, TargetID, Data map, and Timestamp for proper game state synchronization.

4. **JSON-RPC Method Pattern**: New server endpoints must follow JSON-RPC 2.0 specification in `pkg/server/handlers.go`. Pattern: validate session with `getSessionForMove()`, process game logic, emit events, return structured response. See `handleMove` implementation with parseMoveRequest, validateCombatConstraints, executePlayerMovement sequence.

5. **Spatial Awareness**: Use the spatial indexing system (`pkg/game/spatial_index.go`) for efficient world queries. Implement position-based operations through the R-tree-like SpatialIndex structure with Rectangle bounds and SpatialNode children rather than brute-force iteration over game objects.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opd-ai/goldbox-rpg](https://github.com/opd-ai/goldbox-rpg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
