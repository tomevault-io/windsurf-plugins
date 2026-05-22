---
trigger: always_on
description: VPet is a delightful IntelliJ IDEA plugin that brings animated pixel art companions to the
---

# VPet: Pixel Art Companions for IntelliJ

## What This Project Does

VPet is a delightful IntelliJ IDEA plugin that brings animated pixel art companions to the
IDE's status bar. These interactive pets respond to development activities (builds, tests,
executions) and add personality to the coding environment through sprite-based animations.

## How It Works

```
Developer Activity → Event Listeners → Animation State Machine → Sprite Renderer → Status Bar Widget
        |                   |                    |                      |              |
        | Build/Run         | Captures events    | Selects animation    | Renders      | Displays
        | events            | from IDE           | sequences            | frames       | animated pet
        |------------------>|                    |                      |              |
                            |                    |                      |              |
                            |------------------->| Based on success/    |              |
                                                 | failure/progress     |              |
                                                 |--------------------->|              |
                                                                        |------------->|
```

**Architecture Flow:**

1. **Event Capture**: BuildEventListener monitors IDE build/execution events
2. **State Management**: AnimationEventService broadcasts animation state changes
3. **Animation Selection**: PetAnimated selects appropriate animation sequences based on
   events
4. **Sprite Rendering**: DefaultIconRenderer processes sprite sheet frames
5. **UI Display**: AnimatedStatusBarWidget displays frames in the status bar using
   Flow-based reactive updates

## Architecture

### Core Components

**Status Bar Integration**

- `AnimatedStatusBarWidget`: Main widget implementation using Flow-based icon streaming
- `AnimatedStatusBarWidgetFactory`: Factory for creating widget instances
- Integrates with IntelliJ's StatusBar widget system

**Graphics System**

- `PetAnimated`: Main animation controller with state machine for pet behaviors;
  implements
  `Animated` (status bar), `Character` (game physics), and `Game` (game lifecycle hooks)
- `DefaultIconRenderer`: Renders sprite sheet frames into Swing Icons with animation queue
  management
- `Animation`: Data class representing animation sequences with looping and chaining
- `SpriteSheet`: Represents a collection of sprite frames from the atlas

**Configuration & Assets**

- `AsepriteJsonAtlasLoader`: Parses Aseprite JSON atlas files for sprite metadata
- `SpriteSheetAtlas`: Data structure for sprite sheet frame definitions
- Assets stored in `/META-INF/spritesheets/` (PNG images + JSON atlases)

**Event System**

- `BuildEventListener`: Captures ProjectTaskListener and ExecutionListener events
- `AnimationEventService`: Service broadcasting animation events to widgets
- `AnimationEventListener`: Interface for animation state change listeners
- `CoinCollectedListener`: Message bus topic for broadcasting coin collection events from
  game mode to status bar widget

**Game System**

- `Game`: Interface defining game lifecycle hooks (`onGameStart()`, `onGameStop()`) for
  participants in game mode
- `GameEngine`: Coordinator owning the game loop (Timer at 16ms), input gathering, world
  tick (`WorldUpdate.tick()`), and renderer updates — replaces inline logic from
  `GameController`
- `GameController`: Thin plugin.xml adapter; creates and delegates to `GameEngine` on
  `enterGameMode()` / `exitGameMode()`

**ECS System** (organized in game/ecs/)

- `EntityRegistry`: Component-based entity storage with entity lifecycle management;
  supports create/destroy entities, add/get/has components by type, query entities by
  component signature (`allWith(vararg types)`), and deferred removal via mark/flush
  pattern
- `SpatialGrid`: Hash-based spatial partitioning (4-tile cells) for collision detection;
  rebuilds from registry each frame, queries entities by AABB overlap
- `World.registry: EntityRegistry` — holds all entities/components; `World.player:
  EntityID` — player entity ID; `World.score: Int` — accumulated collectible score

**ECS Systems** (organized in game/ecs/systems/)

- `CollisionSystem`: Detects collectible-player collisions using spatial grid; filters
  candidates by Collectible component and AABB overlap test
- `AnimationSystem`: Updates AnimationComponent for all entities each frame; advances
  elapsed time and currentFrame based on AnimationResource frame duration

**ECS Components** (organized in game/ecs/components/)

- Player components: `Transform`, `Velocity`, `SpriteState`, `PhysicsState`, `PhaseState`
- Entity components: `Collectible`, `AABB`, `AnimationComponent`
- `AnimationComponent(resourceId, currentFrame, elapsed)`: References shared
  AnimationResource by ID string, not by holding full Animation instance

**Shared Resource System** (organized in game/resources/)

- `AnimationResource(id, animation, frames)`: Data class holding pre-extracted sprite
  frames and animation metadata; shared across entities
- `AnimationCache`: Singleton object managing shared AnimationResource instances;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stillya/vpet](https://github.com/stillya/vpet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
