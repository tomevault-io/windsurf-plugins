---
trigger: always_on
description: A short guide for AI agents working with the wxTerminalEmulator codebase.
---

# AGENTS.md — wxTerminalEmulator

A short guide for AI agents working with the wxTerminalEmulator codebase.

`CLAUDE.md` in the repository root is a symbolic link to this file. Editing one
edits both.

## Project Overview

wxTerminalEmulator is a cross-platform terminal emulation library for wxWidgets
applications, written in C++20. It provides a `wxTerminalViewCtrl` control that
embeds a working terminal into a wxWidgets application.

**Key facts:**

- Static library (`wxterminal_lib`) plus the demo application `glypht`
  ("GlyphT"), built from `src/glypht/`
- C++20, CMake 3.10+, wxWidgets 3.2.x (CI builds against `v3.2.8.1`; 3.3.x also
  works)
- VT parsing is done by a vendored copy of **libtsm** in `libtsm/`
- PTY: Windows (ConPTY), macOS and Linux (`forkpty`)
- Two renderers: an OpenGL glyph atlas and the older wxDC renderer
- UTF-8, ANSI/VT100 escape sequences, 256 colors and true color

## Documentation Ecosystem

| Resource | Location | Purpose |
|----------|----------|---------|
| **This file (AGENTS.md)** | Repository root | Quick-start guide for common tasks |
| **Detailed docs** | `.agents/summary/*.md` | Deeper reference for architecture, APIs, data models, and workflows |

### How to Use

1. **Start here (AGENTS.md)** for most tasks. It covers the directory layout,
   key entry points, build instructions, gotchas, and patterns.
2. **Go deeper into `.agents/summary/`** only when you need more detail:
   - `index.md` — map of all documentation files; the entry point for deep dives
   - `codebase_info.md` — repository facts and statistics
   - `architecture.md` — system architecture, design patterns, component
     interactions
   - `components.md` — component responsibilities and APIs
   - `interfaces.md` — public API reference, event system, integration patterns
   - `data_models.md` — data structures (`Cell`, `Lines`, `ColourSpec`, etc.)
   - `workflows.md` — data flow, rendering pipeline, escape sequence parsing,
     resize handling
   - `dependencies.md` — external dependencies and build requirements
   - `review_notes.md` — known documentation gaps and recommendations

> **Rule of thumb:** if AGENTS.md does not answer your question, read
> `.agents/summary/index.md` and follow the pointers to the right file.

The `.agents/summary/` files are older than the code. Trust the code first.

## Directory Organization

```
wxTerminalEmulator/
├── src/lib/                       # The library
│   ├── terminal_core.h/cpp        # Terminal engine, drives libtsm (no wx GUI code)
│   ├── terminal_view.h/cpp        # wxTerminalViewCtrl: rendering and input
│   ├── terminal_gl_renderer.h/cpp # OpenGL glyph-atlas renderer
│   ├── pty_backend.h              # Abstract PTY interface
│   ├── pty_backend_windows.h/cpp  # Windows ConPTY implementation
│   ├── pty_backend_posix.h/cpp    # Linux/macOS forkpty implementation
│   ├── keyboard_layout.h          # Keyboard layout translation interface
│   ├── keyboard_layout_mac.cpp    # macOS layout translation (Carbon)
│   ├── terminal_event.h/cpp       # Custom wxWidgets events
│   ├── terminal_theme.h           # Color schemes (dark and light presets)
│   └── terminal_logger.h/cpp      # Debug logging system
├── src/glypht/                    # GlyphT demo application
│   ├── main.cpp                   # Application entry point
│   ├── MainFrame.h/cpp            # Main frame, multi-tab notebook
│   ├── SettingsDlg.hpp/cpp        # Settings dialog
│   ├── wxTerminalUI.hpp/cpp       # Generated UI base classes (wxCrafter)
│   ├── app_persistence.h/cpp      # Application settings persistence
│   └── layout_persistence.h/cpp   # Window and layout persistence
├── libtsm/                        # Vendored libtsm (VT parser), own CMake target `tsm`
├── cmake/                         # CMake helpers, e.g. FindWxWidgetsMSYS.cmake
├── assets/                        # Icons and images
├── CMakeLists.txt                 # Build configuration
└── .github/workflows/             # CI: macos.yml, msys2.yml, ubuntu.yml
```

## Key Entry Points

| Component | File | Purpose |
|-----------|------|---------|
| **TerminalCore** | `src/lib/terminal_core.h` | Terminal state and libtsm driver |
| **wxTerminalViewCtrl** | `src/lib/terminal_view.h` | Embeddable terminal control |
| **PtyBackend** | `src/lib/pty_backend.h` | Interface for platform PTY implementations |
| **TerminalGLRenderer** | `src/lib/terminal_gl_renderer.h` | OpenGL renderer, only when `USE_OPENGL=1` |
| **Demo app** | `src/glypht/main.cpp` | Example usage with tabs, themes, menus |

## Repo-Specific Patterns

### Escape Sequence Parsing

- All parsing is done by **libtsm**. `TerminalCore` owns a `tsm_screen` and a
  `tsm_vte` and feeds bytes into them.
- Callbacks go back to `TerminalCore`: `TsmWriteCb` (the terminal answers the
  program), `TsmOscCb` (title and other OSC), `TsmBellCb` (BEL),
  `TsmDrawCb` (one cell of the screen).
- The visible screen is copied into the cell snapshot after each update, so the
  renderer never reads libtsm state directly.
- Do not add a hand-written escape parser to `TerminalCore`. Older versions had
  one; it is gone.

### Rendering Strategies

Two independent choices:

1. **Renderer**, chosen at configure time with `USE_OPENGL`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eranif/wxTerminalEmulator](https://github.com/eranif/wxTerminalEmulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
