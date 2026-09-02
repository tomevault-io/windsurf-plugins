---
trigger: always_on
description: > AI Agent Guide for Allui Development
---

# AGENTS.md

> AI Agent Guide for Allui Development

This document provides comprehensive guidance for AI coding agents working on Allui, a SwiftUI-inspired declarative UI framework for Rust built on GPUI and gpui-component.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture & Design Philosophy](#architecture--design-philosophy)
- [Build & Development](#build--development)
- [Code Organization](#code-organization)
- [Critical GPUI Patterns](#critical-gpui-patterns)
- [Component Development](#component-development)
- [Modifier System](#modifier-system)
- [State Management](#state-management)
- [Testing](#testing)
- [Code Style & Conventions](#code-style--conventions)
- [Known Limitations & TODOs](#known-limitations--todos)
- [Common Pitfalls](#common-pitfalls)

---

## Project Overview

### What is Allui?

Allui brings SwiftUI's declarative, composable API to Rust desktop applications. If you know SwiftUI, you already know Allui.

**Key Stats:**
- **Lines of Code:** ~8,000+ across 46 files (39 src, 7 storybook)
- **Dependencies:** GPUI 0.2, gpui-component 0.5
- **Status:** Alpha
- **Platform:** macOS, Linux, Windows (wherever GPUI runs)

### Three-Layer Architecture

```
┌─────────────────────────────────────────┐
│        User Application Code            │
├─────────────────────────────────────────┤
│              Allui                      │
│  ┌──────────────┬───────────────────┐   │
│  │   Layout     │    Components     │   │
│  │  Primitives  │  (Display+Input)  │   │
│  ├──────────────┼───────────────────┤   │
│  │   Modifier   │   Control Flow    │   │
│  │   System     │   & Containers    │   │
│  └──────────────┴───────────────────┘   │
├─────────────────────────────────────────┤
│         gpui-component (v0.5.0)         │
│    (Stateful widgets: Switch, Input)    │
├─────────────────────────────────────────┤
│           GPUI (v0.2)                   │
│     (GPU-accelerated rendering)         │
└─────────────────────────────────────────┘
```

**Allui** provides SwiftUI API patterns and semantics
**gpui-component** provides stateful widgets (Switch, Input, Slider, etc.)
**GPUI** provides GPU-accelerated rendering and reactive state management

---

## Architecture & Design Philosophy

### Core Principles

1. **SwiftUI API Parity**
   - Names, methods, and behaviors match SwiftUI exactly
   - Layout behavior must be indistinguishable from SwiftUI
   - Modifiers have the same semantics (order matters)

2. **Presentational Components**
   - All Allui components are stateless
   - State lives in GPUI views (user's `impl Render` types)
   - Components render based on props passed to them

3. **True Modifier Wrapping**
   - Each modifier creates a wrapper `div()` container
   - `.padding().background()` ≠ `.background().padding()`
   - Matches SwiftUI's compositional model exactly

4. **Hybrid Component Strategy**
   - **Layout primitives:** Built directly on GPUI for precise control
   - **Display components:** Thin wrappers around GPUI primitives
   - **Input components:** Wrap gpui-component's stateful widgets

### Design Decisions

**Why no `@State` or `@Binding` equivalents?**
- Avoids reinventing GPUI's proven state system
- SwiftUI's property wrappers require compiler magic Rust doesn't support well
- More verbose but more explicit and debuggable
- Users already understand `Entity`, `Context`, `cx.notify()`

**Why wrapper `div()` for each modifier?**
- Exact SwiftUI semantics (order matters)
- Predictable behavior
- Matches user mental model from SwiftUI
- GPUI's GPU rendering makes nested divs efficient

**Why stable element IDs required?**
- GPUI's event system requires IDs to persist between renders
- Automatic ID generation breaks click handling
- Explicit > implicit for debugging

---

## Build & Development

### Setup

```bash
# Install dependencies
cargo build

# Run interactive component showcase
cargo run --example storybook

# Run with release optimizations (recommended for UI testing)
cargo run --example storybook --release
```

### Project Structure

```
allui/
├── src/
│   ├── lib.rs              # Crate root, public exports
│   ├── prelude.rs          # Convenient re-exports (use allui::prelude::*)
│   ├── types.rs            # Type aliases (ClickHandler)
│   ├── alignment.rs        # Alignment types with helper methods
│   ├── modifier.rs         # Modifier trait + implementations
│   ├── components/         # UI components
│   │   ├── mod.rs
│   │   ├── text.rs         # Display: Text
│   │   ├── button.rs       # Display: Button
│   │   ├── divider.rs      # Display: Divider
│   │   ├── image.rs        # Display: Image (TODO: actual loading)
│   │   ├── label.rs        # Display: Label
│   │   ├── link.rs         # Display: Link
│   │   ├── progress_view.rs # Display: ProgressView (TODO: animation)
│   │   ├── toggle.rs       # Input: Toggle
│   │   ├── text_field.rs   # Input: TextField
│   │   ├── secure_field.rs # Input: SecureField
│   │   ├── text_editor.rs  # Input: TextEditor
│   │   ├── slider.rs       # Input: Slider
│   │   ├── stepper.rs      # Input: Stepper
│   │   └── picker.rs       # Input: Picker
│   ├── layout/             # Layout primitives & containers
│   │   ├── mod.rs
│   │   ├── vstack.rs       # Vertical stack

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AntimaterialLabs/allui-rs](https://github.com/AntimaterialLabs/allui-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
