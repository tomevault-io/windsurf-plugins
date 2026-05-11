---
trigger: always_on
description: Handles full VT100/ANSI: cursor, erase, SGR (16/256/true-colour, underline styles), alternate screen,
---

# Guidelines

## Normative Keywords

- **MUST**: mandatory requirement
- **SHOULD**: recommended but optional with justification
- **MAY**: optional

## Core Principles

- MUST follow idiomatic Go patterns
- MUST minimize dependencies
- MUST use explicit error handling
- SHOULD keep functions <= 50 lines
- SHOULD prefer composition over inheritance patterns

## File Formats

- MUST use Markdown for documentation files

## Libraries

- MUST use log/slog for logging

## Reading order for new contributors

- **Tutorial**: [`doc/tutorial/README.md`](doc/tutorial/README.md) — chapters from a 20-line teaser through a real SQLite query tool. Every snippet has a runnable program under `doc/tutorial/examples/`.
- **Reference**: [`doc/reference/overview.md`](doc/reference/overview.md) — one page per widget.
- **Design principles**: [`doc/principles.md`](doc/principles.md) — the small set of invariants the framework relies on.
- **Events / flags**: [`doc/events.md`](doc/events.md), [`doc/flags.md`](doc/flags.md).

## Project Overview

This is a TUI component library based on `tcell/v3`. Key types:

- **`Widget`** (`widget.go`) — interface for all UI elements; application code works against this type
- **`Component`** (`component.go`) — base struct that satisfies `Widget`; embed in every custom widget
- **`Container`** (`container.go`) — extends `Widget`; manages children and layout
- **`Style`** (`style.go`) — CSS-like styling (colors, borders, margins, padding); hierarchical/inheritable
- **`Theme`** (`theme.go`) — registry of styles indexed by CSS-like selectors (`type.class#id:state`)
- **`Renderer`** (`renderer.go`) — drawing abstraction over tcell; widgets MUST NOT import tcell directly
- **`UI`** (`ui.go`) — root container, event loop, focus management, render pipeline; takes the whole screen
- **`Builder`** (`builder.go`) — fluent API to construct and style a widget tree; preferred way to build UIs
- **`Animation`** (`animation.go`) — embed in widgets that need timed animation; manages goroutine + ticker

### Architecture rules

- All widgets embed `Component` and implement `Render(*Renderer)` and `Apply(*Theme)`
- Every new widget MUST be registered in `Builder` (add a method) and `compose/compose.go` (add a function)
- Containers call `child.SetBounds` and `child.SetParent` during `Layout()`; never touch tcell directly
- Rendering is top-down: `UI.Draw` → each layer → each container → each child
- Events bubble from focused widget up through parents; return `true` from a handler to stop propagation
- `Redraw(widget)` queues a single-widget redraw; call after any state change that affects rendering

### Selector format

`type/part.class#id:state` — all parts optional.
Examples: `"button"`, `"button.primary"`, `"button#submit:focused"`, `"input/placeholder"`.

## Project Structure

```
zeichenwerk/
├── *.go              # Core library (widgets, theme, renderer, …)
├── cmd/
│   ├── compose/      # Compose API demo
│   ├── demo/         # Builder API demo (all widgets)
│   └── showcase/     # Full interactive showcase
├── compose/          # Functional composition API (Option-based)
├── doc/              # Design proposals and reference docs
├── spec/             # Widget specifications (written before implementation)
└── .claude/
    └── skills/
        └── zeichenwerk/
            ├── SKILL.md    # Claude Code skill — auto-loaded in this project
            └── widgets.md  # Full widget reference for the skill
```

## Widget Reference

For the full style-key, event, and method reference per widget see
`.claude/skills/zeichenwerk/widgets.md`.

### Containers

| Widget        | File             | Constructor                                                    | Key methods                                                                                |
| ------------- | ---------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `Flex`        | `flex.go`        | `NewFlex(id, class, alignment Alignment, spacing int)`         | `Add(Widget)` — set `FlagVertical` for vertical layout                                     |
| `Grid`        | `grid.go`        | `NewGrid(id, class, rows, cols int, lines bool)`               | `Add(content Widget, params ...any)` — params are `x, y, w, h`; `Rows(…int)`, `Columns(…)` |
| `Box`         | `box.go`         | `NewBox(id, class, title)`                                     | `Add(Widget)` — single child                                                               |
| `Card`        | `card.go`        | `NewCard(id, class, title)`                                    | `Add(Widget)` — first call = content, second = footer                                      |
| `Switcher`    | `switcher.go`    | `NewSwitcher(id, class)`                                       | `Add(Widget)`, `Select(any)` — dispatches `EvtShow`/`EvtHide` to panes when `connect`      |
| `Viewport`    | `viewport.go`    | `NewViewport(id, class, title)`                                | `Add(Widget)` — scrollable single child                                                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tekugo/zeichenwerk](https://github.com/tekugo/zeichenwerk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
