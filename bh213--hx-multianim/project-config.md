---
trigger: always_on
description: **hx-multianim** is a Haxe library for creating animations and pixel art UI elements using the [Heaps](https://heaps.io/) framework. It provides a custom `.manim` language for defining state animations and programmable UI components.
---

# Claude AI Instructions for hx-multianim

## Project Overview

**hx-multianim** is a Haxe library for creating animations and pixel art UI elements using the [Heaps](https://heaps.io/) framework. It provides a custom `.manim` language for defining state animations and programmable UI components.

## Key Technologies

- **Language**: Haxe
- **Framework**: Heaps (game/graphics framework)
- **Parser**: Custom hand-written lexer/parser in `MacroManimParser.hx` using `peek()`/`advance()`/`match()`/`expect()` (runs both at compile-time and runtime). `.anim` parser in `AnimParser.hx` is separate.
- **Package Manager**: Lix (recommended)

## Project Structure

| Path | Description |
|------|-------------|
| `src/bh/multianim/MultiAnimParser.hx` | Shared parser AST types + thin `parseFile` facade that delegates to `MacroManimParser`. Despite the name this is ~1200 lines of enum/typedef declarations (`NodeType`, `ReferenceableValue`, `TileSource`, `DefinitionType`, …), not parsing logic. Candidate for rename to `MultiAnimTypes.hx` / split. |
| `src/bh/multianim/MultiAnimBuilder.hx` | Builder for resolving parsed structures |
| `src/bh/multianim/MacroManimParser.hx` | Main parser for `.manim` files (used at both compile-time and runtime) |
| `src/bh/multianim/ProgrammableCodeGen.hx` | Macro code generation for `@:manim`/`@:data` |
| `src/bh/multianim/ProgrammableBuilder.hx` | Base class for macro-generated factories |
| `src/bh/multianim/LayoutAlignRoot.hx` | Base class for codegen instances with aligned layouts |
| `src/bh/multianim/dev/DevBridge.hx` | Dev JSON-RPC server for runtime inspection/manipulation (`-D MULTIANIM_DEV`) |
| `src/bh/multianim/dev/HotReload.hx` | Live `.manim`/`.anim` reload (`-D MULTIANIM_DEV`) |
| `src/bh/stateanim/AnimParser.hx` | Parser for `.anim` state animation files |
| `src/bh/multianim/ManimKeywordInfo.hx` | Keyword metadata for LSP/tooling (exhaustive switch on parser enums) |
| `src/bh/base/TweenManager.hx` | Tween/animation system (owned by ScreenManager) |
| `src/bh/ui/screens/ScreenManager.hx` | Screen stack, transitions, modal overlay, TweenManager owner |
| `src/bh/ui/screens/UIScreen.hx` | Screen base class + interactives/helpers wiring |
| `src/bh/ui/screens/UIScrollableScreen.hx` | Scrollable screen base (whole-screen mousewheel scrolling) |
| `src/bh/ui/controllers/` | `UIDefaultController` + interaction controllers (SelectFromHand, PickTarget) |
| `src/bh/ui/UIMultiAnimGrid.hx` | Grid higher-order component (rect/hex, drag-drop, card targeting) |
| `src/bh/ui/UICardHandHelper.hx` | Card hand higher-order component (draw/discard, targeting, combining) |
| `src/bh/ui/ScreenShakeHelper.hx` | Additive screen shake for impact feedback |
| `src/bh/ui/FloatingTextHelper.hx` | AnimatedPath-driven floating text manager |
| `lsp/` | LSP language server for `.manim`/`.anim` (compiles to JS via `-D noheaps`) |
| `vscode/` | VS Code extension: syntax highlighting, language config, LSP client |
| `test/` | Test suite |

## Build & Run Commands

```bash
# Compile the library
haxe ./hx-multianim.hxml

# Run tests (parsing and rendering verification)
test.bat run        # Run all tests
test.bat run 7      # Run only test #7
test.bat gen-refs   # Generate reference images
test.bat report     # Open test report in browser
```

`test.bat` output is optimized for AI parsing (structured key-value results). Running `hl build/hl-test.hl` directly produces raw trace output not designed for automated consumption.

```bash
# LSP server
haxe lsp/lsp-server.hxml       # Compile LSP server to lsp/bin/server.js
haxe lsp/test-lsp.hxml          # Compile LSP tests
node lsp/bin/test.js             # Run LSP tests
```

Playground lives in a separate repository: `../hx-multianim-playground`. Usually running at: http://localhost:3000, if not you can `npm run dev`

## Workflow

1. **Parsing**: `MacroManimParser` converts `.manim` file text to AST with `Node` structures
2. **Building**: `MultiAnimBuilder` resolves references, expressions, and type conversions (runtime)
3. **Macro codegen**: `MacroManimParser` parses `.manim` at compile time, `ProgrammableCodeGen` generates typed Haxe classes

## File Formats

### `.manim` - Multi Animation / UI Elements
Used for programmable UI components, layouts, palettes, and paths.

### `.anim` - State Animations
Used for sprite state animations with playlists. Free-form layout (newlines are whitespace). Structure:

```anim
sheet: sheetName
states: stateName(value1, value2)
center: x,y
allowedExtraPoints: [point1, point2]
fps: 20

@final OFFSET_X = 5

metadata {
    health: 100
    speed: 1.5
    tint: #FF0000
    @(state=>value) damage: 50
    @(state=>other) damage: 30
}

animation animationName {
    fps: 20
    loop: yes | <number>
    playlist {
        sheet: "sprite_${state}_name"
        event <name> trigger | random x,y,radius | x,y
        filter tint: #FF0000
        filter none
    }
    filters {
        tint: #FF0000
        brightness: 0.8
        @(state=>value) outline: 2.0, #FFFF00
        @else pixelOutline: #00FF00
        replaceColor: [#FF0000] => [#0000FF]
    }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bh213/hx-multianim](https://github.com/bh213/hx-multianim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
