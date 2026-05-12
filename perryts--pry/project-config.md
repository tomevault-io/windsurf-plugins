---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pry is a native JSON viewer built with the Perry framework. Perry compiles TypeScript to native binaries — there is no Electron, no browser, no Node.js runtime. The app uses Perry's UI bindings which map directly to platform-native widgets (AppKit on macOS, GTK4 on Linux, Win32 on Windows, UIKit on iOS, Android Views on Android).

## Build Commands

```bash
# Compile the main app (perry repo is expected to be a sibling directory)
# macOS
cd /path/to/perry && cargo run --release -- compile /path/to/perry-pry/src/main.ts -o /path/to/perry-pry/pry
# Linux (requires gtk4-devel)
cd /path/to/perry && cargo run --release -- compile /path/to/perry-pry/src/main_linux.ts -o /path/to/perry-pry/pry

# Compile test harnesses
cd /path/to/perry && cargo run --release -- compile /path/to/perry-pry/src/test_tree.ts -o /path/to/perry-pry/test_tree
cd /path/to/perry && cargo run --release -- compile /path/to/perry-pry/src/minimal.ts -o /path/to/perry-pry/minimal

# Run the compiled binary
/path/to/perry-pry/pry
```

There is no test suite, linter, or package manager. The project is compiled entirely by the Perry compiler (a Rust program in the sibling `perry` repo).

## Architecture

The app is a single-window JSON tree viewer. Platform-specific entry points set up the UI and call `App()`. The remaining `src/` files are shared utility modules:

- **main.ts** — macOS entry point (AppKit). Sets up the scroll view, tree container, keyboard shortcuts, and calls `App()` (which blocks forever).
- **main_linux.ts** — Linux entry point (GTK4). Same structure as macOS but uses Ctrl instead of Cmd.
- **main_windows.ts** — Windows entry point (Win32). Uses Ctrl shortcuts.
- **tree_node.ts** — `buildNode()` renders a single tree row (indent + toggle button + key + value + context menu). `buildClosingBracket()` renders `]` or `}`.
- **tree_builder.ts** — `buildTree()` recursively walks a JSON value and calls `buildNode` for each node. `collectAllPaths()` gathers all container paths for expand-all.
- **json_parser.ts** — `parseJsonInput()` wraps `JSON.parse` with timing, node counting, and error handling. Returns a `ParseResult`.
- **search.ts** — `findMatches()` recursively searches keys and values (case-insensitive) and returns matching path strings.
- **search_bar.ts** — `buildSearchBar()` creates the search UI row (text field + match counter + prev/next buttons).
- **status_bar.ts** — `buildStatusBar()` shows node count, byte size, and parse time.
- **error_view.ts** — `buildErrorView()` displays parse errors with a raw text preview.
- **colors.ts** — RGBA color constants for JSON types (string=green, number=blue, boolean=orange, null=gray).
- **format.ts** — Number/byte/time formatting and string truncation utilities.
- **path.ts** — `jsonPath()` builds JSONPath strings like `$.data.users[0].name` from key arrays.
- **clipboard.ts** — Thin wrappers around `perry/ui` clipboard functions.

Test harnesses (`test_tree.ts`, `test_scroll.ts`, `minimal.ts`) are standalone apps for testing specific features in isolation.

## Perry Framework Constraints

These are critical — violating them causes compiler or runtime errors:

- **`App()` must be the last statement.** It blocks the thread. Code after it never runs. `addKeyboardShortcut`, `appSetMinSize`, etc. must all be called before `App()`.
- **All imports must be static.** No `require()` — Perry is statically compiled.
- **No Set/Map types.** Use arrays with manual `indexOf`/linear scan instead.
- **Widget handles are opaque values** (NaN-boxed f64 or i64 pointers). Treat them as `any`.
- **Don't use `fs.readFileSync` inside `openFileDialog` callbacks** — causes i64/f64 type mismatch. Extract file reading to a separate function.
- **Colors are RGBA floats** in 0.0–1.0 range.
- **`textSetFontWeight(handle, fontSize, weight)`** — requires the fontSize parameter; weight 1.0 = bold.
- **`VStackWithInsets(spacing, top, left, bottom, right)`** — returns a handle; add children via `widgetAddChild`.
- **Keyboard modifier flags:** 1=Cmd, 2=Shift, 4=Option, 8=Control (bitwise OR to combine).

## UI Pattern

The app uses an imperative widget tree pattern (not declarative/reactive):
1. Create widget handles (`Text()`, `HStack()`, `Button()`, etc.)
2. Configure them (`textSetFontSize()`, `textSetColor()`, etc.)
3. Assemble via `widgetAddChild(parent, child)`
4. On state change, call `widgetClearChildren()` and rebuild the subtree

The tree view rebuilds by clearing all children from the container VStack and re-adding them from the parsed JSON data. Expand/collapse state is tracked as a `string[]` of expanded path strings (e.g., `["$", "$.config", "$.data[0]"]`).

# UI Testing with Geisterhand

This project uses [Geisterhand](https://github.com/Geisterhand-io/macos) for UI automation and testing.
All requests/responses use `snake_case` JSON.

**Required macOS permissions** (grant in System Settings > Privacy & Security):
- Accessibility (keyboard/mouse control)
- Screen Recording (screenshots)

## Starting the App Under Test


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PerryTS/pry](https://github.com/PerryTS/pry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
