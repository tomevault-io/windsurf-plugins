---
trigger: always_on
description: dotnet build SageTUI.slnx
---

# SageTUI — Copilot Instructions

## Build & Test

```bash
# Build everything
dotnet build SageTUI.slnx

# Run all tests (~1,112 Expecto tests)
dotnet run --project SageTUI.Tests/SageTUI.Tests.fsproj

# Run a single test or filtered subset by name
dotnet run --project SageTUI.Tests/SageTUI.Tests.fsproj -- --filter "PackedCell roundtrip"
dotnet run --project SageTUI.Tests/SageTUI.Tests.fsproj -- --filter "Phase 3"

# Watch mode (rebuilds on .fs changes)
dotnet watch run --project SageTUI.Tests/SageTUI.Tests.fsproj

# Run benchmarks
dotnet run -c Release --project SageTUI.Benchmarks

# Run a sample
dotnet run --project samples/09-SystemMonitor
```

The test project (`SageTUI.Tests/`) is separate from the library. `SageTUI.Tests/Program.fs` calls `runTestsInAssemblyWithCLIArgs`. Test files are split by domain: `Tests.fs` (core/phases 0–10), `LayoutTests.fs`, `ScrollTests.fs`, `CanvasTests.fs`, `WidgetTests.fs`, `HitTestTests.fs`, `HtmlTests.fs`, `CssTests.fs`, `IntegrationTests.fs`, `SnapshotTests.fs`, `SampleShowcaseTests.fs`.

## Architecture

SageTUI is an F# TUI library using the **Elm Architecture** (Model-View-Update) with a **double-buffered SIMD diff** renderer targeting .NET 10.

### Render Pipeline

```
User Model → View function → Element tree → Render.render → back Buffer
→ Buffer.diff (chunk-skip) → Presenter.present → ANSI escapes → terminal
```

The diff compares 16-cell chunks (256 bytes) via `Span.SequenceEqual` (JIT-vectorized on .NET 8+), then falls through to per-cell comparison for dirty chunks. Only changed cells produce ANSI output.

### Compile Order (dependency chain)

The `.fsproj` compile order is load-bearing — F# requires files ordered by dependency. The actual order in `SageTUI.Library.fsproj`:

```
Color → Buffer → Layout → BorderRender → Element → Measure → CanvasRender
→ Arena → Effects → Terminal → Ansi → Render → ArenaRender → Input → Tea
→ Detect → Transition → App → Widgets → Scroll
```

New files must be inserted at the correct position. The core render pipeline is `Color → Buffer → Element → Render → Ansi → App`.

### Key Types

| Type | File | Description |
|------|------|-------------|
| `PackedCell` | Buffer.fs | 16-byte blittable struct `{Rune:int32; Fg:int32; Bg:int32; Attrs:uint16; _pad:uint16}` |
| `Color` | Color.fs | `Default \| Named(BaseColor,Intensity) \| Ansi256(byte) \| Rgb(byte,byte,byte)` |
| `PackedColor` | Color.fs | int32 bit-packed: tag in bits 0-1, payload in upper bits |
| `TextAttrs` | Color.fs | `{Value: uint16}` — 8 boolean flags OR'd together |
| `Style` | Color.fs | `{Fg: Color option; Bg: Color option; Attrs: TextAttrs}` |
| `Element` | Element.fs | 11-case DU: Empty, Text, Row, Column, Overlay, Styled, Constrained, Bordered, Padded, Keyed, Canvas |
| `Buffer` | Buffer.fs | `{Cells: PackedCell array; Width: int; Height: int}` |
| `Program<'model,'msg>` | Tea.fs | `{Init; Update; View; Subscribe}` — Elm Architecture record |
| `Cmd<'msg>` | Tea.fs | NoCmd, Batch, OfAsync, OfCancellableAsync, CancelSub, Delay, Quit |
| `Sub<'msg>` | Tea.fs | KeySub, TimerSub, ResizeSub, CustomSub |
| `TerminalProfile` | Terminal.fs | 5 capability enums (Color, Unicode, Graphics, Input, Output) |

### TEA Loop (App.fs)

`App.run` is the main loop:
1. Drain message queue → `program.Update` for each → new model + commands
2. Reconcile subscriptions via `program.Subscribe`
3. `program.View model` → Element tree
4. `Render.render` into back buffer
5. `Buffer.diff` front vs back → changed indices
6. `Presenter.present` changed cells → ANSI string → write to terminal
7. Swap front/back buffers
8. Poll for terminal events, dispatch to subscribers

### Terminal Detection (Detect.fs)

Reads environment variables (`COLORTERM`, `WT_SESSION`, `TERM`, `TERM_PROGRAM`, `LANG`, `TMUX`, `STY`) to build a `TerminalProfile` with capability levels. Multiplexer detection downgrades capabilities (tmux caps graphics at HalfBlock, screen caps color at Indexed256). User can override via `SAGETUI_COLOR`/`SAGETUI_UNICODE`/`SAGETUI_GRAPHICS` env vars.

## Conventions

### F# Style

- **2-space indentation** everywhere — never 4
- **Pattern matching over if/else for ALL control flow** — including booleans: use `match x with true -> ... | false -> ...` not `if/then/else`
- Pipeline operators (`|>`) for data transformations
- Immutable by default; `mutable` only at IO boundaries
- `Result<'T,'E>` for fallible operations, not Option (Option hides failure reasons)
- Discriminated unions for domain types representing finite exclusive choices
- Modules with same name as their type contain pure transformation functions

### Element Builder API

The `El` module is the public API for building UI trees:

```fsharp
El.column [
  El.bordered BorderStyle.Rounded (
    El.bold (El.fg Color.Green (El.text "Title"))
  )
  El.row [
    El.width 20 (El.text "Left")
    El.fill (El.text "Right")
  ]
]
```

### PackedColor Bit Layout

```
Tag (bits 0-1): 0=Default, 1=Named, 2=Ansi256, 3=Rgb
Named:  bits 2-4 = BaseColor index, bit 5 = Intensity
Ansi256: bits 8-15 = palette index
Rgb:    bits 8-15 = R, bits 16-23 = G, bits 24-31 = B
```

### Testing

Tests use **Expecto** with **Expecto.Flip** argument order, **FsCheck** for property-based tests, and **Verify.Expecto** for snapshot tests:

```fsharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WillEhrendreich/SageTUI](https://github.com/WillEhrendreich/SageTUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
