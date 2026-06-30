---
trigger: always_on
description: `flutter_window_close` is a Flutter desktop plugin that lets apps intercept and handle window close events. When a user clicks the close button, the plugin redirects the event to a Dart handler so the app can show a confirmation dialog before allowing the window to close. It supports **macOS**, **Windows**, **Linux**, and **Flutter Web**.
---

# GitHub Copilot Instructions for flutter_window_close

## Project Overview

`flutter_window_close` is a Flutter desktop plugin that lets apps intercept and handle window close events. When a user clicks the close button, the plugin redirects the event to a Dart handler so the app can show a confirmation dialog before allowing the window to close. It supports **macOS**, **Windows**, **Linux**, and **Flutter Web**.

Current version: **1.3.0**  
Minimum SDK: Dart `>=3.4.0 <4.0.0`, Flutter `>=3.22.0`

---

## Repository Structure

```
flutter_window_close/
├── .github/
│   ├── copilot-instructions.md   # This file
│   ├── dependabot.yml            # Weekly pub dependency updates
│   └── workflows/
│       ├── ci.yaml               # Multi-OS × multi-Flutter-version CI
│       └── ci_spm.yaml           # Swift Package Manager integration test
├── lib/
│   ├── flutter_window_close.dart       # Public Dart API (desktop + guard for web)
│   └── flutter_window_close_web.dart   # Web implementation via JS interop
├── macos/flutter_window_close/
│   └── Sources/flutter_window_close/
│       └── FlutterWindowClosePlugin.swift  # macOS – NSWindowDelegate
├── windows/
│   └── flutter_window_close_plugin.cpp     # Windows – WM_CLOSE via WindowProc
├── linux/
│   └── flutter_window_close_plugin.cc      # Linux – GTK delete-event signal
├── test/
│   └── flutter_window_close_test.dart      # Unit test scaffold
├── example/
│   └── lib/main.dart             # Full working example (desktop + web)
├── pubspec.yaml
└── analysis_options.yaml         # Uses flutter_lints
```

---

## Architecture

### Method Channels

The plugin uses **two** named Flutter method channels:

| Channel name | Direction | Purpose |
|---|---|---|
| `flutter_window_close` | Dart → Native | `init`, `closeWindow`, `destroyWindow`, `setWebReturnValue` |
| `flutter_window_close_notification` | Native → Dart | `onWindowClose` (event notification) |

### Lifecycle Flow

1. **Dart** calls `setWindowShouldCloseHandler(handler)`.
2. Dart calls `_channel.invokeMethod('init')` on the `flutter_window_close` channel.
3. Native side starts listening for OS-level close events.
4. When a close event fires, native invokes `onWindowClose` on the notification channel.
5. Dart receives the notification, calls the user-supplied `handler()`.
6. If the handler returns `true`, Dart calls `_channel.invokeMethod('destroyWindow')` to close the window; otherwise the close is cancelled.

### Platform-Specific Bridges

| Platform | API used | Key detail |
|---|---|---|
| **macOS** | `NSWindowDelegate.windowShouldClose(_:)` | Listens to the first window in `NSApplication.windows`; single-window apps only |
| **Windows** | `WM_CLOSE` via `RegisterTopLevelWindowProcDelegate` | Uses `GetAncestor(hwnd, GA_ROOT)` to find root window |
| **Linux** | GTK `delete-event` signal | Disconnects Flutter's own handler first to avoid conflicts (see flutter/engine#40033) |
| **Web** | `window.beforeunload` event | Sets `returnValue` on the `BeforeUnloadEvent`; uses `dart:js_interop` |

---

## Dart Public API

```dart
// Assign a handler – returns true to allow close, false to cancel.
// Throws if called on Flutter Web.
static Future<void> setWindowShouldCloseHandler(
    Future<bool> Function()? handler)

// Triggers a closeable window-close (goes through the handler).
// Throws if called on Flutter Web.
static Future<void> closeWindow()

// Closes the window immediately without confirmation.
// Throws if called on Flutter Web.
static Future<void> destroyWindow()

// Web only – sets the browser's beforeunload return value.
// Throws if called outside Flutter Web.
static Future<void> setWebReturnValue(String? returnValue)
```

All desktop methods guard against web usage with `if (kIsWeb) throw Exception(...)`. Conversely, `setWebReturnValue` throws on non-web platforms.

---

## Code Style & Conventions

- **Dart:** Follow `flutter_lints` rules (`analysis_options.yaml`). Use `const` constructors wherever possible; prefer named constructors for widgets.
- **Swift (macOS):** Standard Swift style; `//MARK: -` comment separators for protocol sections.
- **C++ (Windows/Linux):** Google-style C++; `snake_case` for variables and methods; braces on same line; use `std::optional<LRESULT>` for the Windows proc delegate return type.
- **Naming:** Plugin class is `FlutterWindowClosePlugin` on all platforms; channel names are string literals, not constants, and must match exactly across Dart and native.
- **No mobile support:** Do not add iOS or Android platform entries. The plugin intentionally targets desktop and web only.

---

## Testing

Run all checks from the repository root:

```bash
# Analyze Dart code
flutter analyze

# Run unit tests
flutter test

# Build the example app for the current platform
cd example
flutter build macos    # or windows / linux
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zonble/flutter_window_close](https://github.com/zonble/flutter_window_close) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
