---
trigger: always_on
description: This document details the core philosophy, system architecture, native Rust
---

# Architecture and Code Style Guide for `xnano`

This document details the core philosophy, system architecture, native Rust
binding structures, and high-level Python code style conventions for the
`xnano` package.

## Library Structure

`xnano` is a Python multi-surface UI framework (TUI first, with web and CLI
surfaces) built on **`xnano-core`**, which exposes ratatui, crossterm, and
tachyonfx through PyO3.

The public DSL (`BaseGrid`, `Field`, components, `@on_*` hooks, `Action`,
`Style`, `render`) is interface-neutral. Presentation hosts are top-level
modules (`xnano.terminal`, `xnano.web`) plus the `xnano.cli` package. Shared
runtime machinery lives under `xnano.core`. Helpers live under `xnano.utils`.
HTTP serving lives under `xnano.server`.

```
User app (BaseGrid + Field + @on_* hooks + Action)
        |
        v
   xnano                 public DSL: grids, fields, hooks, components, …
        |
        +-- xnano.core   Runtime, Frame, content, controller, stage,
        |                dispatch, layout, exceptions, demo
        |
        +-- xnano.terminal   Terminal host (wraps Runtime)
        +-- xnano.web        Web host (offscreen Runtime + native server)
        +-- xnano.cli        Command CLI abstraction
        +-- xnano.server     NativeWebServer + RequestServer
        +-- xnano.utils      focus, validation, markup, dispatch helpers
        |
        v
   xnano_core.core       session, scene graph, render IR, unified events
        |
        v
   xnano_core.rust.native   raw ratatui/crossterm/tachyonfx bindings
```

### `xnano` — public DSL and package layout

**Location:** `xnano/` (version `1.1.7`; depends on `xnano-core==0.0.14`).

The package root lazily exports:

| Export | Module |
|--------|--------|
| `render` | `xnano.rendering` |
| `Action` | `xnano.actions` |
| `BaseGrid`, `GridSettings` | `xnano.grids` |
| `Field` | `xnano.fields` |
| `Context` | `xnano.context` |
| `Terminal` | `xnano.terminal` |
| `Web` | `xnano.web` |
| `Runtime`, `Frame` | `xnano.core` |
| `Style` | `xnano.tailwind` |
| `Component` | `xnano.components` |
| `Command` | `xnano.cli` |
| `@on_*` / `@on_action` | `xnano.hooks` |
| `hooks`, `requests`, `cli`, `components`, `core`, `events` | package barrels |

There is **no** `Grid` alias — use `BaseGrid`. Import components and supporting
types from their concrete modules (for example
`from xnano.components.text import Text`).

```
xnano/
├── __init__.py, __main__.py, py.typed
│
│  ── public DSL (flat modules) ────────────────────────────────────────
├── actions.py             # Action hierarchy + Actions performer
├── colors.py              # Color, ColorLike, Tailwind palette helpers
├── context.py             # Context passed to hooks
├── cursor.py              # Cursor controls
├── device.py              # Device / display controls
├── effects.py             # Effect descriptions (native lowering is TUI)
├── events.py              # Event types + core→Event conversion
├── fields.py              # Field, FieldInfo / GridFieldInfo, FieldState
├── grids.py               # BaseGrid, GridSettings
├── hooks.py               # @on_* / @on_action decorators
├── markdown.py            # Markdown viewport + CLI document runner
├── rendering.py           # print-like render() helper
├── requests.py            # HTTP request hooks + Request/Response
├── state.py               # State helper
├── tailwind.py            # Style + Tailwind class resolution
├── terminal.py            # Terminal host
├── types.py               # geometry, sizing, frame, keyboard/mouse/focus
├── web.py                 # Web host
│
│  ── packages ─────────────────────────────────────────────────────────
├── components/            # Bar, Button, Chart, Component, Dropdown,
│                          # Image, Input, Link, Loader, Markdown,
│                          # Options, Scrollbar, Table, Text, …
├── core/                  # Runtime, Frame, content, controller, stage,
│                          # dispatch, layout, interface, exceptions, demo
├── cli/                   # Command, Option, Argument, help
├── server/                # NativeWebServer, RequestServer
└── utils/                 # focus, validation, markup, introspection,
                           # dispatch, responsive, deprecation
```

#### Key abstractions and flow

- **`BaseGrid`** / **`Field`** — declarative layout and state fields.
  Subclass `BaseGrid` and annotate slots with `Field(...)`. Use
  `state=True` for non-rendered app data.
- **`Action`** — declarative/imperative triggers. Events answer *"what
  happened"*; actions answer *"what to do"*. Hooks bind actions; the
  runtime's `Actions` performer synthesizes them. Matching is centralized
  on `Action.matches`.
- **`Runtime`** — owns a `CoreSession`, the run loop, viewport mode,
  cursor/device controls, focus, stage, and paint pipeline. Prefer
  `Terminal` for apps; use `Runtime` directly when you need explicit
  session ownership. `get_active_runtime()` returns the context-local
  active runtime.
- **`Terminal`** (`xnano.terminal`) — presentation host over `Runtime`.
  Selects a live session when a TTY is available, otherwise offscreen.
  `Terminal.offscreen(...)` is the test path. `Terminal.run(...)` paints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hsaeed3/xnano](https://github.com/hsaeed3/xnano) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
