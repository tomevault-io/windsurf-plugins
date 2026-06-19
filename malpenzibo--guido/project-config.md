---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Guido is a reactive Rust GUI library using wgpu for rendering Wayland layer shell widgets (status bars, panels, etc.). The library emphasizes composition from minimal primitives, reactive properties, and GPU-accelerated rendering with animations.

**Note: Backward compatibility is NOT a concern for this project.** Feel free to remove legacy code, refactor APIs, and make breaking changes when it improves the codebase. The library is under active development and not yet stable.

## Documentation

### Developer Reference (`docs/`)

Quick-reference documentation for developers:

- **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** - System design, module structure, and code organization
- **[docs/STATE_LAYER.md](docs/STATE_LAYER.md)** - Hover/pressed state overrides, ripple effects, animations
- **[docs/TRANSFORMS.md](docs/TRANSFORMS.md)** - Translate, rotate, scale with transform origins and animations
- **[docs/REACTIVE.md](docs/REACTIVE.md)** - Signals, computed values, effects, and reactive patterns
- **[docs/STYLING.md](docs/STYLING.md)** - Colors, gradients, borders, corners, shadows, and layout

Read these docs before making significant changes to understand existing patterns.

### User Documentation (`book/`)

The `book/` directory contains an mdbook-based documentation website with tutorials, guides, and screenshots.

```bash
# Build the book
mdbook build book

# Serve locally with live reload
mdbook serve book
```

**IMPORTANT: Keep the book updated when making changes.**

When adding new features or changing APIs:
1. Update relevant chapters in `book/src/`
2. Add new screenshots if the feature has visual components (use `grim` to capture)
3. Build and verify the book renders correctly: `mdbook build book`

Key sections to update based on change type:
- **New widget methods** → `book/src/concepts/container.md` or relevant chapter
- **New styling options** → `book/src/building-ui/`
- **New state layer features** → `book/src/interactivity/`
- **New animation options** → `book/src/animations/`
- **New transform features** → `book/src/transforms/`
- **API changes** → Update all affected chapters and code examples

## Build and Development Commands

```bash
# Build the project
cargo build

# Run an example (status bar on Wayland layer shell)
cargo run --example status_bar

# Run the reactive example (demonstrates signals and events)
cargo run --example reactive_example

# Check for errors without building
cargo check

# Format code
cargo fmt

# Lint with clippy
cargo clippy

# Run tests
cargo test
```

## Architecture

### Core Modules

**`reactive/`** - Single-threaded reactive system inspired by SolidJS
- `RwSignal<T>`: Read-write reactive signal (8 bytes). Created via `create_signal` (requires Clone+PartialEq+Send). Has `.get()`, `.set()`, `.update()`, `.writer()`. Converts to `Signal<T>` via `.read_only()` or `.into()`
- `Signal<T>`: Read-only reactive signal (16 bytes). Created via `create_stored` (static, requires Clone) or `create_derived` (closure-backed). Has `.get()`, `.with()` — no mutation methods. Widget props accept `Signal<T>` via `IntoSignal<T, M>` (marker-type disambiguation — integers accepted where `f32`/`Length`/`Padding` expected)
- `Memo<T>`: Eager computed values that recompute when dependencies change, only notify on actual changes (`PartialEq`)
- `Effect`: Side effects that re-run when tracked signals change
- `Owner`: Ownership system for automatic resource cleanup (signals, effects, custom callbacks)
- Runtime uses thread-local storage for automatic dependency tracking on the main thread
- Container paint/layout auto-tracks signal reads via `with_signal_tracking()` — closures work as reactive properties
- Background threads update signals via `WriteSignal` (queued writes, flushed each frame)

**`widgets/`** - Composable UI primitives implementing the `Widget` trait
- `Container`: Handles padding, background colors, gradients, borders, corner radius, and event handlers (click, hover, scroll)
- `Row` / `Column`: Flexbox-style layouts with alignment and spacing
- `Text`: Text rendering with reactive content and styling
- `AnyWidget`: Type alias for `Box<dyn Widget>` with `Widget::into_any()` for type erasure
- All widget properties can be static values or reactive (via `IntoSignal` trait)
- `#[component]` macro: Creates reusable widgets from functions with reactive props, callbacks, children, and slots

**`renderer/`** - GPU rendering using wgpu
- SDF-based shape rendering with custom shader pipeline
- Supports rounded rectangles with superellipse corners (CSS K-value system)
- SDF-based border rendering for crisp anti-aliased borders with uniform width
- Supports circles, gradients, and clipping
- Text rendering via glyphon library
- HiDPI-aware with automatic scaling
- Layered rendering: shapes → text → overlay shapes (for effects like ripples)

**`platform/`** - Wayland layer shell integration
- Uses smithay-client-toolkit for Wayland protocol handling
- Supports layer shell positioning (top, bottom, overlay) and anchoring
- Keyboard interactivity modes (None, OnDemand, Exclusive)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MalpenZibo/guido](https://github.com/MalpenZibo/guido) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
