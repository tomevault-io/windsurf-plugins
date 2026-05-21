---
trigger: always_on
description: This is **Platter**, a Cities: Skylines 2 mod.
---

# Copilot Instructions — CS2-Platter

## Project Overview
This is **Platter**, a Cities: Skylines 2 mod.
Platter introduces a new way for players to manage their city's zoning and building by implementing a custom "parcel" entity. Parcels are flexible, player-defined areas that can be freely placed, allowing for more precise control over zoning and building placement compared to the vanilla zone-block system.
In the base game, the zoning system is based on "blocks" that are automatically created alongside roads, each containing a grid of cells. Players can zone these blocks, but they have limited control over the position of the blocks. Parcels decouple the zoning system from the road network, allowing players to create custom-shaped zones that can be placed anywhere, even overlapping existing blocks. This enables more creative and efficient city layouts.

The mod has two major layers:

| Layer | Stack | Location |
|-------|-------|----------|
| **Backend (C#)** | .NET Framework 4.8, Unity ECS (Entities / Burst / Jobs), Harmony patching | `Platter/` |
| **Frontend (TypeScript)** | React 18, SCSS Modules, CS2 UI Modding SDK (`cs2/*` externals), webpack | `Platter/UI/` |

### Key Architectural Concepts

- **ECS (Entity Component System)**: All game-state data lives in ECS components (`Platter/Components/`). Systems (`Platter/Systems/`) run jobs to process entities. Many systems use Burst-compiled `IJob` / `IJobParallelForDefer` structs.
- **Parcels**: The core domain object. A parcel is an entity with `Parcel`, `ParcelData`, `ParcelSubBlock`, and related components. Parcels interact with vanilla `Block` / `Cell` / `ZoneType` entities.
- **Prefabs**: `ParcelPrefab` and `ParcelPlaceholderPrefab` (in `Game.Prefabs` namespace) define the prefab templates that the game instantiates.
- **Harmony Patches**: Located in `Platter/Patches/`, used to intercept and modify vanilla game systems (tool systems, bulldoze, etc.).
- **UI ↔ C# Bindings**: Two-way communication uses `ValueBindingHelper<T>` / `TriggerBinding` on the C# side and `TwoWayBinding<T>` / `TriggerBuilder` on the TS side, with the mod id `"Platter"` as the namespace.
- **Localization**: English strings are defined in `Platter/L10n/EnUsConfig.cs`.

---

## FAQ

**Q: What game is this mod for?**
A: Cities: Skylines II (CS2).

**Q: Why .NET Framework 4.8?**
A: CS2 mods target Unity's Mono runtime which requires `net48`. This is set in the `.csproj` and enforced by the CS2 modding toolchain (`Mod.props`).

**Q: How is the UI built?**
A: The UI is a React/TypeScript module built with webpack (`Platter/UI/`). It is compiled into a JS bundle and deployed alongside the C# DLL. The CS2 UI SDK injects it into the game's Coherent GT browser.

**Q: How do C# and TypeScript communicate?**
A: Through `ValueBinding` / `TriggerBinding` pairs. C# side: `ExtendedUISystemBase.CreateBinding` / `CreateTrigger`. TS side: `TwoWayBinding` class in `utils/bidirectionalBinding.ts`. The binding key convention is `"BINDING:<KEY>"` / `"TRIGGER:<KEY>"`.

**Q: What are the build configurations?**
A: `Debug` (profiler + debug defines), `Release` (Burst enabled), and `I18N` (debug + locale export).

---

## Goals

1. **Maintain ECS discipline**: All mutable game state must live in ECS components. Systems must use jobs for data processing wherever possible.
2. **Burst compatibility**: Code inside `[BurstCompile]` jobs must not use managed types, allocations, or virtual calls. Guard Burst-compiled structs with `#if USE_BURST` / `[BurstCompile]`.
3. **Minimal Harmony patches**: Only patch vanilla methods when absolutely necessary; prefer ECS-based solutions.
4. **Type-safe UI bindings**: Every new binding must have a matching C# `ValueBindingHelper<T>` and TS `TwoWayBinding<T>` declaration with the same key.
5. **Backward compatibility**: Serializable components implement `ISerializable`; changes to serialized data must be migration-safe.

---

## Guidelines

### C# Coding Conventions

- **Namespace per folder** — e.g., `Platter.Systems`, `Platter.Components`, `Platter.Utils`, `Platter.Constants`, `Platter.Patches`.
  - Exception: Prefabs live in `Game.Prefabs` namespace to integrate with the game's prefab system.
- **File header** — Every `.cs` file starts with the standard copyright header:
  ```csharp
  // <copyright file="FileName.cs" company="Luca Rager">
  // Copyright (c) Luca Rager. All rights reserved.
  // Licensed under the MIT license. See LICENSE file in the project root for full license information.
  // </copyright>
  ```
- **Using statements** — Wrapped in `#region Using Statements` / `#endregion` blocks at the top of the namespace.
- **Naming**:
  - Fields: `m_PascalCase` prefix (e.g., `m_ZoneSearchSystem`, `m_Log`).
  - ECS component fields: `m_camelCase` (e.g., `m_LotSize`, `m_RoadEdge`) following vanilla CS2 convention.
  - Constants: `PascalCase` or `UPPER_CASE` (match surrounding file).
  - System classes: `P_` prefix (e.g., `P_NewCellCheckSystem`, `P_UISystem`).
  - Partial class jobs: separate file per job struct, named `<SystemName>.<JobName>.cs`.
- **Brace style** — Opening brace on the **same line** as declaration (Egyptian/K&R style):
  ```csharp
  public class Foo : Bar {
      // ...
  }
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucarager/CS2-Platter](https://github.com/lucarager/CS2-Platter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
