---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.
See [agents.md](https://agents.md/) for the specification.

## Project Overview

TamboUI is a Java library for building modern terminal user interfaces, inspired by Rust's ratatui and Go's bubbletea. It uses immediate-mode rendering with an intermediate buffer system for diff-based terminal updates.

## Build and Test Commands

```bash
# Build the project
./gradlew -q assemble

# Run all tests
./gradlew -q test

# Run a single test class
./gradlew -q :tamboui-core:test --tests "dev.tamboui.buffer.BufferTest"

# Publish to local Maven repository
./gradlew publishToMavenLocal

# Publish to build directory (preferred for inspection)
./gradlew publishAllPublicationsToBuildRepository
# Artifacts appear in build/repo/

# Run a demo on JVM
./run-demo.sh sparkline-demo

# Run a demo as native executable (requires GraalVM)
./run-demo.sh sparkline-demo --native

# Alternative: install and run demo manually
./gradlew :demos:sparkline-demo:installDist
./demos/sparkline-demo/build/install/sparkline-demo/bin/sparkline-demo

# Compile demo to native image
./gradlew :demos:sparkline-demo:nativeCompile
```

## Module Structure

| Module                | Purpose |
|-----------------------|---------|
| `tamboui-core`        | Core types: Buffer, Cell, Rect, Style, Layout, Text, Widget/StatefulWidget interfaces, InlineDisplay |
| `tamboui-widgets`     | All widget implementations (Block, Paragraph, List, Table, Chart, Canvas, etc.) |
| `tamboui-jline`       | JLine 3 terminal backend implementation |
| `tamboui-tui`         | High-level TUI framework: TuiRunner, event handling, bindings, action handlers |
| `tamboui-toolkit`     | Fluent DSL for declarative UI with retained-mode elements, focus management, event routing |
| `tamboui-css`         | CSS-based styling with TCSS format, selectors, cascade resolution, theme switching |
| `tamboui-annotations` | Annotation definitions (`@OnAction`) for action handling |
| `tamboui-processor`   | Annotation processor for compile-time action handler generation (avoids reflection) |
| `tamboui-image`       | Image rendering with multiple protocols (Kitty, iTerm, Sixel, HalfBlock, Braille) |
| `tamboui-picocli`     | PicoCLI integration for CLI argument parsing |
| `**/demos/*`          | Demo applications showcasing features |

## Architecture

### Rendering Model

1. **Widget interface** (`tamboui-core`): Stateless widgets implement `Widget.render(Rect, Buffer)`. Stateful widgets implement `StatefulWidget<S>.render(Rect, Buffer, S)`.

2. **Buffer system** (`tamboui-core`): Widgets render to a `Buffer` (2D grid of `Cell`s). The `Terminal` diffs buffers between frames for efficient updates.

3. **Layout system** (`tamboui-core`): `Layout` with `Constraint`s (length, percentage, ratio, min, max, fill) splits areas. `Rect` represents rectangular regions.

### Application Layers

**Low-level (immediate mode):**
```java
Terminal<Backend> terminal = new Terminal<>(BackendFactory.create());
terminal.draw(frame -> widget.render(frame.area(), frame.buffer()));
```

**Mid-level (TuiRunner):**
```java
try (var tui = TuiRunner.create(TuiConfig.builder().mouseCapture(true).build())) {
    tui.run((event, runner) -> { /* handle events */ return shouldRedraw; },
            frame -> { /* render widgets */ });
}
```

**High-level (Toolkit DSL):**
```java
import static dev.tamboui.toolkit.Toolkit.*;

class MyApp extends ToolkitApp {
    protected Element render() {
        return panel("Title", text("Hello").bold().cyan());
    }
}
```

### Event System

- `TuiRunner` provides the main event loop with `EventHandler` callback
- Event types: `KeyEvent`, `MouseEvent`, `TickEvent`, `ResizeEvent`, `UiRunnable`
- `KeyEvent` provides semantic methods (`isQuit()`, `isUp()`, `isDown()`, `isSelect()`, etc.) that respect configured bindings
- Bindings map physical inputs to semantic actions; use `BindingSets.standard()`, `BindingSets.vim()`, or custom bindings
- `@OnAction` annotation on Component methods handles actions; use annotation processor for compile-time generation
- Toolkit elements handle events via `handleKeyEvent()`/`handleMouseEvent()` or handler lambdas

### Threading Model

- TamboUI TUI framework uses a dedicated **render thread model** similar to JavaFX or Swing
- All rendering and UI state modifications must happen on the render thread (the thread running `TuiRunner.run()`)
- `RenderThread.isRenderThread()` checks if current thread is render thread; `RenderThread.checkRenderThread()` asserts it
- `TuiRunner.runOnRenderThread(Runnable)` executes on render thread (immediately if already on it, queued otherwise)
- `TuiRunner.runLater(Runnable)` always queues for later execution
- Scheduled actions via `ToolkitRunner.schedule()` run on scheduler thread; use `runOnRenderThread()` for UI state changes
- Thread checks only enforce when render thread is set (allows unit tests without special setup)

### Key Packages

- `dev.tamboui.buffer` - Buffer, Cell for rendering
- `dev.tamboui.layout` - Rect, Constraint, Layout, Direction
- `dev.tamboui.style` - Style, Color, Modifier
- `dev.tamboui.text` - Text, Span, Line for styled text

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tamboui/tamboui](https://github.com/tamboui/tamboui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
