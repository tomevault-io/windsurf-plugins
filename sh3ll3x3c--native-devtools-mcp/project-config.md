---
trigger: always_on
description: **About:** This is the AGENTS.md for **native-devtools-mcp**, an MCP (Model Context Protocol) server that enables **computer use** / **desktop automation** on macOS, Windows, and Android: screenshots, OCR, mouse/keyboard input, window management, and Android device control via ADB.
---

# Agent Context: native-devtools-mcp

**About:** This is the AGENTS.md for **native-devtools-mcp**, an MCP (Model Context Protocol) server that enables **computer use** / **desktop automation** on macOS, Windows, and Android: screenshots, OCR, mouse/keyboard input, window management, and Android device control via ADB.

**Search keywords:** MCP, Model Context Protocol, computer use, desktop automation, UI automation, RPA, screenshots, OCR, screen reading, mouse, keyboard, macOS, Windows, Android, ADB, mobile testing, native-devtools-mcp.

**Role:** You are an agent equipped with "Computer Use" capabilities. You can see the screen, type, move the mouse, and interact with native desktop and mobile applications.

**Constraint:** You are operating a real machine. Actions are permanent. Ensure you verify the state of the screen before and after actions.

## 🧠 Core Reasoning Loop

For robust automation, follow this "Visual Feedback Loop":

1.  **OBSERVE:** Call `take_screenshot(app_name="TargetApp")` to see the current state.
2.  **LOCATE:** Analyze the image or use the OCR summary text in the response to find coordinates.
3.  **ACT:** Call `click()`, `type_text()`, or `scroll()` using those coordinates.
4.  **VERIFY:** Call `take_screenshot` again to confirm the action had the intended effect.

**macOS-preferred branch (native apps):** substitute OBSERVE with `take_ax_snapshot(app_name='...')`; LOCATE reads uid + bbox from the emitted tree; ACT calls `ax_click(uid)` for pressable controls, `ax_set_value(uid, text)` for text fields, or `ax_select(uid)` for `NSOutlineView` / `NSTableView` row selection (sidebars, rule lists); VERIFY re-snapshots and reads the new state. This branch does not move the cursor or steal focus, so it composes with background work.

---

## 🗺️ Capabilities Matrix (Strategy Guide)

Use this table to choose the right tool sequence for the user's goal.

| User Goal | Tool Sequence | Why? |
|-----------|---------------|------|
| "Click the 'Submit' button" | `find_text(text="Submit")` → `click(x, y)` | Fastest. No visual analysis needed if text is known. |
| "Click the red icon" | `take_screenshot()` → (Analyze Image) → `click(screenshot_x=..., screenshot_y=..., screenshot_origin_x=..., screenshot_origin_y=..., screenshot_scale=...)` | Visual features require full screenshot analysis. |
| "What element is at (500, 300)?" | `element_at_point(x=500, y=300)` | Returns the accessibility element at those coordinates (name, role, bounds, etc.). |
| "Type into the search bar" | `find_text(text="Search")` → `click(x, y)` → `type_text("hello")` | Must click to focus before typing. |
| "Scroll down" | `scroll(x=500, y=500, delta_y=200)` | Positive `delta_y` scrolls down. |
| "Find an open window" | `list_windows()` → `focus_window(window_id=...)` | Don't guess window names; list them first. |
| "Track what I hover over" | `start_hover_tracking(min_dwell_ms=300)` → user moves mouse → `stop_hover_tracking()` | Records element transitions with dwell filtering. |
| "Record what the user does" | `start_recording(output_dir="/tmp/rec")` → user interacts → `stop_recording()` | Captures frontmost app at ~5fps as JPEG frames. |
| "Launch Safari with debug port" | `launch_app(app_name="Safari", args=["--remote-debugging-port=9222"])` | Pass CLI args on fresh launch. |
| "Quit an app" | `quit_app(app_name="Safari")` | Graceful by default; use `force=true` to kill immediately. |
| "Click a named button in a native app (macOS)" | `take_ax_snapshot` → `ax_click(uid)` | Focus-preserving. Generation-tagged uids; fresh snapshot invalidates prior uids. |
| "Enter text into a text field via value assignment (macOS)" | `take_ax_snapshot` → `ax_set_value(uid, text)` | No key events, no IME, no undo-stack entry. Fall back to `click` + `type_text` on `not_dispatchable`. |
| "Select a sidebar row in System Settings / a `NSOutlineView` row (macOS)" | `take_ax_snapshot` → `ax_select(uid)` | Writes `AXSelectedRows` on the enclosing outline/table. Use this instead of `ax_click` for row targets — rows typically refuse `AXPress`. |
| "AX snapshot invalidation rule (macOS)" | — | Every `take_ax_snapshot` call bumps the generation. All prior uids become stale; `ax_*` tools return `snapshot_expired`. |
| "Click a button in Chrome" | `cdp_connect(port=9222)` → `cdp_take_ax_snapshot()` → `cdp_click(uid="a42")` | CDP is more reliable than coordinates for web content. |
| "Type in a web input" | `cdp_take_ax_snapshot()` → `cdp_fill(uid="a42", value="hello")` | Works for `<input>`, `<textarea>`, and `<select>` elements. |
| "Run JS in a browser page" | `cdp_evaluate_script(function="() => document.title")` | Evaluate any JS in the selected page. |
| "Navigate to a URL" | `cdp_navigate(url="https://example.com")` | Also supports back, forward, reload. |
| "Press Enter or shortcut" | `cdp_press_key(key="Enter")` or `cdp_press_key(key="Control+A")` | Supports modifier combos. |
| "Wait for page content" | `cdp_wait_for(text="Success")` | Polls snapshot until text appears or timeout. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sh3ll3x3c/native-devtools-mcp](https://github.com/sh3ll3x3c/native-devtools-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
