---
trigger: always_on
description: Apply Python best practices and clean code principles. Only change code relevant to the prompt.
---

# Project Guidelines

Apply Python best practices and clean code principles. Only change code relevant to the prompt.
Prioritize readability and auditability - users handle credentials and must be able to verify the code is safe at a glance.

## Platform
- Windows-only application - no `sys.platform` checks or cross-platform guards needed
- Windows APIs (`ctypes.windll`, `winreg`) can be used unconditionally

## Popup Window & DPI
- The popup uses pywebview with a WinForms host window and Edge WebView2
- pywebview 6.x `resize()` **and** `move()` both expect **logical pixels** (pywebview applies DPI scaling internally for both)
- `_tray_position()` still receives physical pixel dimensions (needed to calculate position against Win32 physical coordinates) and returns **logical coordinates** for `move()` - never change this to physical
- `_tray_position()` uses `Shell_TrayWnd` + `MonitorFromWindow` + `GetMonitorInfoW` to find the monitor that owns the taskbar, then compares `work.left > mon.left` (not `> 0`) to detect a left-side taskbar - this correctly handles multi-monitor layouts where the primary monitor is not at virtual x=0
- Never replace `resize()`/`move()` with direct `SetWindowPos` calls for tray-anchored positioning - pywebview's internal scaling means raw Win32 calls would fight with pywebview's coordinate handling. The one exception is the pinned-popup drag (next bullet)
- The pinned-popup drag (`_begin_drag`/`_drag`/`_end_drag`) deliberately uses raw `SetWindowPos` with **physical** cursor coordinates (`GetCursorPos` minus the grab offset captured on mouse-down), not pywebview's `move()`. Reason: `move()` and JS `screenX` deltas are scaled by a single monitor's DPI, which jumps at a monitor boundary and makes the cursor drift off the window and the size break. After a drag that crosses a DPI boundary, `_end_drag` re-asserts the size once via `resize()` against the destination monitor's DPI. Do not collapse this back to `move()` - it reintroduces the mixed-DPI drift
- The taskbar icon is hidden via Win32 extended styles (`WS_EX_TOOLWINDOW` + remove `WS_EX_APPWINDOW`). Do **not** use WinForms `ShowInTaskbar = False` - it recreates the native window handle, which crashes WebView2 from background threads

## Tray Icon Interaction
- pystray has no native double-click support (it fires the default menu item on every `WM_LBUTTONUP`). Double-click is added only when `on_double_click_command` is set: `_install_double_click_handler()` swaps the `WM_NOTIFY` entry in pystray's private `_message_handlers` table (matched by identity against `icon._on_notify`) for `_on_tray_message`. This reaches into pystray internals - if a pystray upgrade renames `_message_handlers`/`_on_notify`, this is where it breaks
- With a command configured, the single click (popup) is deferred by `GetDoubleClickTime()` via a `threading.Timer` and cancelled when the second click arrives; the trailing `WM_LBUTTONUP` that always follows a `WM_LBUTTONDBLCLK` is swallowed via `_swallow_next_up`. All tray-message state is guarded by `_click_lock`, and `_fire_single_click()` re-checks the timer under the lock so a double-click landing exactly as the timer fires still suppresses the popup
- When no `on_double_click_command` is set, the handler is **not** installed - pystray's instant single-click popup must stay untouched (no double-click delay). Do not make the deferral unconditional
- `WM_NOTIFY` and other message handlers (right-click menu) must still fall through to the saved `_pystray_on_notify`

## Event Commands
- Event commands run fire-and-forget with output discarded (`run_event_command` in `command.py`). User-driven actions - the "Test event commands" menu handlers and `on_double_click_command` - pass `capture_output=True`, which captures stdout/stderr, prints them, and raises an error message box when the command exits non-zero, so a wrong path is not swallowed silently. Automatic events (`on_reset_command`, `on_threshold_command`, `on_startup_command`) must stay silent (no `capture_output`) - a background event must never pop a dialog. A new event command belongs on whichever side matches: user-driven surfaces failures, automatic stays silent
- `capture_output` waits for the command on a daemon thread, so the caller (a tray/menu/poll thread) is never blocked, even when the command launches a long-running app

## Claude CLI
- The `cli_command` setting (name -> base command, e.g. a WSL install) is **display only**: `find_installations()` lists each entry *in addition to* the auto-detected native CLI and the IDE extensions, all of which stay exactly as they were. It must never gain a second job - not the token refresh, not the API User-Agent, not authentication of any kind

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jens-duttke/usage-monitor-for-claude](https://github.com/jens-duttke/usage-monitor-for-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
