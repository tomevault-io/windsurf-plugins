---
trigger: always_on
description: This file provides guidance to AI when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to AI when working with code in this repository.

## Working as AI

- Use caveman skill if available.
- Load appropriate drawnui related skill.
- Explain fixes and solutions BEFORE implementing — wait for approval.
- No summaries or explanations AFTER approved changes are made. Make the changes and stay silent.

### AI skills (canonical source: `docs/skills/` in this repo)

Skills live in-repo at `docs/skills/<name>/SKILL.md` and are published to `https://drawnui.net/skills/` by the docs workflow. If not installed in your agent, read them directly from `docs/skills/`. Load by task:

| Task | Skill |
|---|---|
| Any DrawnUI controls/layout/caching/gesture work | `drawnui` (always) |
| Writing C# code-behind / fluent composition / XAML→C# porting | + `drawnui-fluent` (mandatory for fluent C#) |
| Games (DrawnGame, sprites, pooling, WASM startup) | + `drawnui-game` |
| Blazor head apps, font subsetting, GitHub Pages publishing | + `drawnui-blazor` |
| Pure-WASM DrawnUi.Web apps, WASM runtime bug hunting | + `drawnui-web-app` |
| OpenTK desktop apps, GL overlays, window chrome | + `drawnui-opentk` |
| Headless testing/repros (no device/GPU) | + `drawnui-net-harness` |
| SkiaSharp SKMesh / SkSL mesh drawing | + `skmech` |

---

## Project Overview

DrawnUI is a cross-platform rich UI rendering engine that draws controls with SkiaSharp instead of native widgets. It targets:

| Target | Implementation | Platforms |
|---|---|---|
| .NET MAUI | `src/Maui/` | iOS, Android, Windows, MacCatalyst |
| Blazor | `src/Blazor/` | Browser (WASM), Server, Hybrid |
| OpenTK | `src/OpenTk/` | Windows, Linux |
| Native Windows | `src/Native/` | WinUI |

**Technology stack:**
- .NET 9 and .NET 10 (both supported; MAUI targets multi-TFM)
- SkiaSharp v4 (4.148+) — SKPaint has no FilterQuality/text state anymore (SKSamplingOptions / SKFont; see `drawnui` skill)
- Hardware-accelerated rendering via Skia GPU canvas

---

## Project Structure

```
src/
  Shared/           # Cross-platform Skia rendering core (shared by all targets)
  SharedNet/        # Shared .NET (non-MAUI) code — used by OpenTK and Net
  SharedGame/       # Shared game loop and input
  Maui/
    DrawnUi/        # Main MAUI library
    Addons/         # Camera, Game, MapsUi, Rive, MauiGraphics addons
    Samples/        # Sandbox, Tutorials, GameTemplate, Player, FastRepro
  Net/
    DrawnUi/        # .NET desktop base (used by OpenTK)
    Samples/        # SkiaEditorHarness
  OpenTk/
    DrawnUi/        # OpenTK integration: DrawnUiWindow, CanvasHost, GpuDrawable
    Addons/         # DrawnUi.OpenTk.Game
    Samples/        # OpenTkPong, OpenTkGpuHost, OpenTkOverlay
  Blazor/
    DrawnUi/        # Blazor component implementation
    DrawnUi.Server/ # Server-side host
    DrawnUi.Wasm/   # WASM-specific
    Addons/         # DrawnUi.Blazor.Game
    Samples/        # BlazorSandbox, BlazorSandboxHybrid, BlazorSandboxServer
  Native/
    DrawnUi.Native.Windows/  # WinUI native
  Tests/            # UnitTests, SomeBenchmarks
dev/                # Build/pack scripts and dev solution files
nugets/             # NuGet pack scripts
docs/               # Documentation (articles, api)
```

**Key architectural components:**
- `SkiaControl` — base class for all drawn controls
- `Canvas` — hosts drawn controls; wraps the Skia surface
- `SkiaShell` — navigation system for drawn apps
- Caching system: Operations, Image, ImageDoubleBuffered, GPU, ImageComposite
- Gesture handling system for touch and pointer input
- Layout system: Column, Row, Grid, Wrap, Absolute

---

**SkiaSharp version:**
- All targets use SkiaSharp v4
- Lib supports .NET 10 and .NET 9 targets

---

**AI assistance**
- when creating new apps prefer .NET 10
- when adding new public props/method always add XML comments
- when creating new controls create a ,NET harness and iterate there untill getting maximum possible result
- when working on shared specifics always test your outcome with .NET harness, never ask user to test anything in case you can test in harness
- when meeting a "Inline children" fluent code-behind flow (see below) always propose to fix
- when you miss a fluent extension ask for approval to create one to use, instead of breaking fluent flow

---

## Build Commands

**Main solution (all targets):**
```bash
dotnet build src/DrawnUi.sln
```

**MAUI library only:**
```bash
dotnet build src/Maui/DrawnUi/DrawnUi.Maui.csproj --configuration Debug
```

**MAUI Camera addon:**
```bash
dotnet build src/Maui/Addons/DrawnUi.Maui.Camera/DrawnUi.Maui.Camera.csproj --configuration Debug
```

**OpenTK library:**
```bash
dotnet build src/OpenTk/DrawnUi/DrawnUi.OpenTk.csproj --configuration Debug
```

**OpenTK sample (OpenTkOverlay):**
```bash
dotnet build src/OpenTk/Samples/OpenTkOverlay/OpenTkOverlay.csproj --configuration Debug
dotnet run --project src/OpenTk/Samples/OpenTkOverlay/OpenTkOverlay.csproj
```

**NuGet packages:**
```bash
cd nugets
./makenugets.bat   # Windows
```

**Tests:**
```bash
dotnet test src/Tests/UnitTests/UnitTests.csproj
```

**Clean build artifacts:**
```powershell
# From src/ directory
./DeleteBinObj.ps1
```

**Docs (drawnui.net):**
```powershell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrawnUi/DrawnUi.Net](https://github.com/DrawnUi/DrawnUi.Net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
