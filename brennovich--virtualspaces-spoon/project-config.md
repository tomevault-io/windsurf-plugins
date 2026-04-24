---
trigger: always_on
description: Provides window metadata extraction and comparison utilities:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

VirtualSpaces is a Hammerspoon Spoon that implements virtual workspace management for macOS, similar to i3/Linux window managers. It provides instant workspace switching by managing window visibility across two physical macOS spaces (active and storage).

## Commands

### Running Tests
```bash
# Recommended: Run tests via make
make test

# Or via luarocks
luarocks test

# Direct invocation (requires luarocks path setup)
eval $(luarocks --local path) && lua tests/test.lua -o TAP

# Without TAP output
lua tests/test.lua

# Run acceptance tests
make test/acceptance
```

### Building and Installation
```bash
# Install dependencies
make dependencies

# Build the spoon (creates release/VirtualSpaces.spoon.zip)
make build

# Build and install to ~/.hammerspoon/Spoons (sets debug telemetry level)
make install

# Generate documentation JSON
make docs.json
```

### Testing Individual Modules
Tests are organized by module in `tests/`. Each test file can be run independently by requiring it from `tests/test.lua`:
- `tests/test_windows_sort.lua` - WindowsSort logic
- `tests/test_spaces_model.lua` - SpacesModel state management
- `tests/test_native_space_manager.lua` - NativeSpaceManager setup
- `tests/test_move_window_to_virtual_space.lua` - Window movement tests
- `tests/test_space_watcher.lua` - Space watcher behavior
- `tests/test_telemetry.lua` - Telemetry/instrumentation tests
- `tests/test_window_cache.lua` - WindowCache caching logic
- `tests/test_get_windows_api.lua` - Public API for getting windows
- `tests/test_public_api.lua` - Public API tests

### Development
This is a Hammerspoon Spoon. Load it in Hammerspoon with:
```lua
hs.loadSpoon("VirtualSpaces")
spoon.VirtualSpaces:init()

-- Enable telemetry for debugging
spoon.VirtualSpaces:instrument('info')   -- Log operations only
spoon.VirtualSpaces:instrument('debug')  -- Log operations + performance
```

## Architecture

### Three-Layer Design

1. **SpacesModel** (`SpacesModel.lua`): Pure state management
   - Tracks window-to-virtual-space mappings (`_windowVirtualSpaceMap`)
   - Tracks virtual-space-to-windows mappings (`_virtualSpaceWindowsMap`)
   - Tracks focused window per virtual space (`_focusedWindows`)
   - Current virtual space tracking (`_currentVirtualSpace`)
   - Tabbed window management:
     - `_windows` - registry of window metadata (id, tabCount, frame, appName)
     - `_tabGroups` - groups of windows that share the same tabs
     - `_windowToTabGroup` - mapping from window ID to its tab group
   - No side effects, pure data operations

2. **WindowsSort** (`WindowsSort.lua`): Window movement logic
   - Takes categorized windows and maps them to native spaces
   - Handles space swapping when user navigates to storage space
   - Injects `windowMoverFn` (typically `hs.spaces.moveWindowToSpace`)
   - Injects `windowSpaceGetter` (typically `hs.spaces.windowSpaces`) to check current window location
   - Skips window moves when window is already in target space (performance optimization)

3. **NativeSpaceManager** (`NativeSpaceManager.lua`): macOS Spaces setup
   - Ensures exactly two native spaces exist on main screen
   - Designates first space as "active", second as "storage"
   - Handles space creation/removal on initialization

### Window Module (`Window.lua`)

Provides window metadata extraction and comparison utilities:
- `Window.new(hsWindow)` - Creates window metadata object with:
  - `id` - window ID
  - `tabCount` - number of tabs (from `hsWindow:tabCount()`)
  - `frame` - window frame/position
  - `appName` - application name
- `Window.makeKey(frame, appName)` - Creates composite key for indexing windows by position and app
- `Window.framesEqual(frame1, frame2)` - Compares two frames with Y_TOLERANCE for y-coordinate
- `Window.Y_TOLERANCE` - Tolerance (10 pixels) for y-coordinate comparison to handle Terminal.app tabs
- Used by SpacesModel to detect and group tabbed windows (windows with same frame + app)

### Tabbed Windows Support

VirtualSpaces automatically detects and manages tabbed windows (Safari tabs, Terminal tabs, etc.):

**Detection Strategy:**
- Tabbed windows share similar frame positions (with 10-pixel y-coordinate tolerance) and belong to the same application
- Y-coordinate tolerance handles Terminal.app tabs which have slightly different y positions
- Uses `hsWindow:tabCount()` to identify windows with multiple tabs
- Groups windows by matching `frame + appName` combination

**Behavior:**
- When a tabbed window is moved to a virtual space, all tabs in the group move together
- When a tab is closed, focus is restored to a sibling tab if available
- Tab groups are automatically created/updated as windows are created or destroyed
- Maintains tab group consistency across virtual space transitions

**Implementation:**
- SpacesModel maintains tab group registry indexed by frame+appName
- Tab detection occurs during `registerWindowObject()` on window creation
- `_assignWindowAndTabGroupToVirtualSpace()` ensures entire tab groups move atomically

### Telemetry System (`Telemetry.lua`)

Provides observability for operations and performance:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brennovich) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
