---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

sysc-Go is a terminal animation library for Go that provides multiple visual effects with customizable color themes. The project includes a Go library (`animations/`), a plain CLI (`syscgo`), a Bubble Tea TUI (`syscgo-tui`), and a FIGlet font converter (`flf2bit`).

## Build, Test, and Development Commands

```bash
# Build individual binaries
go build -o syscgo ./cmd/syscgo/
go build -o syscgo-tui ./cmd/syscgo-tui/
go build -o flf2bit ./cmd/flf2bit/

# Build all packages
go build ./...

# Run all tests
go test ./...

# Run specific test
go test -run TestName ./animations/

# Run with verbose output
go test -v ./...

# Lint
golangci-lint run
```

### Running Effects

```bash
# Default (fire effect, dracula theme, 10 seconds)
./syscgo

# Specific effect and theme
./syscgo -effect matrix -theme nord -duration 30

# Text-based effect with file input
./syscgo -effect decrypt -theme tokyo-night -file message.txt -duration 15

# Infinite duration
./syscgo -effect skull-text -file art.txt -theme rama -duration 0

# Launch interactive TUI
./syscgo-tui

# Convert FIGlet font to .bit format
./flf2bit myfont.flf output.bit
```

**All effects** (particle: fire, matrix, rain, fireworks, aquarium, beams; text: fire-text, matrix-art, rain-art, beam-text, ring-text, blackhole, pour, print, decrypt; scene: skull, skull-text, burn)

**All themes**: dracula, gruvbox, nord, tokyo-night, catppuccin, material, solarized, monochrome, transishardjob, rama, eldritch, dark

## Architecture

### Packages

```
sysc-Go/
├── animations/      # Core library: effects, palettes, registry, interfaces
├── tui/             # Bubble Tea TUI (syscgo-tui binary)
├── cmd/syscgo/      # Plain CLI
├── cmd/syscgo-tui/  # TUI entry point
├── cmd/flf2bit/     # FIGlet → .bit font converter
└── examples/        # Usage examples
```

### Core Interfaces (`animations/common.go`)

```go
type Animation interface {
    Update()        // Advance one frame
    Render() string // Return frame as string
    Reset()         // Restart animation
}

// Optional: effects that support runtime text changes
type TextUpdatable interface {
    SetText(text string)
}
```

**Palette system**: Each effect has a `Get*Palette(theme string) []string` function returning hex colors. `animations/palettes.go` centralizes all theme definitions.

**Effect registry**: `animations/registry.go` exports `EffectRegistry` and `ThemeRegistry` with metadata (name, category, `RequiresText` flag, version added). Use `IsTextBasedEffect(name)` to check if an effect requires a `-file` input.

**Effect configuration**: Complex effects use dedicated config structs (e.g., `DecryptConfig`, `PourConfig`, `PrintConfig`) passed to constructors.

### TUI Package (`tui/`)

Full interactive UI built with Bubble Tea. The `Model` struct manages:
- Animation selector (effect, theme, file, duration) with live preview
- Custom text editor (`editorMode`) with save/export to assets
- BIT text editor (`bitEditorMode`): banner text renderer using `.bit` JSON fonts (converted from FIGlet `.flf` files via `flf2bit`). Supports gradient, shadow, scale, alignment, kerning.

Assets (`.txt` files) are discovered from the `assets/` directory and shown in the file selector.

### Animation Implementation Pattern

When adding a new effect:
1. Create effect struct with width, height, palette, and internal state
2. Implement `Animation` interface; implement `TextUpdatable` if the effect takes text input
3. Add palette function to `animations/palettes.go`
4. Add entry to `EffectRegistry` in `animations/registry.go`
5. Add effect runner to `cmd/syscgo/main.go` (switch statement + runner function)
6. Add to the `animations` list in `tui/model.go` (`NewModel`)
7. Wire up in `tui/animfactory.go`
8. Implement `Resize(width, height int)` if the effect needs to handle terminal resize

### Key Dependencies

- `github.com/charmbracelet/lipgloss/v2` - Terminal styling (animations)
- `github.com/charmbracelet/bubbletea` + `bubbles` + `lipgloss` v1 - TUI
- `golang.org/x/term` - Terminal size detection
- `gonum.org/v1/gonum` - Math for physics-based effects

## Common Development Patterns

### Frame Rate and Timing
- Target 20 FPS: `time.Sleep(50 * time.Millisecond)` in CLI; `tea.Tick(50ms)` in TUI
- Duration: `frames = duration * 20` (0 = infinite)

### Terminal Handling (CLI)
```go
fmt.Print("\033[2J\033[H")  // Clear screen (setup)
fmt.Print("\033[?25l")      // Hide cursor
fmt.Print("\033[H")         // Move to top (per frame)
fmt.Print("\033[?25h")      // Show cursor (cleanup)
```

### Text Input
- Normalize line endings with `normalizeMultilineText()` (in `common.go`) before processing any text input
- Use `wrapText(text, width)` in `cmd/syscgo/main.go` to fit terminal width for text-based effects

### Adding a New Theme
1. Add a case for the theme name in every palette function in `animations/palettes.go`
2. Define 3–9 hex colors progressing from background to highlights
3. Add a `ThemeMetadata` entry to `ThemeRegistry` in `animations/registry.go`
4. Add to CLI help text in `cmd/syscgo/main.go` and the themes slice in `tui/model.go`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nomadcxx/sysc-Go](https://github.com/Nomadcxx/sysc-Go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
