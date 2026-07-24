---
trigger: always_on
description: Zinc is a small C# 2D game framework built on the sokol family of headers (windowing, GFX, immediate-mode 2D, ImGui), Box2D 3.x (physics + collision math), and the [Arch ECS](https://github.com/genaray/Arch). Source generators stitch authored `Entity` classes to ECS components so user code reads as OO while storage stays cache-friendly.
---

# Zinc

Zinc is a small C# 2D game framework built on the sokol family of headers (windowing, GFX, immediate-mode 2D, ImGui), Box2D 3.x (physics + collision math), and the [Arch ECS](https://github.com/genaray/Arch). Source generators stitch authored `Entity` classes to ECS components so user code reads as OO while storage stays cache-friendly.

This file orients an AI assistant or a new contributor to the *internals* of the framework. For a tutorial, run the demos in the parent `Zinc.Demos` project.

## Top-level layout

| Path | What lives there |
|---|---|
| `Zinc.csproj` | The framework's csproj. Targets `net10.0`, references `Arch`, `Arch.AOT.SourceGenerator`, `Depot.SourceGenerator`, `Zinc.ECSGenerator`, `Zinc.Magic`. |
| `src/Core.cs` | The `Engine` static class. Owns the boot/init/frame/cleanup callbacks, the ECS world, the physics world, the scene registry, the default systems, and the immediate-mode draw helpers (`DrawTexturedRect`, `DrawShape`, `DrawText`, `DrawParticles`). Everything starts here. |
| `src/Core/` | Engine-internal modules: physics wrapper, collision API, resources, ECS components, systems, entity types, ImGui C# wrapper, utility code. |
| `src/Quick.cs` | One-call convenience helpers (centering, palettes, random). |
| `src/NativeUtils.cs` | `NativeArray<T>` (unmanaged buffer with managed lifetime) + `AsSgRange` for sokol uploads. |
| `src/Bindings/` | **Submodule** ([zinc-framework/Zinc.Bindings](https://github.com/zinc-framework/Zinc.Bindings)). All P/Invoke bindings: sokol (`sokol/Sokol.*.cs`), Box2D (`box2d/Box2D.cs`), STB (`stb/`). Do not hand-edit — regenerate upstream. |
| `libs/` | **Submodule** ([zinc-framework/Zinc.Libs](https://github.com/zinc-framework/Zinc.Libs)). Prebuilt native dylibs/dlls. Loaded at runtime by `NativeLibResolver`. |
| `data/`, `logos/` | Default fonts / palettes / images shipped with the framework. Copied to output via the csproj. |

## The single source of truth: `Engine`

`Engine` is a `static partial class` and is the only top-level thing user code talks to. Read `src/Core.cs` first; the rest of the framework is supporting cast.

### Boot path

```
Program.cs → Engine.Run(RunOptions) → Engine.Boot(opts) →
  NativeLibResolver.kick() (registers DllImport resolver)
  fills sapp_desc + delegates → App.run(&desc)   // sokol drives the main loop
```

`App.run` is the sokol_app entry point. It does NOT return until the window closes; the sokol library calls back into our delegates for `Initialize` / `Event` / `Frame` / `Cleanup`. All four are `[UnmanagedCallersOnly(CallConvs=[Cdecl])]` static methods on `Engine`. Read `Core.cs` for the exact order of operations in each.

Two facts about the frame loop that aren't obvious from the code:
- `DeltaTime` is set from `App.frame_duration()` (EMA-smoothed) for animation/UI; physics is stepped separately with `App.frame_duration_unfiltered()` so spikes aren't masked from the accumulator.
- Input events are *not* processed inside `Event` directly. They're recorded as one-frame `FrameEvent` ECS entities and consumed during the update phase by `InputSystem`, after ImGui has had a chance to claim them.

## Scenes and entities

### Scenes (`src/Core/Entites/Scene.cs`)

Lifecycle: `Mount(depth)` → `Load(callback)` → `Start()`. `Unmount(callback)` schedules removal at end of frame. Multiple scenes can be mounted simultaneously; `Engine.TargetScene` is the one that receives input focus. Each scene owns a list of entity IDs in `Engine.SceneEntityMap`.

### Entities (`src/Core/Entites/`)

Folder name is misspelled (`Entites`), intentionally not fixed. Contains:

- `Entity` — base type. Wraps an `Arch.Core.Entity` (`ECSEntity` property).
- `Anchor` — transform/parenting helper. Most renderables are `Anchor`-derived.
- `SceneEntity` / `SceneObject` — scene-aware entities.
- Concrete renderables: `Shape`, `Sprite`, `AnimatedSprite`, `Text`, `Pointer`, `ParticleEmitter`, `Grid`, `Coroutine`, `Temporary`.

Each concrete entity class is decorated with `[Component<T>("Name")]` attributes (from `Zinc.ECSGenerator` / `Depot.SourceGenerator`). The source generator emits property accessors so user code can do `myShape.Collider_Active = true` instead of touching the underlying ECS component. Example, see `Pointer.cs`:

```csharp
[Component<Position>]
[Component<Collider>("Collider")]
[Component<UpdateListener>]
public partial class Pointer : Entity { ... }
```

The `"Collider"` string is the *prefix* generated property names get (`Collider_Width`, `Collider_Active`, ...). Omitting the string uses the bare component name.

### Components (`src/Core/Components/`)

`record struct`s with `[Arch.AOT.SourceGenerator.Component]` so Arch can pack them into archetypes. Notable ones:

- `Position(X, Y)` — implicitly converts to `System.Numerics.Vector2`.
- `Collider` — width/height/pivot/active + lifecycle and mouse callbacks + `ActiveCollisions` set + an `IsPoint` flag (when true, width/height/pivot are ignored and the entity's `Position` *is* the collider).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zinc-framework/Zinc](https://github.com/zinc-framework/Zinc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
