---
trigger: always_on
description: Notion/Ion3-style static tiling window manager for the River Wayland compositor (0.4.x+).
---

# notion-river

Notion/Ion3-style static tiling window manager for the River Wayland compositor (0.4.x+).

## Project Overview

A window manager process for River 0.4.x that implements "static tiling" from the Notion WM: the screen layout is a persistent wireframe of frames that exist independently of windows. Windows are placed into frames as tabs. Opening/closing windows never changes the layout — only explicit user actions (split/unsplit) do.

## Build / Test / Run

```sh
cargo build            # debug build
cargo build --release  # release build
cargo test             # run unit tests (layout + focus tests)
cp target/release/notion-river ~/.local/bin/
```

After installing, press `Super+Shift+R` inside River to restart the WM with the new binary. Windows survive restarts.

### Native (from TTY / login manager)

River is built from source at `~/repos/river` with XWayland support:

```sh
cd ~/repos/river && zig build -Doptimize=ReleaseSafe -Dxwayland=true
cp zig-out/bin/river ~/.local/bin/
```

lightdm is configured with a "Notion River" session (`/usr/share/wayland-sessions/river-custom.desktop`) pointing to `~/.local/bin/start-river`.

The `start-river` script sets XKB layout (de/neo), Wayland env vars, and execs River.

The init script (`~/.config/river/init`) starts waybar, nm-applet, keepassxc, and runs notion-river in a restart loop (always restarts, not just on exit 0). notion-river itself owns monitor configuration (mode/scale/position/transform) via `wlr-output-management-unstable-v1`; no kanshi or external tool is involved. wp_viewporter protocol handles fractional scaling.

### Nested testing (inside X11)

```sh
weston --backend=x11-backend.so --width=1920 --height=1080 --shell=kiosk-shell.so &
WAYLAND_DISPLAY=wayland-1 XKB_DEFAULT_LAYOUT=de XKB_DEFAULT_VARIANT=neo \
  river -c ~/Projects/notion-river/target/release/notion-river -no-xwayland &
WAYLAND_DISPLAY=wayland-2 foot &
```

## Architecture

- `src/main.rs` — entry point, Wayland connection, event loop, signal handler, log file setup
- `src/protocol.rs` — wayland-scanner generated bindings (river-window-management-v1, river-xkb-bindings-v1, river-layer-shell-v1)
- `src/dispatch.rs` — Wayland `Dispatch` impls for all protocol interfaces (WM, output, seat, window, pointer, layer-shell, decorations)
- `src/wm.rs` — core WM state, manage/render cycle, focus logic integration
- `src/window_actions.rs` — action execution: perform_action, perform_split, perform_unsplit, cross-monitor moves, command spawning
- `src/rendering.rs` — layout application: window dimensions, focus, visibility, position/border/decoration drawing
- `src/pointer_ops.rs` — pointer operation handling: move-drop, seat ops (resize), resize axis detection, cursor warping
- `src/layout.rs` — static split tree (binary tree of frames), geometry calculation, neighbor finding, ratio adjustment
- `src/decorations.rs` — tab bar rendering (per-window decoration surfaces via Cairo+Pango) + empty frame indicators (shell surfaces)
- `src/control.rs` — IPC control socket server: accepts commands on `$XDG_RUNTIME_DIR/notion-river.sock`
- `src/bin/notion-ctl.rs` — CLI client for the control socket
- `src/workspace.rs` — workspace manager, deterministic 3-tier output assignment (monitor memory → preferred_output → fallback), multi-monitor
- `src/bindings.rs` — keybinding parsing, built-in profiles (i3_neo, notion), media keys, modifier constants
- `src/actions.rs` — action enum and config string parsing
- `src/config.rs` — TOML config loading and defaults
- `src/focus.rs` — focus-follows-mouse logic, extracted for testability with 12 unit tests
- `src/state.rs` — state persistence: save/restore layout tree, window placement, active tabs, visible workspaces to `~/.config/notion-river/`
- `src/app_bindings.rs` — app-to-frame bindings: bind/unbind apps to frames, wildcard app_id matching, fixed dimensions, persistence to `~/.config/notion-river/bindings.json`, enforce_app_bindings auto-move
- `src/monitor_memory.rs` — per-monitor "last workspace shown here" memory, keyed by EDID description (with connector/geometry fallbacks), persisted to `~/.config/notion-river/monitor-memory.json`
- `src/monitors.rs` — monitor (output) layout via `wlr-output-management-unstable-v1`: bind manager, observe heads/modes, apply saved profiles on topology change, persist user edits to `~/.config/notion-river/monitors.json` (keyed by sorted EDID set)
- `src/ipc.rs` — waybar workspace status: writes JSON to `$XDG_RUNTIME_DIR/notion-river-workspaces`, streams updates to IPC subscribers
- `protocol/` — River protocol XML files (vendored)

## Key Concepts

- **SplitNode**: Binary tree. Leaves are `Frame`s, interior nodes are `Split`s with orientation + ratio.
- **Frame**: A cell that holds 0+ windows as tabs. Empty frames are valid and render as bordered outlines.
- **Workspace**: Owns a SplitNode tree, assigned to an Output by preferred output name.
- **Physical keys**: `set_layout_override(0)` on xkb bindings for layout-independent keybindings.
- **Two-phase commit**: River's manage/render sequence. manage_start → WM decisions → manage_finish → render_start → positioning → render_finish.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Marenz/notion-river](https://github.com/Marenz/notion-river) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
