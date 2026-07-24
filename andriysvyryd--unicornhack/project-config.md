---
trigger: always_on
description: Guidance for generating code for UnicornHack. If unsure, don't guess: say you don't know or ask. Don't copy a pattern into a different context; evaluate code by implementation and usage, not names. Verify generated code is correct and compilable.
---

# UnicornHack - GitHub Copilot Instructions

Guidance for generating code for UnicornHack. If unsure, don't guess: say you don't know or ask. Don't copy a pattern into a different context; evaluate code by implementation and usage, not names. Verify generated code is correct and compilable.

## Project Overview

Traditional turn-based roguelike on .NET: ASP.NET Core Razor Pages web frontend, SignalR multiplayer, procedural level generation, Entity Component System (ECS) with message-driven systems.

Projects: `UnicornHack.Core` (game engine/ECS/logic), `UnicornHack.Web` (Razor Pages + SignalR), `UnicornHack.Editor` (content editing/serialization tools), `UnicornHack.Core.Tests`, `UnicornHack.Core.PerformanceTests`.

Key patterns:
- ECS: entities (players, monsters, items, levels) hold components (data); systems process entities by component combination; systems communicate via messages.
- Messaging: `IMessageQueue`/`SequentialMessageQueue<T>` for ordered processing; messages implement `IMessage`; systems implement `IMessageConsumer<TMessage, TState>`; results control flow.
- State: entities persist via `IRepository`; lifecycle via `GameManager`; turn-based tick timing.

## Game Mechanics

Roguelike dungeon crawler. Attributes: Might, Focus, Perception, Speed. Systems for skills/abilities, equipment, magic (energy-point abilities), and races/traits/feats customization.

Core components: `BeingComponent` (living creatures: HP/EP/attributes), `PlayerComponent` (skill/trait points), `PhysicalComponent` (size/weight/material), `ItemComponent` (type/equipment slots), `AbilityComponent`, `LevelComponent` (levels/terrain), `PositionComponent` (location/movement), `SensorComponent` (vision/sensing), `KnowledgeComponent` (world knowledge).

Key systems: `LivingSystem` (health/energy/attributes), `PlayerSystem` (turn processing/player actions), `AbilityActivationSystem` (casting), `ItemMovingSystem` (inventory/equipment), `TerrainSystem` (geometry/visibility), `KnowledgeSystem` (discovery), `TimeSystem` (turn scheduling).

### Per-Turn Change Set Architecture

When a player acts, the server processes all turns until the player can act again. Instead of one delta covering all accumulated changes, the server collects changes per turn (one entity's action + cascading effects) and sends them as a list of `TurnChangeSet` via the `ReceiveChangeSets` SignalR method. The client applies them sequentially with a configurable delay to animate what happened.

Server flow (`GameStateManager.Turn`):
1. Per turn: clear terrain dicts -> capture visible terrain snapshot -> `TimeSystem.AdvanceSingleTurn()` -> compute visible terrain changes.
2. Per registered `PlayerChangeListener`: if `HasObservableChanges` -> `BuildChangeSet()` -> append to that player's list (a forced emit also happens on the terminating `PlayerTurn` tick to keep the last state aligned).
3. `levelChangeBuilder.Clear()` -> repeat until `TurnResult.PlayerTurn` or `GameOver`.
4. Returns `Dictionary<int, List<TurnChangeSet>>` keyed by player entity id; the hub sends each player their list in one `ReceiveChangeSets` call.

Change detection - under [src/UnicornHack.Web/Hubs/ChangeTracking/](../src/UnicornHack.Web/Hubs/ChangeTracking):
- `LevelChangeBuilder`: shared per-game aggregator. Owns the level-scope builders (`ActorChangeBuilder`, `ItemChangeBuilder`, `ConnectionChangeBuilder`) and a list of `PlayerChangeListener`s. Registered once on the ECS groups; `RegisterPlayerListener` is idempotent.
- `PlayerChangeListener`: per-player. Owns `RaceChangeBuilder`, `AbilityChangeBuilder`, `LogChangeBuilder`. Implements `IEntityChangeListener` on `LevelActors` to capture the player entity's scalar property changes (HP/EP/XP/...) into an internal `_pendingChange: PlayerChange`. `BuildChangeSet` assembles the final `PlayerChange` from level, races, abilities, log, terrain, and scalar bits.
- `ChangeBuilder<TChange>` (abstract): shared state machine for the five entity-scope builders (Actor/Item/Connection/Race/Ability). Handles Added/Modified/Removed tracking and serialization emission. Subclasses override only: which groups to subscribe to, how to filter relevant entities, how to translate property changes onto the DTO, and how to produce full snapshots. See [ChangeBuilder.cs](../src/UnicornHack.Web/Hubs/ChangeTracking/ChangeBuilder.cs) for the state-transition table.
- `TerrainChangeBuilder`: static helper. Diffs the dictionaries `LevelComponent` maintains (`KnownTerrainChanges`, `TerrainChanges`, `WallNeighborsChanges`, `VisibleTerrainChanges`) into a full `LevelMap` or sparse `LevelMapChanges`.
- `LogChangeBuilder`: high-water-mark on log-entry id. Not a `ChangeBuilder` subclass - ordered append-only event streams don't need the state machine.

Invariants enforced by `ChangeBuilder<TChange>`:
- `IChangeWithState.LastState` is assigned once when an entry is created, never mutated after.
- Add-then-Remove within a turn cancels iff `LastState == EntityState.Added` (client never knew the entity).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndriySvyryd/UnicornHack](https://github.com/AndriySvyryd/UnicornHack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
