---
trigger: always_on
description: Guidance for AI agents working in the PTerm repository.
---

# AGENTS.md

Guidance for AI agents working in the PTerm repository.

## What this repo is

PTerm (`github.com/pterm/pterm`) is a Go library for building beautiful,
cross-platform terminal output. It is a **library, not an application** — there
is no `main` package at the root and nothing to "run". Consumers import it and
call printers like `pterm.Info.Println(...)` or `pterm.DefaultTable.WithData(...).Render()`.

It works across Windows CMD, macOS terminals, Linux, and CI systems, degrading
gracefully (falls back from TrueColor → ANSI → no color, and to raw text when
styling is unsupported or disabled).

- Module path: `github.com/pterm/pterm` (Go 1.26+)
- Root package `pterm` holds all the printers. Each printer lives in its own
  `*_printer.go` file with a matching `*_printer_test.go`.
- `putils/` — optional helper utilities (build tables from CSV/structs, letters
  from strings for BigText, download-with-progressbar, etc.). Depends on `pterm`.
- `internal/` — non-exported helpers: color-level detection & downsampling
  (`internal/color`), snapshot test harness (`internal/snapshot`), text
  measurement/centering, cancelation signals, etc.
- `_examples/` — runnable example programs, one subfolder per printer. The
  leading underscore keeps them out of the normal build. These are the source
  of truth for the README examples and the VHS animations.
- `docs/`, `README.md` — docs. Large sections of the README are generated (see
  the `<!-- ... -->` marker comments); do not hand-edit generated regions.

## How printers are set up

Every printer follows the same conventions. When adding or changing one, match
the existing pattern exactly.

1. **A struct + a `Default*` value.** Each printer is a struct (e.g.
   `SectionPrinter`) with an exported package-level default instance
   (`DefaultSection`). Users start from the default and customize it.

2. **Builder pattern with value receivers.** Configuration methods are
   `WithX(...)` methods that take a **value receiver**, mutate the copy, and
   return a **pointer** to the copy:

   ```go
   func (p SectionPrinter) WithLevel(level int) *SectionPrinter {
       p.Level = level
       return &p
   }
   ```

   This means `With*` never mutates the printer it was derived from — chaining
   is safe and side-effect-free. Preserve this; do not switch `With*` to pointer
   receivers.

3. **Printers belong to one of four families** (documented and compile-time
   enforced in `printers.go`):

   - **`TextPrinter`** (`interface_text_printer.go`) — prints formatted text
     directly. Implements `Sprint/Sprintln/Sprintf/Sprintfln` (return string)
     and `Print/Println/Printf/Printfln` (write to output) plus
     `PrintOnError(f)`. Examples: `BasicTextPrinter`, `PrefixPrinter` (powers
     `Info`/`Success`/`Warning`/`Error`/`Fatal`/`Debug`), `HeaderPrinter`,
     `SectionPrinter`, `BoxPrinter`, `CenterPrinter`, `Color`, `RGB`.
   - **`RenderPrinter`** (`interface_renderable_printer.go`) — renders complex
     multi-line content via `Render()` (to output) and `Srender()` (to string).
     Examples: `TablePrinter`, `TreePrinter`, `BarChartPrinter`,
     `BulletListPrinter`, `PanelPrinter`, `HeatmapPrinter`, `BigTextPrinter`.
   - **`LivePrinter`** (`interface_live_printer.go`) — output updates in place
     over time. `Start()` returns the started instance; `Stop()` terminates it.
     Examples: `SpinnerPrinter`, `ProgressbarPrinter`, `AreaPrinter`,
     `MultiPrinter`.
   - **Interactive printers** — read user input and return a result from
     `Show()`. No shared interface (return types differ) but follow the same
     `Default*` + `With*` + `Show()` shape. Examples:
     `InteractiveConfirmPrinter`, `InteractiveSelectPrinter`,
     `InteractiveMultiselectPrinter`, `InteractiveTextInputPrinter`,
     `InteractiveContinuePrinter`.

   `printers.go` has compile-time assertions (`var _ TextPrinter = ...`) that
   enforce family membership. If you add a printer, add its assertion there.

4. **The `Sprint` method is the core.** For text printers, `Print*` methods
   delegate to `Sprint*`, which delegate to `Fprint`/the package writers. Put
   the actual rendering logic in `Sprint`; the rest is boilerplate that follows
   the section_printer.go template.

## Global output, color, and styling

- Output goes through `print.go` (`Print`, `Fprint`, `Printo`, etc.). The
  default writer is `os.Stdout`; override with `SetDefaultOutput` or a
  printer's `WithWriter`.
- Global toggles live in `pterm.go`: `Output` (all output), `PrintColor`
  (color), `RawOutput` (styling). **Do not read/write these vars directly for
  concurrency** — use the `Enable*`/`Disable*` functions, which take
  `globalMu`. `PrintColor` is auto-detected at init from the environment
  (`NO_COLOR`, `TERM=dumb`, `FORCE_COLOR`, legacy Windows console) via
  `internal/color`.
- Theme: all default styles come from `ThemeDefault` in `theme.go`. Printers
  reference theme styles by pointer (e.g. `&ThemeDefault.SectionStyle`).
- Color: `color.go` (ANSI 3/4-bit `Color`), `rgb.go` (TrueColor `RGB` with
  fading/gradients). Color downsampling to the terminal's capability lives in
  `internal/color`.

## Build, test, lint

A `Taskfile.yml` exists — prefer its tasks:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pterm/pterm](https://github.com/pterm/pterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
