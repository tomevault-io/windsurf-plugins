---
trigger: always_on
description: > Target docs: **ratatui 0.30.0** (docs.rs “latest” at time of writing).
---

# Ratatui (Rust) — TUI “field guide” for building a full-featured CLI
> Target docs: **ratatui 0.30.0** (docs.rs “latest” at time of writing).  
> This file is meant to be a **practical, complete map** of Ratatui’s public surface area: *what exists, what it’s for, how it fits together, and where to read the authoritative docs / examples.*

---

## 0) What Ratatui is (and what it is not)

**Ratatui** is an *immediate-mode* terminal UI library: each frame, your app renders the entire UI into an off-screen **Buffer**, and Ratatui diff-renders only changes to the terminal via a **Backend**.

**Key implications of “immediate mode”:**
- Your “UI tree” is not persisted by the framework.
- **Your app owns the state** (selection index, scroll offset, input text, toggles).
- Each `Terminal::draw` call must fully repaint the UI for the current state (even if nothing changed).

---

## 1) Crate architecture & “components” at a glance

Ratatui is modular (a workspace of crates) and the top-level `ratatui` crate re-exports what you usually need:

### 1.1 Core building blocks
- **Terminal + Frame** (render loop entry points)
- **Backend** (how bytes reach a real terminal)
- **Buffer + Cell** (the “canvas” widgets draw to)
- **Layout** (splitting areas with constraints)
- **Text** (`Span`, `Line`, `Text`)
- **Style** (`Color`, `Style`, `Modifier`, `Stylize`)
- **Widgets** (Block, Paragraph, Table, …)
- **Symbols** (box drawing, markers, sets)
- **Init utilities** (`run`, `init`, `restore`, viewports)

### 1.2 “Where things live” (major modules)
- `ratatui::backend` — the `Backend` trait, backend implementations (Crossterm/Termion/Termwiz/TestBackend), and backend-related traits.
- `ratatui::buffer` — `Buffer`, `Cell`.
- `ratatui::layout` — `Rect`, `Layout`, `Constraint`, `Direction`, `Flex`, `Margin`, alignment types, etc.
- `ratatui::style` — `Style`, `Color`, `Modifier`, and convenience styling (`Stylize`).
- `ratatui::text` — `Span`, `Line`, `Text`, and helpers.
- `ratatui::symbols` — line/box characters, border sets, markers.
- `ratatui::widgets` — all built-in widgets, widget traits, widget states.
- `ratatui::init` — documentation & helpers around init/restore and panic hooks (also see the root-level functions).
- `ratatui::prelude` — a convenience glob-import set (still available, but not universally recommended).

### 1.3 Feature flags (important!)
Ratatui uses feature flags to control:
- which backend(s) are included (e.g. `crossterm`, `termion`, `termwiz`)
- whether “all widgets” are built (default is typically “all”)
- macros, layout cache, underline color support, serde, palette integration, etc.

**Authoritative list (always check your exact version):**
```text
https://docs.rs/crate/ratatui/latest/features
```

---

## 2) “Hello World” mental model: the rendering pipeline

### 2.1 The minimal moving parts
1. **Backend** talks to the terminal emulator (stdout, raw mode, alternate screen, cursor).
2. **Terminal<B>** owns:
   - double buffers (previous + current)
   - viewport configuration
   - cursor state
3. Each frame:
   - `Terminal::draw(|f| { ... })`
   - you use the `Frame` to render widgets into the current `Buffer`
   - Ratatui diffs current vs previous buffer
   - backend writes only changed cells

### 2.2 Key types you’ll touch daily
- `Terminal<B>`
- `Frame`
- `Rect`
- `Buffer`
- `Layout`, `Constraint`
- `Style`, `Color`, `Modifier`
- `Span`, `Line`, `Text`
- widget types (`Block`, `Paragraph`, `Table`, …)

### 2.3 Immediate-mode best practice (non-negotiable)
**Always fully render the frame.**  
Do not “skip” rendering unchanged areas. Diffing happens *after* your closure returns.

---

## 3) Terminal initialization & teardown (the “right” way in 0.30)

Ratatui now provides convenience helpers so you don’t need boilerplate raw-mode / alternate-screen / panic hook setup in every project.

### 3.1 `run()` — simplest “just do the right thing”
Use this for most CLIs:

```rust
fn main() -> std::io::Result<()> {
    ratatui::run(|terminal| {
        loop {
            terminal.draw(|f| f.render_widget("Hello", f.area()))?;
            // read events, update state, maybe exit...
        }
    })
}
```

Docs:
```text
https://docs.rs/ratatui/latest/ratatui/fn.run.html
```

### 3.2 `init()` / `restore()` — explicit control
If you want the terminal value and to control shutdown:
- `init()` initializes a default terminal (Crossterm-based).
- `restore()` returns terminal settings to normal (disable raw mode, leave alt-screen).

Docs:
```text
https://docs.rs/ratatui/latest/ratatui/fn.init.html
https://docs.rs/ratatui/latest/ratatui/fn.restore.html
```

### 3.3 Viewports (`Viewport`) & `TerminalOptions`
Ratatui supports different ways to “own” the terminal:
- **Fullscreen**: typical TUI (entire screen)
- **Inline(height)**: persistent TUI area while normal output scrolls above it (status panels, progress, logs)
- **Fixed(Rect)**: render into a fixed region

Docs:
```text
https://docs.rs/ratatui/latest/ratatui/enum.Viewport.html
https://docs.rs/ratatui/latest/ratatui/struct.TerminalOptions.html
https://docs.rs/ratatui/latest/ratatui/fn.init_with_options.html
```

Examples (inline viewport):
```text
https://ratatui.rs/examples/apps/inline/
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tigranbs/mcgravity](https://github.com/tigranbs/mcgravity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
