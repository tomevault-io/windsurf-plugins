---
trigger: always_on
description: An MCP server that lets AI agents interact with Windows desktop applications using the same patterns as Playwright's browser automation - structured accessibility snapshots with element refs, not screenshot-based guessing.
---

# Windows App Automation MCP Server

An MCP server that lets AI agents interact with Windows desktop applications using the same patterns as Playwright's browser automation - structured accessibility snapshots with element refs, not screenshot-based guessing.

## Why MCP Server (not CLI or library)

| Approach | Agent Experience |
|----------|-----------------|
| **MCP Server** | `windows_snapshot` → structured tree with refs → `windows_click ref="btn7"` ✅ |
| **CLI exe** | Run command → parse text output → figure out element IDs → run another command |
| **Screenshot-only** | Take screenshot → vision model guesses coordinates → hope click lands |

The Playwright MCP pattern works extremely well for agents:
- `browser_snapshot` returns an accessibility tree with refs like `[ref=s1e5]`  
- Agent picks element by semantic meaning, not coordinates
- `browser_click ref="s1e5"` is precise and reliable

This project brings that same pattern to Windows desktop apps.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  AI Agent (GitHub Copilot, Claude, etc.)                        │
│  - Calls MCP tools: windows_snapshot, windows_click, etc.      │
│  - Sees structured element tree, picks refs by meaning          │
└─────────────────────────────────────────────────────────────────┘
                              │ MCP Protocol (JSON-RPC over stdio)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  Windows MCP Server (.NET / FlaUI)                              │
│  - Implements MCP tool handlers                                 │
│  - Translates UI Automation tree → agent-friendly snapshot      │
│  - Manages element refs ↔ AutomationElement mapping             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  FlaUI (.NET) - github.com/FlaUI/FlaUI                         │
│  - UIA3Automation (WPF, UWP, Store Apps, modern Win32)         │
│  - UIA2Automation (fallback for older WinForms)                │
│  - Handles platform quirks, control patterns, tree walking      │
└─────────────────────────────────────────────────────────────────┘
```

## MCP Tools

### `windows_launch`
Launch a Windows application and return a session ID.

```json
{ "app": "calc.exe" }
{ "app": "Microsoft.WindowsCalculator_8wekyb3d8bbwe!App" }  // UWP
{ "app": "C:\\Program Files\\MyApp\\app.exe", "args": ["--debug"] }
```

### `windows_snapshot`
**The key tool for agents.** Returns accessibility tree of the active window with refs.

```
- window "Calculator" [ref=w1]
  - group "Number pad" [ref=w1e1]
    - button "Seven" [ref=w1e2] 
    - button "Eight" [ref=w1e3]
    - button "Nine" [ref=w1e4]
  - group "Display" [ref=w1e10]
    - text "0" [ref=w1e11]
  - button "Equals" [ref=w1e20]
```

Like Playwright's `browser_snapshot`, this gives structured semantic data, not pixels.

### `windows_click`
Click an element by ref.
```json
{ "ref": "w1e2" }  // Click the "Seven" button
{ "ref": "w1e2", "button": "right" }  // Right-click
{ "ref": "w1e2", "doubleClick": true }
```

### `windows_type`
Type text into a focused or specified element.
```json
{ "ref": "w1e5", "text": "Hello world" }
{ "text": "Hello", "submit": true }  // Type and press Enter
```

### `windows_fill`
Clear and fill a text field (like Playwright's fill vs type).
```json
{ "ref": "w1e5", "value": "new text" }
```

### `windows_screenshot`
Take a screenshot (for visual verification when needed).
```json
{ }  // Screenshot active window
{ "ref": "w1e10" }  // Screenshot specific element
{ "fullScreen": true }  // Entire screen
```

### `windows_get_text`
Get text content of an element.
```json
{ "ref": "w1e11" }  // Returns "0" from the display
```

### `windows_list_windows`
List all open windows with their titles and process info.
```json
[
  { "handle": "w1", "title": "Calculator", "process": "calc.exe" },
  { "handle": "w2", "title": "Untitled - Notepad", "process": "notepad.exe" }
]
```

### `windows_focus`
Bring a window to foreground.
```json
{ "handle": "w2" }
{ "title": "Notepad" }
{ "process": "notepad.exe" }
```

### `windows_close`
Close a window or application.
```json
{ "handle": "w1" }
```

## Snapshot Format Design

The snapshot format is critical for agent usability. Key principles:

1. **Semantic over structural** - Show "button" not "ControlType.Button"
2. **Concise refs** - Short refs like `w1e5` not UUIDs
3. **Relevant properties only** - Name, role, state. Skip internal IDs.
4. **Indentation shows hierarchy** - Visual tree structure
5. **Actionable hints** - Mark disabled/readonly elements

```
- window "Calculator" [ref=w1]
  - menubar [ref=w1e1]
    - menuitem "View" [ref=w1e2]
  - group "Number pad" [ref=w1e3]
    - button "7" [ref=w1e4]
    - button "8" [ref=w1e5]
    - button "9" [ref=w1e6]
    - button "÷" [ref=w1e7]
  - text "Display is 0" [ref=w1e8] [readonly]
  - button "Equals" [ref=w1e9] [disabled]
```

## Development Commands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shanselman/FlaUI-MCP](https://github.com/shanselman/FlaUI-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
