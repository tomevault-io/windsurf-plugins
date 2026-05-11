---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Martillo** (Spanish for "hammer") is a powerful, declarative configuration framework for [Hammerspoon](https://www.hammerspoon.org/) that provides fast, ergonomic, and reliable productivity tools for macOS. The project aims to be an open-source Raycast alternative built with Lua and Hammerspoon.

### Vision & Philosophy

- **Declarative Configuration**: Single-line setup inspired by [lazy.nvim](https://github.com/folke/lazy.nvim)
- **Zero Dependencies**: Pure Lua implementation with no external libraries or compilation required
- **Developer-First**: Built by developers, for developers who want powerful automation without leaving the keyboard
- **Extensible**: Easy to create custom bundles and actions with beautiful 3D icons
- **Performance**: Lightweight, fast, and runs entirely in the background
- **Auto-Launch**: ActionsLauncher opens automatically on Hammerspoon load/reload

### Current Status
- **Beta**: Core functionality is stable and ready for daily use
- Comprehensive set of productivity actions
- Active development with regular improvements
- 11 built-in action bundles with 60+ actions

## Architecture

### Core Structure
- **`martillo.lua`**: Main framework module that handles spoon loading, configuration, and hotkey binding
- **`spoons/`**: Collection of custom Hammerspoon spoons (productivity tools)
- **`bundle/`**: Action bundles (window, system, utilities, converter, keyboard, clipboard_history, etc.)
- **`store/`**: External custom actions directory (folder-based with init.lua)
- **`lib/`**: Shared library modules (icons, search, navigation, leader)
- **`assets/`**: Static resources (120+ 3D icons from 3dicons.co)

### Key Design Patterns

1. **Declarative Configuration**: Users configure everything through a single `require("martillo").setup({ ... })` call

2. **Action Bundles**: Modular action collections in `bundle/` directory that can be selectively imported

3. **Store Structure**: External actions in `store/` as folders with `init.lua` for easy distribution

4. **Icon System** (`lib/icons.lua`):
   - 120+ beautiful icons from 3dicons.co in `assets/icons/`
   - `icons.preset.iconName` - Get absolute path to preset icon
   - `icons.getIcon(path)` - Load icon from absolute path
   - `icons.ICON_SIZE` - Standard icon size ({ w = 32, h = 32 })
   - Automatic discovery from `assets/icons/` and `store/*/` directories
   - Store icons can override default icons
   - Automatic caching for performance

5. **Leader Key Support**: `<leader>` placeholder in hotkeys expands to configured `leader_key` modifiers

6. **Child Chooser Pattern**: Actions that open child choosers call `spoon.ActionsLauncher:openChildChooser({...})` without needing to return any value. The framework automatically detects when a child chooser is opened and manages the navigation stack internally.

7. **Action Events** (`lib/events.lua`):
   - `events.copyToClipboard(getText?)` - Copy to clipboard with toast
   - `events.copyAndPaste(getText?)` - Copy + paste with Shift modifier support
   - `events.showToast(getMessage?)` - Show toast message
   - `events.noAction()` - Display-only (no action on Enter)
   - `events.custom(fn)` - Custom handler function

8. **Action Options System**:
   - Actions can define default options via `opts` field
   - Users can override options in their config: `{ "action_id", opts = { option_name = value } }`
   - Options are merged by `martillo.lua` during action processing
   - Access options via `events.getActionOpt(actionId, optionName, defaultValue)`
   - Common option: `success_toast` (default: true) - disable success toast notifications

9. **Shared Modules**:
   - `lib/icons.lua` - Icon management with automatic discovery and caching
   - `lib/events.lua` - Composable action helpers for common patterns + action options helper
   - `lib/search.lua` - Fuzzy search with ranking
   - `lib/chooser.lua` - Chooser state management (stack-based navigation with Tab support)
   - `lib/leader.lua` - Leader key expansion
   - `lib/thumbnail_cache.lua` - Disk-based thumbnail caching with memory limits
   - `lib/toast.lua` - Toast notification helpers

## Built-in Spoons

### ActionsLauncher
**Purpose**: Searchable command palette with selective action loading, per-action keybindings, and 3D icons

**Key Features**:
- Fuzzy search with alias support
- Beautiful 3D icons for all actions
- Icon inheritance for child choosers
- Per-action keybindings
- Child choosers (query-based transformations)
- Auto-opens on Hammerspoon load/reload

**Configuration**:
```lua
{
  "ActionsLauncher",
  actions = {
    { "window_center", keys = { { "<leader>", "return" } } },
    { "clipboard_history", alias = "ch" },
    { "f1_standings", alias = "f1" },  -- From store/
  },
  keys = { { "<leader>", "space", desc = "Toggle Actions Launcher" } }
}
```

**Note:** All bundles from `bundle/` and custom actions from `store/` are automatically loaded. You don't need to manually import them!

### LaunchOrToggleFocus
**Purpose**: Ultra-fast application switching

**Key Features**:
- Single hotkey per app

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sjdonado/martillo](https://github.com/sjdonado/martillo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
