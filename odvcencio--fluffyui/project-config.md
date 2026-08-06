---
trigger: always_on
description: This document provides essential context for AI agents and contributors working on FluffyUI, a Go-based terminal UI framework with sub-cell graphics, reactive state, and accessibility support.
---

# AGENTS.md - Contribution Guide for FluffyUI

This document provides essential context for AI agents and contributors working on FluffyUI, a Go-based terminal UI framework with sub-cell graphics, reactive state, and accessibility support.

## Project Overview

FluffyUI is a comprehensive terminal UI framework featuring:
- **28+ packages** organized by responsibility
- **35+ widgets** with consistent patterns
- **Sub-cell graphics** using Braille/Sextant/Quadrant rendering
- **Reactive state** via signals and computed values
- **Two backends**: tcell (terminal), sim (testing)
- **Built-in accessibility** with screen reader support
- **Agent integration** via MCP protocol

**Go Version:** 1.24+

---

## Directory Structure

```
fluffyui/
├── runtime/          # Core app loop, render pipeline, message handling
├── widgets/          # 35+ reusable UI components
├── state/            # Reactive signals and computed values
├── animation/        # Tweens, springs, particle systems
├── graphics/         # Sub-cell canvas with shapes and curves
├── gpu/              # GPU canvas and drivers (software/OpenGL/Metal)
├── backend/          # Backend abstractions
│   ├── tcell/        # Real terminal (tcell v2)
│   └── sim/          # Simulation for testing
├── keybind/          # Keyboard routing and command registry
├── forms/            # Form validation and coordination
├── accessibility/    # Screen reader support, focus management
├── agent/            # Out-of-process agent server with MCP
├── recording/        # Asciicast and video export
├── audio/            # Music and SFX service
├── compositor/       # Screen buffer and ANSI rendering
├── theme/            # Theme management
├── style/            # Style system (colors, attributes)
├── toast/            # Toast notifications
├── scroll/           # Virtual scrolling utilities
├── effects/          # Visual effects (gradients, glow)
├── dragdrop/         # Drag-and-drop interfaces
├── terminal/         # Terminal abstraction (keys, mouse)
├── clipboard/        # Clipboard abstraction
├── markdown/         # Markdown parsing and highlighting
├── progress/         # Progress tracking
├── testing/          # Test helpers
├── examples/         # 19+ example applications
├── docs/             # Comprehensive documentation
└── scripts/          # Recording and agent tools
```

---

## Architecture

### Message-Loop Architecture

```
Event Loop (tick-based, 30 FPS)
    │
    ├── Backend Input (keys, mouse, resize)
    ├── Timers
    └── Custom Events (postings)
    │
    ▼
Message Loop
    │
    ▼
handleMessage (root widget)
    │ - Routes input to focused widget
    │ - Collects commands (bubbling)
    │
    ▼
Render Pipeline
    1. Measure (top-down constraints)
    2. Layout (position assignment)
    3. Render (to buffer)
    4. Diff (dirty cell tracking)
    5. Show (to backend)
```

### Key Interfaces

```go
// Core widget interface (runtime/widget.go)
type Widget interface {
    Measure(constraints Constraints) Size
    Layout(bounds Rect)
    Render(ctx RenderContext)
    HandleMessage(msg Message) HandleResult
}

// Focusable widgets
type Focusable interface {
    IsFocused() bool
    SetFocused(bool)
    CanFocus() bool
}

// Reactive binding
type Bindable interface {
    Bind(services Services)
}
type Unbindable interface {
    Unbind()
}

// Container traversal
type ChildProvider interface {
    ChildWidgets() []Widget
}
```

### Constraint-Based Layout

Three-phase rendering:
1. **Measure**: Parent provides Constraints, widget returns preferred Size
2. **Layout**: Parent assigns Rect bounds to widget
3. **Render**: Widget draws to RenderContext.Buffer

```go
type Constraints struct {
    MinWidth, MaxWidth, MinHeight, MaxHeight int
}
type Size struct { Width, Height int }
type Rect struct { X, Y, Width, Height int }
```

---

## Coding Conventions

### Naming

| Element | Convention | Example |
|---------|------------|---------|
| Packages | lowercase, single word | `widgets`, `state`, `keybind` |
| Types | PascalCase | `Button`, `Signal`, `GridLayout` |
| Option functions | `With*` prefix | `WithVariant`, `WithDisabled` |
| Constructors | `New*` prefix | `NewButton`, `NewSignal` |
| Accessors | `Get*/Set*` | `GetValue`, `SetLabel` |
| Predicates | `Is*/Can*` | `IsFocused`, `CanFocus` |
| Interface suffixes | `-able`, `-Provider` | `Focusable`, `BoundsProvider` |

### File Organization

```go
package widgets

// 1. Imports (stdlib, then third-party, then local)
import (
    "fmt"

    "github.com/external/pkg"

    "fluffyui/runtime"
)

// 2. Type definition
type Button struct {
    widgets.FocusableBase
    label *state.Signal[string]
}

// 3. Constructor
func NewButton(label string, opts ...ButtonOption) *Button { ... }

// 4. Option type and functions
type ButtonOption func(*Button)
func WithVariant(v Variant) ButtonOption { ... }

// 5. Interface implementations (Measure, Layout, Render, etc.)
func (b *Button) Measure(constraints runtime.Constraints) runtime.Size { ... }
func (b *Button) Render(ctx runtime.RenderContext) { ... }

// 6. Other methods
func (b *Button) SetLabel(label string) { ... }
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [odvcencio/FluffyUI](https://github.com/odvcencio/FluffyUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
