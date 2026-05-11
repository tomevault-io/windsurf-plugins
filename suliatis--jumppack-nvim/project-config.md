---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Jumppack is a Neovim plugin that provides an enhanced navigation interface for Vim's jumplist. The plugin creates a floating window picker that allows users to visualize and navigate their jump history with preview functionality.

## Development Environment
- **Language**: Lua (LuaJIT)
- **LSP**: Configured via `.luarc.json` for Neovim development
- **Code Style**: StyLua with 2-space indentation, 120 character width
- **Plugin Structure**: Modular architecture in `lua/Jumppack/` (~3,276 lines across 9 files)

## Code Architecture

The plugin follows a **modular flat namespace design** with clean separation of concerns and explicit dependency injection.

### Module Structure

```
lua/Jumppack/
├── init.lua         (1,067 lines) - Main entry point, public API, config, logging
├── utils.lua        (341 lines)   - Utilities & logging (no dependencies)
├── hide.lua         (93 lines)    - Session-persistent hide system
├── filters.lua      (191 lines)   - Filter logic (file/cwd/hidden)
├── window.lua       (85 lines)    - Float window creation & management
├── display.lua      (546 lines)   - Rendering, formatting, preview
├── sources.lua     (190 lines)   - Jumplist processing & source creation
├── instance.lua     (565 lines)   - Instance lifecycle & state (singleton)
└── actions.lua      (198 lines)   - User action handlers
```

### Module Dependencies (Clean DAG)

```
utils (logging, errors, vim helpers)
  ↓
hide (session persistence)
filters (file/cwd/hidden filtering)
window (float windows)
  ↓
display (rendering, preview) ← filters, window
sources (jumplist processing) ← utils, filters, hide
  ↓
instance (lifecycle, state) ← window, display, filters, hide, sources
  ↓
actions (user handlers) ← instance, filters, hide, display
  ↓
init (public API, config) ← all modules
```

**Key architectural principles:**
- **No circular dependencies**: Clean dependency graph
- **Flat structure**: All modules export `H` table, accessed as `H.module_name.*`
- **Explicit dependencies**: Modules receive dependencies via `set_*()` injection functions
- **Singleton state**: Only `instance.lua` manages stateful singleton (active picker instance)
- **Pure functions**: Most modules are stateless, working with passed data

### Module Responsibilities

**init.lua** - Entry point
- Public API (`Jumppack.setup()`, `Jumppack.start()`, etc.)
- Configuration management (`H.config.*`)
- Logging system (`H.log.*`) with levels (trace/debug/info/warn/error)
- Module coordination and dependency injection

**utils.lua** - Foundation utilities
- Error handling (`H.error()`, `H.check_type()`)
- Buffer/window operations (`H.create_scratch_buf()`, `H.set_buflines()`)
- Vim API wrappers (safe operations that pcall internally)
- Path utilities (`H.full_path()`, `H.get_fs_type()`)
- No dependencies on other modules

**hide.lua** - Hide system
- Session-persistent hide management via `vim.g.Jumppack_hidden_items`
- Functions: `H.load()`, `H.save()`, `H.toggle()`, `H.is_hidden()`, `H.mark_items()`
- Storage: Newline-separated string (Vim sessions only save strings/numbers)
- Depends on: utils (for logging)

**filters.lua** - Filter logic
- Runtime filters: file_only, cwd_only, show_hidden
- Functions: `H.apply()`, `H.toggle_*()`, `H.reset()`, `H.is_active()`, `H.get_status_text()`
- Filters are NOT persistent (reset when picker closes)
- Depends on: utils (for path operations)

**window.lua** - Window management
- Float window creation and configuration
- Functions: `H.create_buffer()`, `H.create_window()`, `H.compute_config()`
- Constants: GOLDEN_RATIO (0.618), WINDOW_ZINDEX (251)
- Depends on: utils

**display.lua** - Rendering & preview
- ALL rendering logic: list view, preview mode, border updates
- Item formatting: `H.item_to_string()` with icon/path/position/preview
- Preview with syntax highlighting: `H.render_preview()`, `H.preview_set_lines()`
- Smart filename display for ambiguous names
- Depends on: utils, window, filters (injected), instance (injected)

**sources.lua** - Jumplist processing
- Processes Vim's jumplist into picker items
- Functions: `H.create_source()`, `H.get_all()`, `H.create_item()`, `H.find_target_offset()`
- Handles offset calculation for navigation (backward/forward)
- Depends on: utils, filters, hide

**instance.lua** - Instance lifecycle (stateful singleton)
- **ONLY module with singleton state**: `local active = nil`
- Instance creation, event loop, state management
- Functions: `H.create()`, `H.run_loop()`, `H.update()`, `H.set_items()`, `H.get_active()`
- Selection management: `H.set_selection()`, `H.move_selection()`, `H.get_selection()`
- Focus tracking and cleanup: `H.track_focus()`, `H.destroy()`
- Depends on: utils, window, display, filters, hide

**actions.lua** - User actions
- Action handlers for all keymaps
- Functions: `H.jump_back()`, `H.jump_forward()`, `H.choose()`, `H.toggle_*()`, etc.
- Integrates with instance, filters, and hide systems
- Depends on: instance, filters, hide, display, utils

### Key Components

- **Picker Interface**: Float window with item selection and preview (instance + display + window)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suliatis/Jumppack.nvim](https://github.com/suliatis/Jumppack.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
