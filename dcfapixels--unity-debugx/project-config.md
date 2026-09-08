---
trigger: always_on
description: Usage rules for agents that use the product-ready `Unity-DebugX` package in a Unity project.
---

# AGENTS.md

Usage rules for agents that use the product-ready `Unity-DebugX` package in a Unity project.

## Purpose

`Unity-DebugX` draws non-interactive debug gizmos from regular gameplay or editor code. Use it when you need visual diagnostics similar to `Debug.DrawLine`, but with richer shapes, text, dots, casts, and primitives.

The core usage pattern is:

```csharp
DebugX.Draw(duration, color).Line(start, end);
DebugX.Draw(color).WireSphere(position, radius);
```

Draw calls enqueue gizmo data into DebugX buffers. DebugX renders that data later during its render update. You can call drawing methods from ordinary code such as `Update`, systems, services, utility methods, and debugging helpers.

## Public API Only

Use only public API exposed by the package:

- `DCFApixels.DebugX`
- `DCFApixels.DebugX.DrawHandler`
- `DCFApixels.DebugXEvents`
- `DCFApixels.DebugXLine`
- `DCFApixels.DebugXTextSettings`
- `DCFApixels.DebugXTextSettingsExtensions`
- public extension methods available on `DebugX.DrawHandler`
- public static data marker types from `DCFApixels.DebugXCore`, such as `SphereMesh`, `CubeMesh`, `LitMat`, `UnlitMat`, `WireMat`, when needed for public generic mesh calls

Do not use anything from `Runtime/Internal`. That folder contains implementation details of the package and is not a supported API surface.

Do not rely on private, internal, nested, unsafe, renderer, buffer, pinned-array, type-code, command-buffer, or lifecycle classes. They can change without notice.

## Files Agents Should Ignore

- Do not inspect or depend on `Runtime/Internal/` for normal usage.
- Do not use `Samples/` as required project knowledge. Samples are for users. You may copy simple call patterns from them when you need examples, but do not treat sample scripts, scene setup, or sample utilities as API.
- Do not modify package source, `.asmdef`, `.meta`, `Runtime/Resources`, shaders, meshes, or materials unless the user explicitly asks to develop the package itself.

## Basic Drawing

Import the package namespace:

```csharp
using DCFApixels;
```

Use `DebugX.Draw(...)` to choose color and duration:

```csharp
DebugX.Draw(Color.red).Line(start, end);
DebugX.Draw(1f, Color.yellow).Cube(center, rotation, size);
DebugX.Draw(Color.cyan).WireSphere(center, radius);
```

Common duration forms:

- `DebugX.Draw(color)` - draw with default duration for the current context.
- `DebugX.Draw(duration)` - draw for a duration using the default color.
- `DebugX.Draw(duration, color)` - draw for the specified duration and color.
- `DebugX.Draw()` - default duration and default color.

Methods are chainable when useful:

```csharp
DebugX.Draw(0.5f, Color.green)
    .Line(a, b)
    .WireSphere(b, 0.25f)
    .Text(b, "hit");
```

## Gizmo Call Families

The examples below cover the public gizmo families exposed by the package, not only the calls used in `Samples/`. Many methods have overloads for `Ray`, `Transform`, `Quaternion`, `Vector2`/`Vector3` collections, `float` vs vector sizes, and generic material variants.

Lines, line lists, and line strips:

```csharp
DebugX.Draw(Color.white).Line(start, end);
DebugX.Draw(Color.white).Lines(points);
DebugX.Draw(Color.white).LineStrip(points);
DebugX.Draw(Color.green).LineArrow(start, end);
DebugX.Draw(Color.green).LineFade(start, end);
DebugX.Draw(Color.yellow).Line(start, end, DebugXLine.Arrow, DebugXLine.Fade);
DebugX.Draw(Color.cyan).WidthLine(start, end, 0.1f);
DebugX.Draw(Color.cyan).WidthOutLine(start, end, 0.1f);
DebugX.Draw(Color.magenta).ZigzagLine(start, end, 0.3f);
DebugX.Draw(Color.white).Distance(start, end);
```

Rays and ray-shaped helpers:

```csharp
DebugX.Draw(Color.white).Ray(origin, direction);
DebugX.Draw(Color.cyan).Ray(origin, direction, DebugXLine.Arrow);
DebugX.Draw(Color.green).RayArrow(origin, direction);
DebugX.Draw(Color.green).RayFade(origin, direction);
DebugX.Draw(Color.yellow).RayWireBox(origin, direction, rotation, size);
DebugX.Draw(Color.yellow).RayWireSphere(origin, direction, radius);
DebugX.Draw(Color.yellow).RayWireCapsule(origin, direction, rotation, radius, height);
```

Line-to-shape helpers:

```csharp
DebugX.Draw(Color.yellow).LineWireBox(start, end, rotation, size);
DebugX.Draw(Color.yellow).LineWireSphere(start, end, radius);
DebugX.Draw(Color.yellow).LineWireCapsule(start, end, rotation, radius, height);
```

3D primitives:

```csharp
DebugX.Draw(Color.yellow).Cube(position, rotation, size);
DebugX.Draw(Color.yellow).WireCube(position, rotation, size);
DebugX.Draw(Color.yellow).CubePoints(position, rotation, size);
DebugX.Draw(Color.yellow).CubeGrid(position, rotation, size, Vector3Int.one * 3);
DebugX.Draw(Color.cyan).Sphere(position, radius);
DebugX.Draw(Color.cyan).WireSphere(position, radius);
DebugX.Draw(Color.blue).Cylinder(position, rotation, radius, height);
DebugX.Draw(Color.blue).WireCylinder(position, rotation, radius, height);
DebugX.Draw(Color.green).Capsule(position, rotation, radius, height);
DebugX.Draw(Color.green).Capsule(point1, point2, radius);
DebugX.Draw(Color.green).WireCapsule(position, rotation, radius, height);
DebugX.Draw(Color.red).Cone(position, rotation, radius, height);
DebugX.Draw(Color.red).WireCone(position, rotation, radius, height);
```

Planar and 2D-style primitives:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DCFApixels/Unity-DebugX](https://github.com/DCFApixels/Unity-DebugX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
