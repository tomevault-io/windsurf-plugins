---
trigger: always_on
description: A Godot 4.6 + .NET 8.0 port of the Emuera text game engine. Emuera is a Japanese derivative of eramaker that executes `.ERB` script files and reads `.CSV` data files. This project replaces the original Windows Forms / GDI rendering stack with Godot nodes and a `System.Drawing` compatibility shim (`uEmuera` namespace).
---

# gemuera-c# (Godot Emuera Port)

A Godot 4.6 + .NET 8.0 port of the Emuera text game engine. Emuera is a Japanese derivative of eramaker that executes `.ERB` script files and reads `.CSV` data files. This project replaces the original Windows Forms / GDI rendering stack with Godot nodes and a `System.Drawing` compatibility shim (`uEmuera` namespace).

## Tech Stack

- **Engine**: Godot 4.6 (C# backend)
- **Framework**: .NET 8.0 (`Godot.NET.Sdk/4.6.2`)
- **Target Frameworks**: `net8.0` (desktop), `net9.0` (Android)
- **Rendering**: Mobile renderer, D3D12 on Windows, `canvas_items` stretch mode
- **Editor Plugins**: gdUnit4, godot_mcp
- **Excluded from compilation**: `uEmuera-0.2.9d/`, `XEmuera-0.5.1/` (reference/legacy code)

## Architecture

The project follows a 4-layer architecture:

```
┌─────────────────────────────────────────────────────────┐
│  Godot UI Layer (Nodes)                                 │
│  EmueraContent → HBoxContainer/VBoxContainer rows       │
│  EmueraImage, Button, Label, ColorRect, Inputpad, etc.  │
├─────────────────────────────────────────────────────────┤
│  Console Layer (GameView)                               │
│  EmueraConsole → ConsoleDisplayLine → parts             │
│  PrintStringBuffer, StringStyle, HtmlManager            │
├─────────────────────────────────────────────────────────┤
│  Process Layer (GameProc)                               │
│  Process → runScriptProc → Instruction execution        │
│  ErbLoader, LogicalLineParser, LabelDictionary          │
├─────────────────────────────────────────────────────────┤
│  Data Layer (GameData)                                  │
│  VariableEvaluator, ExpressionParser, GameBase          │
│  ConstantData, CharacterData, IdentifierDictionary      │
└─────────────────────────────────────────────────────────┘
```

## Entry Points & Game Loop

1. **`first_window.tscn`** → `FirstWindow._Ready()` scans for `era*` game folders and lists them.
2. User selects a game → scene changes to **`main.tscn`**.
3. **`EmueraMain._Ready()`** resolves `Sys.ExeDir`, loads SHIFT-JIS/UTF-8 config maps, resets `GlobalStatic`, sets up GPU renderer, adds `EmueraContent` to the scene tree, and starts `EmueraThread.instance.Start()`.
4. **`EmueraThread.Work()`** calls `Program.Main()` on a dedicated background `Thread`.
5. **`Program.Main()`** sets directory paths (`csv/`, `erb/`, `resources/`), loads config, creates a `MainWindow` stub, and starts the engine.
6. **`Process.Initialize()`** loads CSV/ERB files, builds `LabelDictionary`, and begins execution at the title screen.
7. **`Process.ScriptProc.runScriptProc()`** is the inner script execution loop.
8. **`EmueraConsole`** manages all output; **`GenericUtils`** forwards display operations to `EmueraContent`.
9. **User input** is captured by Godot UI nodes → forwarded to `EmueraThread.instance.Input()` → wakes the background thread via `ManualResetEventSlim` → fed into `console.PressEnterKey()`.

## Key Components

### Godot UI Layer (`Scripts/`)

| File | Role |
|------|------|
| `EmueraMain.cs` | Main entry Node; bootstraps engine, loads config maps, GPU work queue, starts `EmueraThread` |
| `EmueraThread.cs` | Runs `Program.Main()` on a background `Thread`; bridges Godot input with Emuera's blocking model via `ManualResetEventSlim` |
| `EmueraContent.cs` | Core Godot UI renderer; fixed-height line layout in VBoxContainer with overflow for images; manages menu bar, input pad, quick buttons, CBG layer, message boxes; enforces MaxVisibleLines=1000 node cap; `EffectiveLineHeight` adapts to font metrics |
| `EmueraImage.cs` | Custom `Control` that draws `Texture2D` / `AtlasTexture` regions via `_Draw()`; MouseFilter=Ignore |
| `SpriteManager.cs` | Static texture cache; loads images into `ImageTexture`; manages `AtlasTexture` sprites; `UpdateOtherThreads` processes max 1 texture per frame to avoid main-thread stalls |
| `GenericUtils.cs` | Bridge shims connecting `EmueraConsole` → `EmueraContent` (`AddText`, `ClearText`, `SetBackgroundColor`, `RefreshCBG`, etc.) |
| `FirstWindow.cs` | Launcher UI; scans for `era*` game folders (desktop: exe dir; Android: `/storage/emulated/0/emuera`) and transitions to `main.tscn` |
| `ColorMatrixGPU.cs` | GPU ColorMatrix shader material creation and uniform management |
| `SpriteDebugViewer.cs` | Debug overlay for sprite inspection (F3 toggle, desktop only) |
| `Inputpad.cs` | On-screen input pad for text/number entry |
| `QuickButtons.cs` | Quick-access button panel for recent choices |
| `Scalepad.cs` | UI scaling controls |
| `OptionWindow.cs` | Settings/options popup |

### Core Emuera Engine (`Scripts/Emuera/`)

| Layer | Key Files | Role |
|-------|-----------|------|
| **Config** | `Config/Config.cs`, `ConfigData.cs`, `ConfigCode.cs`, `ConfigItem.cs`, `KeyMacro.cs` | Engine configuration, key macros, replace dictionaries; Android overrides `WindowX` to screen width |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wwwXiaoHan17/gEmuera](https://github.com/wwwXiaoHan17/gEmuera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
