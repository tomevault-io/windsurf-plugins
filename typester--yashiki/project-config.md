---
trigger: always_on
description: macOS tiling window manager written in Rust.
---

# Yashiki

macOS tiling window manager written in Rust.

## Workspace Notes

- Version control: `jj` (not `git`)
- Before starting work:
  1. Run `jj workspace root` to get the workspace root path
  2. Run `jj status` to confirm current workspace state
- All file edits must target files under the workspace root path
- Update only this CLAUDE.md, not the root one
- **jj write operations (commit, describe, new, etc.) are done by the user, not Claude** (unless explicitly requested)
  - When user explicitly asks (e.g., "jj describeして", "commitして"), execute the command
  - For PRs: output title and description text, let the user handle jj/git operations

## Project Structure

```
yashiki/                  # WM core daemon + CLI
yashiki-ipc/              # Shared protocol definitions (commands, layout)
yashiki-layout-tatami/    # Tile layout engine (master-stack)
yashiki-layout-byobu/     # Accordion layout engine (stacked windows)
```

Future: `engawa/` (status bar), `yashiki-layout-rasen` (spiral), `yashiki-layout-koushi` (grid)

## Architecture

### Thread Model

- **Main thread**: CFRunLoop - Accessibility API, global hotkeys (CGEventTap), window operations
- **Tokio runtime**: IPC server (Unix Domain Socket), event forwarding

### Communication

- IPC/Hotkey commands → main thread via `std::sync::mpsc` + `CFRunLoopSource`
- `CFRunLoopSourceSignal` + `CFRunLoopWakeUp` for immediate processing (no polling delay)
- Layout engine: stdin/stdout JSON (synchronous, from main thread)

### Virtual Workspaces (No SIP Required)

Like AeroSpace, uses virtual workspaces instead of macOS native Spaces:
- All windows on single macOS Space, workspace switching hides windows (per-display corner positioning)
- Only uses public Accessibility API, NSScreen visibleFrame for layout area

**Why per-display hiding?** Native fullscreen moves a display to a separate macOS Space where Accessibility API cannot access windows. If windows were hidden to a global position (e.g., bottom-right of all displays), they would become inaccessible when any display enters fullscreen. Per-display hiding ensures each display's windows stay within its own bounds.

## Key Features

- **Multi-monitor support** (river-style) - each display has independent visible tags
- **Tag-based workspace management** (like dwm/river) - windows can have multiple tags (bitmask)
- **External layout engine** (like river) - separate process, stdin/stdout JSON, custom engines supported
- **Per-tag layout switching** - each tag can have different layout engine
- **River-style configuration** - shell script (`~/.config/yashiki/init`), CLI commands
- **Window rules** (riverctl-style) - glob patterns, actions: ignore, float, tags, output, position, dimensions
- **Cursor warp** - `disabled`, `on-output-change`, `on-focus-change`
- **Auto-raise** (focus follows mouse) - `disabled`, `enabled` with optional delay
- **Keybinding modes** (river-style) - named modes (normal, resize, passthrough, etc.) with per-mode bindings
- **State streaming** - real-time events via `/tmp/yashiki-events.sock`

## Layout Protocol

```rust
// yashiki → layout engine
enum LayoutMessage {
    Layout { width: u32, height: u32, windows: Vec<u32> },
    Command { cmd: String, args: Vec<String> },
}

// layout engine → yashiki
enum LayoutResult {
    Layout { windows: Vec<WindowGeometry> },  // id, x, y, width, height
    Ok,
    NeedsRetile,
    Error { message: String },
}
```

Focus notification: `focus-changed <window_id>` sent automatically on focus change.

## State Streaming

Events via `/tmp/yashiki-events.sock` (JSON lines). Client sends `SubscribeRequest` with optional snapshot and filter. Events: WindowCreated/Destroyed/Updated, WindowFocused, DisplayFocused/Added/Removed/Updated, TagsChanged, LayoutChanged, ModeChanged, Snapshot.

## CLI Usage

Tags use bitmask: tag 1 = 1, tag 2 = 2, tag 3 = 4, tags 1+2 = 3

```sh
yashiki start                     # Start daemon
yashiki declare-mode resize        # Declare a keybinding mode
yashiki enter-mode resize          # Switch to a mode
yashiki get-mode                   # Get current mode
yashiki bind alt-1 tag-view 1     # Bind hotkey (normal mode)
yashiki bind --mode resize h layout-cmd dec-main-ratio  # Bind in specific mode
yashiki unbind alt-1              # Unbind hotkey (normal mode)
yashiki unbind --mode resize h    # Unbind in specific mode
yashiki list-bindings             # List all bindings (all modes)
yashiki list-bindings --mode normal  # List bindings for specific mode
yashiki tag-view 1                # Switch to tag
yashiki tag-view --output 2 1     # Switch on specific display
yashiki tag-toggle 2              # Toggle tag visibility
yashiki tag-view-last             # Switch to previous tags
yashiki window-move-to-tag 1      # Move window to tag
yashiki window-toggle-tag 2       # Toggle tag on window
yashiki window-focus next|prev|left|right|up|down
yashiki window-swap next|prev|left|right|up|down
yashiki window-toggle-fullscreen
yashiki window-toggle-float
yashiki window-close
yashiki output-focus next|prev
yashiki output-send next|prev
yashiki retile [--output N]
yashiki layout-set-default tatami
yashiki layout-set [--tags N] [--output N] byobu
yashiki layout-get [--tags N] [--output N]
yashiki layout-cmd [--layout name] <cmd> [args]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [typester/yashiki](https://github.com/typester/yashiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
