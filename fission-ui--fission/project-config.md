---
trigger: always_on
description: **Last Updated:** December 15, 2025
---

# Fission Framework - Project Knowledge Base (`GEMINI.md`)

**Last Updated:** December 15, 2025
**Scope:** Architecture, Design Principles, Implementation Details, and Development Workflows.

---

## 1. Project Vision & Core Principles

Fission is a next-generation UI framework built on the principle that **UI is a pure function of state**, enforced rigorously through architectural constraints. It aims to solve the "correctness" and "testability" problems in modern UI development.

### Core Principles
1.  **Determinism First:** Given the same input state, the framework *must* produce the exact same pixel output and internal state, regardless of platform, time (mocked), or previous history.
2.  **UI = f(State):** The UI is rebuilt from scratch (logically) whenever state changes. There is no manual DOM manipulation.
3.  **Separation of Concerns:**
    *   **Authoring Layer:** Expressive, open-world API for developers.
    *   **Core Runtime:** Closed-world, platform-agnostic engine handling layout, diffing, and state.
    *   **Shell:** Thin, platform-specific adapter (Windowing, GPU context, Media decoding).
    *   **Renderer:** Pure function turning Display Lists into pixels (Skia).
4.  **Zero-Closure Tree:** The widget tree contains *data*, not behavior. Event handlers are registered alongside the tree, not embedded within it as closures.
5.  **Owned Time:** The framework owns the clock. Animations and physics are deterministic functions of this explicit clock, enabling perfect frame-by-frame replay and testing.

---

## 2. Architecture Overview

The system is divided into four distinct layers:

### Layer 1: Authoring (`crates/authoring`)
*   **Role:** The "Frontend" API used by app developers.
*   **Key APIs:** `Widget`, `View<S>`, `Selector<S>`, and `fission::build::current::<S>()`.
*   **Output:** Produces public `Widget` values via `From<Component> for Widget`.
*   **State:** Reads `AppState` (User) and `RuntimeState` (Framework) via `View`.

### Layer 2: Core (`crates/core`)
*   **Role:** The "Brain". Handles logic, diffing, layout, and event routing.
*   **Key Components:**
    *   **Runtime Compilation:** Compiles authored `Widget` values into `CoreIR` (Ops).
    *   **Runtime:** Manages `AppState`, `RuntimeState` (Scroll, Focus, Anim), and dispatches `Actions`.
*   **Layout:** Computes geometry using a constraint-based layout engine (Flutter-style).
    *   **Diffing:** Compares `CoreIR` trees to optimize updates (Hybrid Retained Mode).
*   **Data:** `CoreIR` is a flat list of `Op`s (Layout, Paint, Semantics).

### Layer 3: Shell (`crates/shell`)
*   **Role:** The "Body". Interfaces with the OS.
*   **Responsibilities:**
    *   Creates Windows/Surfaces (`winit`, `softbuffer`).
    *   Captures OS Events (Mouse, Keyboard, Window).
    *   Provides Media Backends (Video decoding, Audio).
    *   Runs the Event Loop.

### Layer 4: Renderer (`crates/rendering`)
*   **Role:** The "Painter".
*   **Input:** `DisplayList` (Platform-agnostic drawing commands).
*   **Implementation:** `fission-render-vello` is the maintained GPU renderer backend.
*   **Policy:** All rendering must be anti-aliased and resolution-independent.

---

## 3. Core Concepts & Implementation

### 3.1 Authoring: `From<Component> for Widget`
Widgets are pure data structs. Components implement `From<Component> for Widget` to compose other widgets or primitives.

```rust
impl From<MyWidget> for Widget {
    fn from(component: MyWidget) -> Widget {
        let (ctx, view) = fission::build::current::<S>();
        let val = view.select::<MySelector>();
        Button {
            on_press: Some(ctx.bind(MyAction, on_action)),
            child: Some(Box::new(Text { ... }.into())),
            ...Default::default()
        }.into()
    }
}
```

### 3.2 Runtime Compilation: `Widget` Values to `CoreIR`
Framework-owned runtime compilation turns authored `Widget` values into low-level `CoreIR` operations (`Op`).
*   **Public Boundary:** App authors do not implement rendering/lowering traits or return renderer internals.
*   **Primitives:** `Button`, `Text`, `Row`, and other built-ins are compiled by framework internals.
*   **Custom Rendering:** Specialized render paths are framework-owned and exposed to apps only as normal values that convert into `Widget`.
*   **Result:** A list of `LayoutOp` (Box, Flex, Scroll), `PaintOp` (Rect, Text, Image), and `Semantics` (Roles, Actions).

### 3.3 The Action System
*   **Action:** A pure data struct deriving `Action`.
*   **Binding:** `ctx.bind(Action, Handler)` registers the handler in the `ActionRegistry`.
*   **Envelope:** The tree stores an `ActionEnvelope` (ID + Payload), not the closure.
*   **Dispatch:** The Runtime receives the envelope, looks up the handler in the registry, and invokes it with mutable access to `AppState`.

### 3.4 Layout Engine
*   **Backing:** Constraint-based layout engine (Flutter-style).
*   **Integration:** `fission-layout` consumes `LayoutOp` directly.
*   **Text:** `TextMeasurer` trait abstracts platform text sizing (implementation-dependent).
*   **Scroll:** Implemented via unbounded constraints in the scroll axis + clamping in the runtime.

### 3.5 Rendering
*   **Display List:** A serializable list of `DisplayOp` (DrawRect, DrawText, Clip, Translate).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fission-ui/fission](https://github.com/fission-ui/fission) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
