---
trigger: always_on
description: Welcome to the **termui** codebase. This document serves as a comprehensive developer and AI agent handbook detailing the product vision, core architectures, package layouts, element tree lifecycles, and testing practices.
---

# Developer & AI Agent Handbook: CLI Element-based Windowing System

Welcome to the **termui** codebase. This document serves as a comprehensive developer and AI agent handbook detailing the product vision, core architectures, package layouts, element tree lifecycles, and testing practices.

---

## 1. Product Overview & System Map

`termui` is a high-performance, double-buffered **Terminal User Interface (TUI)** and **Windowing System** written in Dart. It moves away from naive CLI output printing (which causes terminal flicker and high overhead) and provides a desktop-like environment inside standard ANSI/TTY terminal applications.

### Core Goals
* **Overlapping Window Management**: Supports floating, draggable, and resizeable frames with custom titles, borders, and Z-index layering.
* **Double Buffering**: Prevents terminal flickering by maintaining an in-memory frame buffer of what is visible on-screen and comparing it with a previous frame to compute delta updates.
* **Minimal ANSI Diffing**: Emits the shortest possible terminal sequences (cursor jumps and style transitions) to repaint only the modified cells.
* **Element & Widget Tree**: Re-implements a Flutter-like reactive layout system where widgets describe configurations, elements manage tree lifecycles, and states hold stateful properties.
* **Hierarchical Input & Focus System**: Translates raw ANSI byte streams from `stdin` into high-level event objects (keys, mouse clicks/scrolls/drags, paste segments) and dispatches them down a keyboard focus node tree.
* **Modular Widget Toolkit**: Standard widgets including paragraph wrappers, lists, interactive input fields with visual cursors, progress bars, a braille-based grid canvas, tile maps, and menu overlays.

---

## 2. Monorepo Package Architecture

The project is managed as a Melos monorepo with the following workspace packages:

1. **[termui](/packages/termui)** (Core package): Implements the layout engine, widgets, elements, focus management, and ANSI rendering pipeline.
2. **[termui_flutter](/packages/termui_flutter)** (Core package): Performant Flutter renderer.
3. **[termui_test](/packages/termui_test)** (Core package): Fakes, Mocks, Integration testing, Matchers, Goldent testing.
4. **[termui_shared_examples](/packages/termui_shared_examples)**: Contains example interfaces and reusable scenario layouts.
5. **[termui_recorder](/packages/termui_recorder)**: Asciicast v3 recording and playback, screenshots.

### Dependencies & Platforms
* **Dart SDK**: Target environment is `sdk: ">=3.11.0 <4.0.0"`.
* **Platform APIs**: Interacts with libc APIs on Linux/Unix using `dart:ffi` and Windows APIs using [win32](/pubspec.yaml#L17) (`^6.0.0`) to configure TTY raw mode and query screen sizing.
* **Unicode / Wide Characters**: Uses [characters](/pubspec.yaml#L12) (`^1.4.0`) to correctly measure and slice grapheme clusters (correctly handling multi-byte characters, ZWJ sequences, and emojis).

---

## 3. Development Guidelines & Rules

When writing or modifying code in this repository:

1. **Use Modern Dart Collections**: Leverage Dart collection language features such as `if` elements, `for` elements, spread operators (`...`), and null-aware collection entries. See the [Dart Collections Guide](https://dart.dev/language/collections) for syntax references.
2. **Multi-byte & Emoji Safety**: Never use `String.length` or `String.substring` for coordinates or drawing offsets. Always use `text.characters` from `package:characters` to handle grapheme cluster boundaries.
3. **Always Check if Mounted**: Focus changes can fire when widgets are being unmounted or cleaned up. Always guard `setState` calls in focus listeners with an `if (mounted)` check:
   ```dart
   focusNode.onFocusChange = (hasFocus) {
     if (hasFocus && mounted) {
       setState(() {
         // Update state properties
       });
     }
   };
   ```
4. **Propagate InheritedWidget Updates**: Ensure [InheritedElement.update](/packages/termui/lib/ui/layout.dart#L734) always calls `rebuild()` to ensure that children configurations get updated downstream when the parent rebuilds.
5. **Conventional Commits**: Write structured, clear commit messages conforming to the Conventional Commits 1.0.0 specification (e.g. `feat(core): ...`, `fix(focus): ...`).
6. **Avoid stdout Terminal Properties**: Never use `stdout.terminalColumns` or `stdout.hasTerminal` to determine screen dimensions or check for a terminal. Tests run in headless environments where these getters throw exceptions. Always use the `terminal` instance provided by the mock environment (e.g., `MockTerminalBackend` from `termui_test`) or `globalSceneManager.terminal.size`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jtmcdole/termui](https://github.com/jtmcdole/termui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
