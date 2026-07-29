---
trigger: always_on
description: TinyEcs is a high-performance, reflection-free entity component system (ECS) framework for .NET. It targets zero-runtime-allocation workflows, supports NativeAOT/bflat, and ships with an optional Bevy-inspired scheduling layer that brings modern stage orchestration, observers, system-parameter injection, and component bundles to C# game and simulation projects.
---

# CLAUDE Project Brief

## Overview
TinyEcs is a high-performance, reflection-free entity component system (ECS) framework for .NET. It targets zero-runtime-allocation workflows, supports NativeAOT/bflat, and ships with an optional Bevy-inspired scheduling layer that brings modern stage orchestration, observers, system-parameter injection, and component bundles to C# game and simulation projects.

## Core Philosophy
- **Reflection-free**: All component registration and lookups avoid runtime reflection, enabling AOT and high determinism. **This is a strict requirement - no `GetType()`, `GetProperty()`, or other reflection APIs in hot paths.**
- **Zero-allocation**: Designed for minimal GC pressure in hot paths
- **Cache-friendly**: Archetype-based storage for optimal memory layout
- **Compile-time safety**: Strong typing with ref structs and source generation
- **Thread-safe**: Deferred command system for safe multi-threaded execution

## Repository Layout
- `src/TinyEcs/` — Core ECS runtime (world, entity views, archetype storage, queries, entity-tied observers)
- `src/TinyEcs.Bevy/` — Bevy-inspired extensions (App, stages, plugins, global observers, system parameters, bundles, resources, events, states)
- `samples/` — Example programs (TinyEcsGame with Raylib, MyBattleground)
- `tests/` — xUnit test suites (115+ tests covering all features)
- `benchmarks/` — Performance evaluation scenarios

## Core ECS (src/TinyEcs/)

### World & Entities
- `World` - Pure ECS storage. Manages entities, components, archetypes, and entity-tied observers. Does NOT own resources, events, or states (those live on `App`).
- `EntityView` - Lightweight handle to an entity with `.Set()`, `.Get()`, `.Has()`, `.Unset()`, `.Delete()`
- Entities are 64-bit IDs with recycling support

### Components
- Must be `struct` types (value types)
- Zero-sized tags supported (e.g., `struct Player {}`)
- Change detection built-in with tick tracking
- Access via `Ptr<T>` (ref struct) for zero-copy reads/writes

### Queries
- `Query<Data<T1, T2, ...>>` - Multi-component iteration
- `Filter<T1, T2, ...>` - Combine multiple filters
- Built-in filters:
  - `With<T>` / `Without<T>` - Component presence
  - `Changed<T>` / `Added<T>` - Change detection
  - `Optional<T>` - Nullable component access
  - `MarkChanged<T>` - Manually mark components as changed
- Queries use `foreach` pattern with ref access via `.Ref`

### Archetypes
- Entities grouped by component signature
- Cache-friendly columnar storage
- Automatic archetype transitions on component add/remove

## Bevy Layer (src/TinyEcs.Bevy/)

### App & Stages
```csharp
var app = new App(ThreadingMode.Auto); // or Single, Multi
app.AddPlugin(new MyPlugin());
app.RunStartup(); // Runs once
while (running) app.Update(); // Run all stages
```

**Default Stages** (in execution order):
- `Stage.Startup` - Runs once on first frame (always single-threaded)
- `Stage.First` - First regular update stage
- `Stage.PreUpdate` - Before main update
- `Stage.Update` - Main gameplay logic
- `Stage.PostUpdate` - After main update
- `Stage.Last` - Final stage (rendering, cleanup)

**Custom Stages**:
```csharp
var stage = Stage.Custom("MyStage");
app.AddStage(stage).After(Stage.Update).Before(Stage.PostUpdate);
```

### System Registration
```csharp
// Fluent API with system parameters
app.AddSystem((Query<Data<Position, Velocity>> query, Res<Time> time) =>
{
    foreach (var (pos, vel) in query)
        pos.Ref.Value += vel.Ref.Value * time.Value.Delta;
})
.InStage(Stage.Update)
.Label("movement")
.After("input")
.SingleThreaded() // Force single-threaded execution
.RunIf(_ => !app.GetResource<GameState>().Paused)
.Build();
```

### System Ordering
- **Declaration order preserved** - Systems run in the order they're added when no dependencies exist
- **Explicit ordering**:
  - `.After("label")` - Run after labeled system
  - `.Before("label")` - Run before labeled system
  - `.Chain()` - Run after the previously added system
- **Topological sort** - Automatically resolves dependency graph
- **Error handling** - Throws exception if label doesn't exist

### Threading Modes
- `ThreadingMode.Auto` - Use parallel execution if `ProcessorCount > 1`
- `ThreadingMode.Single` - Force all systems to run sequentially
- `ThreadingMode.Multi` - Enable parallel execution
- **Per-system override**: `.SingleThreaded()` or `.WithThreadingMode(mode)`
- **Batching** - Systems without conflicts run in parallel batches
- **Declaration order in batches** - Batches preserve system declaration order
- **Startup stage**: `Stage.Startup` always runs in single-threaded mode regardless of app threading mode. This ensures deterministic initialization and proper resource setup.

### System Parameters

**Query Parameters**:
```csharp
Query<Data<Position, Velocity>> query
Query<Data<Sprite>, Filter<Changed<Position>>> filtered
```

**Resource Access**:
```csharp
Res<TimeResource> time        // Immutable resource

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andreakarasho/TinyEcs](https://github.com/andreakarasho/TinyEcs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
