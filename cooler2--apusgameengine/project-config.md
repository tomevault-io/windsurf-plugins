---
trigger: always_on
description: This file provides guidance to Claude Code when working on the Apus Game Engine refactoring.
---

# CLAUDE.md

This file provides guidance to Claude Code when working on the Apus Game Engine refactoring.

## Overview

**Apus Game Engine** - cross-platform 2D/3D game engine in Delphi/Pascal by Ivan Polyacov. Used in Spectromancer, Astral Heroes, Astral Towers.

Structure: `Base/` (platform-independent utilities) + root (engine modules).

**`Game/` is NOT part of the engine project** — it is just an EXAMPLE of game code built on the engine (separate product). Do not migrate, refactor, or count it as a consumer during engine refactoring; it is not in the engine CI build and adapts downstream on its own.

## Building

- Projects: `.dproj` (Delphi), `.lpi` (Lazarus/FPC)
- Defines: `DELPHI;OPENGL;LODEPNG;FREETYPE`
- Output: `bin\` (Win32), `bin64\` (Win64)
- Entry point: `TGameApplication.Create` → `Prepare()` → `Run()`

## Code Style

- 2 spaces indent, no tabs, use 2 spaces indent for functions declared in the interface section
- No spaces around operators: `a:=b+c`
- No space between colon and type name: `var x:integer`, `function Foo(a:string):integer`
- `begin` on same line after `then/do/else`, new line for procedures
- Classes: `TName`, Interfaces: `IName`, vars: `camelCase`
- Comments in English, translate Russian when modifying
- Method directive order matters: `overload; static;` not `static; overload;` — wrong order is a compile error
- Preserve UTF-8 BOM

## Architecture

- Main target CPU is x64, but should also support x86 and ARM. `ASM` blocks must be inside conditional compilation directives and accompanied by a pure Pascal implementation.
- Code should be compatible with both Delphi 12+ and FPC 3.2+ compilers.
- We use GitHub actions to run tests on Windows and Linux.

### Coordinate Conventions

- World space: **Z-up**, right-handed.
- Camera/view space: **forward = Z** (matches the depth buffer), X-right, Y-down.
- This is not enforced on user content - users may pick any "up" axis for their own
  scenes/models. But engine demos, samples and generator defaults must use Z-up.

### String Types

- **Primary**: `String8` (UTF-8) — main string type for all text
- **Alternative**: `String32` (UCS-4) — not used yet, but plan to support in future
- **Compatibility**: built-in `string` — use when String8 doesn't fit (e.g. RTL interop)
- Focus on String8 for new code

**Function overloads for string parameters:**
```pascal
// String8 is primary - always present (unconditional)
function Foo(const st:String8):...;
// UnicodeString for compatibility - only in Unicode mode
{$IFDEF UNICODE}
function Foo(const st:UnicodeString):...;
{$ENDIF}
```
This replaces the old `ADDANSI` pattern. Use `{$IFDEF UNICODE}` directly.

### Base Library (`Base/Apus.*.pas`) - 47 modules

**Dependency Hierarchy:**

```
Level 0 (no Apus dependencies):
  Types, EventMan, Colors, CPU, Crypto

Level 1:
  Classes, Common, Geom2D, FastGFX

Level 2:
  Structs, Geom3D, Images, Socket, CrossPlatform

Level 3:
  GfxFormats, Regions, AnimatedValues, TCP, HttpRequests

Level 4:
  UnicodeFont, TextUtils, Logging, Database, Translation, HtmlTree

Level 5:
  FreeTypeFont, GeoIP, Clipboard, Profiling, StackTrace
```

**Module Groups:**
- **Foundation**: Types, Classes, Common, EventMan
- **Geometry**: Geom2D, Geom3D, VertexLayout
- **Graphics**: Colors, FastGFX, Images, GfxFormats, GfxFilters, Regions
- **Text**: TextUtils, UnicodeFont, FreeTypeFont, GlyphCaches
- **Animation**: AnimatedValues, Tweenings
- **Network**: Socket, TCP, HttpRequests, GeoIP
- **Platform**: CrossPlatform, Android
- **Utilities**: Logging, Profiling, StackTrace, Clipboard, CPU
- **Specialized**: Crypto, RSA, Database, Translation, HtmlTree, ControlFiles
- **Auxiliary**: ProdCons, Huffman, ADPCM, LongMath, RegExpr, SCGI

### Engine (`Apus.Engine.*.pas`) - 57 modules

**Core**: GameApp, Game, API, Types
**Scenes**: Scene, SceneEffects, UIScene, ConsoleScene, TweakScene
**UI**: UITypes, UIWidgets, UI, UILayout, UIRender, UIScript, DefaultStyle
**Graphics**: Graphics, OpenGL, ResManGL, ShadersGL, Draw, TextDraw
**Resources**: Resources, ImageTools, ImgLoadQueue, NinePatch
**Platform**: WindowsPlatform, SDLplatform, AndroidGame, IOSgame
**3D**: Model3D, Mesh, OBJLoader, IQMloader, AEMLoader
**Audio**: Sound, SoundBass, SoundSDL, SoundImx

### Key Patterns

- **Interfaces**: `ISystemPlatform`, `IGraphicsSystem`, `IDrawer` for abstraction
- **Signals**: `"UI\Element\Click"` via `Link()` and `Apus.EventMan`
- **Scene lifecycle**: `Load()` (async) → `Initialize()` (fast) → `Process()` → `Render()`
- **Resources**: Reference counted, call `Free` when done
- **Singleton**: Global `game` object (NOT thread-safe, use `RunAsync`)

## Deprecated Code

**To remove/replace:**
- `Apus.Network.pas` → use `Apus.Socket` (marked deprecated 2023)
- `Apus.Engine.PainterGL.pas` / `Apus.Engine.PainterGL2.pas` - removed legacy painter backends
- `DxImages8.pas` → Direct3D 8 legacy
- `Apus.Engine.UdpTransport.pas` - symmetric UDP transport (legacy name: `Apus.Engine.Networking2`)
- `Apus.Engine.Networking3.pas` - renamed to `Apus.Engine.HttpGameClient`; no compatibility facade
- `deprecated/` folders in Base and root
- `bin/`, `bin64/` DLL files (moved/removed in git status)

## Test Coverage

**Existing tests:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cooler2/ApusGameEngine](https://github.com/Cooler2/ApusGameEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
