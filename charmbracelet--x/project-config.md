---
trigger: always_on
description: This file documents essential information for AI agents working in the pony codebase.
---

# AGENTS.md - Guide for AI Agents Working on pony

This file documents essential information for AI agents working in the pony codebase.

## Project Overview

**pony** is a declarative, type-safe markup language for building terminal user interfaces. It uses [Ultraviolet](https://github.com/charmbracelet/ultraviolet) as the rendering engine and integrates with [Bubble Tea](https://github.com/charmbracelet/bubbletea).

- **Language**: Go 1.24.2
- **Module**: `github.com/charmbracelet/x/pony`
- **Primary dependencies**: 
  - `github.com/charmbracelet/ultraviolet` (UV rendering)
  - `github.com/charmbracelet/x/ansi` (ANSI parsing)
  - `golang.org/x/text` (text processing)
- **Experimental**: This is primarily AI-generated and experimental

## Essential Commands

All commands use Task (Taskfile.yaml) or standard Go tooling.

### Testing
```bash
# Run all tests
task test
go test ./...

# Run tests with verbose output
go test -v ./...

# Run specific test
go test -run TestRender

# Run tests with coverage
task test:coverage

# Update golden test files (IMPORTANT for rendering tests)
task test:update
go test -update ./...
```

### Linting & Formatting
```bash
# Run linter (uses .golangci.yml config)
task lint

# Run linter with auto-fix
task lint:fix

# Format code (gofumpt + goimports)
task fmt

# Install golangci-lint
task lint:install
```

### Maintenance
```bash
# Clean build artifacts and test cache
task clean

# Tidy dependencies
task tidy
go mod tidy

# List all available tasks
task
```

## Code Organization

### Main Source Files (Root)

Core element types and primitives:
- `element.go` - Element interface, Constraints, Size types
- `box.go` - Box container with borders, padding, margin
- `text.go` - Text element with styling and alignment
- `container.go` - VStack, HStack container implementations
- `zstack.go` - ZStack (layering/overlay)
- `flex.go` - Flex wrapper for flexible sizing
- `positioned.go` - Positioned element (absolute positioning)
- `spacer.go` - Spacer element (fixed/flexible)
- `divider.go` - Divider element
- `scrollview.go` - ScrollView with scrollbar support
- `button.go` - Button component with click handling

Template and parsing:
- `template.go` - Template[T] type with Go template integration
- `parser.go` - XML parser (markup → element tree)
- `slot.go` - Slot system for dynamic content injection

Styling and layout:
- `style.go` - Style parsing and builder API
- `layout.go` - Size constraint helpers
- `constants.go` - Border, alignment, unit constants

Component system:
- `registry.go` - Global component registry
- `components.go` - Built-in components (Badge, Progress, Header)

Interactivity:
- `bounds.go` - BoundsMap, hit testing, BaseElement
- `doc_interactive.go` - Mouse interaction documentation

Helpers:
- `helpers.go` - Layout and style helper functions

### Test Files

Tests follow `*_test.go` naming convention. Key test files:
- `element_test.go` - Element layout and rendering
- `parser_test.go` - XML parsing
- `style_test.go` - Style parsing and rendering
- `template_test.go` - Go template integration
- `layout_test.go` - Size constraints
- `alignment_test.go` - Alignment
- `bounds_test.go`, `bounds_hitall_test.go` - Hit testing
- `scrollview_test.go`, `scrollview_props_test.go`, `scrollview_scrollbar_test.go` - Scrolling
- `slot_test.go`, `slots_bounds_test.go` - Slot system
- `input_click_test.go`, `nested_interactive_test.go` - Mouse interactions

### Test Data

- `testdata/` - Golden files for rendering tests (`.golden` extension)
- Golden files are named after test functions: `TestMyFunc.golden` or `TestMyFunc/subtest.golden`

### Examples

- `examples/` - Complete working examples
- Each example has its own `go.mod` and `main.go`
- Examples demonstrate features: hello, layout, styled, dynamic, components, custom, stateful, scrolling, bubbletea, buttons, etc.

## Code Patterns & Conventions

### Element Pattern

All elements implement the `Element` interface:
```go
type Element interface {
    uv.Drawable                          // Draw(scr uv.Screen, area uv.Rectangle)
    Layout(constraints Constraints) Size // Calculate size
    Children() []Element                 // Return child elements (or nil)
    ID() string                          // Element identifier
    SetID(id string)                     // Set identifier
    Bounds() uv.Rectangle                // Last rendered bounds
    SetBounds(bounds uv.Rectangle)       // Set bounds (call in Draw)
}
```

### BaseElement Embedding

**All element types embed `BaseElement`** to get ID and bounds tracking:
```go
type MyElement struct {
    BaseElement  // Required for ID and bounds
    // ... other fields
}
```

`BaseElement` provides:
- `ID()` - Returns explicit ID or pointer-based ID (`elem_%p`)
- `SetID(string)` - Set explicit ID
- `Bounds()` - Get last rendered bounds
- `SetBounds(uv.Rectangle)` - Record bounds (call at start of `Draw()`)

### Fluent API Pattern

**All elements use method chaining** for configuration:
```go
box := NewBox(child).
    WithBorder("rounded").
    WithPadding(2).
    WithMargin(1)

text := NewText("Hello").
    WithStyle(style).
    WithAlignment"center")
```

Method naming: `With<Property>(*Element) *Element`

### Draw() Implementation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [charmbracelet/x](https://github.com/charmbracelet/x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
