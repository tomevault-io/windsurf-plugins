---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build / Run / Test

### Flutter

```bash
# Get dependencies
flutter pub get

# Run code generation (freezed, json_serializable)
flutter pub run build_runner build

# Watch mode — regenerate on file changes (useful during development)
flutter pub run build_runner watch

# Analyze
flutter analyze

# Run tests (only a placeholder widget test exists)
flutter test

# Build Android APK (release, arm64 only)
flutter build apk --release --target-platform android-arm64

# Build Windows release
flutter build windows --release
```

### WebView Frontend (`lib/webview/`)

The chat message rendering layer is a separate **Vite + Vue 3** project. It embeds inside Flutter via `flutter_inappwebview`.

```bash
cd lib/webview

# Install dependencies (first time only)
npm install

# Start dev server (http://localhost:5173) — HMR, no Flutter restart needed
npm run dev

# Build production assets
npm run build

# Preview production build
npm run preview
```

## Project Overview

SillyChat is a Flutter-based AI chat app inspired by NextChat and SillyTavern. Targets Android (primary) and Windows/Linux/macOS. Uses Material 3.

- Flutter 3.35.5, Dart SDK ^3.5.4
- Package name: `flutter_example` (legacy — do not rename)
- Version: 1.18.0

## Development Workflow

**Flutter-only changes**: Run `flutter run` on a connected device/emulator. Hot restart works as usual.

**WebView frontend changes**: The chat message UI is rendered by a Vue 3 app in `lib/webview/`. In debug mode, Flutter's `InAppWebView` loads `http://localhost:5173/`. Workflow:

1. Start the Vite dev server: `cd lib/webview && npm run dev`
2. Run the Flutter app: `flutter run`
3. Edit Vue/JS/CSS in `lib/webview/src/` — Vite HMR applies changes instantly within the WebView, **no Flutter restart needed**

**Full-stack changes**: Run both the Vite dev server and Flutter app simultaneously (two terminals).

**Production**: WebView assets must be built (`npm run build`) and placed in `assets/webview/` before a Flutter release build.

## Architecture

**State management**: GetX (`get: ^4.7.2`) with observable reactive variables (`.obs` / `RxList` / `RxMap`) and `GetBuilder` widgets.

**Persistence**: File-based JSON storage in a "vault" directory (`{appDocDir}/SillyChat/{vaultName}/`). Supports multiple vaults with WebDAV cloud sync. Each chat is a single `.chat` JSON file. No database — manual JSON serialization everywhere.

### Directory Layout

```
lib/
  main.dart                  # App entry point, controller init, theme setup
  chat-app/
    constants.dart           # App-wide constants
    events.dart              # Simple event classes (FileDeleted, FileCreated, etc.)
    themes.dart              # Theme data
    main_page.dart           # Desktop layout (sidebar + page view)
    mobile_main_page.dart    # Mobile layout (bottom nav)
    models/                  # Data models — manual toJson/fromJson
    providers/               # GetX controllers (BaseController pattern)
    pages/                   # Full-screen route pages
      character/             # Character CRUD, gallery, contact list
      chat/                  # Chat detail, message editing, search, file manager
      chat_options/          # Chat option presets
      common/                # Category management
      lorebooks/             # World book / lorebook editing
      other/                 # API management, prompts, onboarding
      regex/                 # Regex rule editing
      settings/              # Settings, import from SillyTavern, appearance
      story/                 # Story/group chat management
    widgets/                 # Reusable UI components
      chat/                  # Message bubbles, input area, think widget
      common/                # Shared form widgets (chips, switches, avatars)
      webview/               # WebView-based components (relation map, message rendering)
    utils/
      AIHandler.dart         # Dio HTTP client, SSE stream parsing, background task mgmt
      promptBuilder.dart     # Builds LLM message list: lorebook activation → prompt insertion → format
      promptFormatter.dart   # Macro/variable substitution in prompts
      LoreBookUtil.dart      # World book activation logic
      FileUtils.dart         # File system helpers
      init_app.dart          # First-run data initialization
      service_handlers/      # LLM provider adapters (see below)
      sillyTavern/           # SillyTavern import (characters, lorebooks, regex, config)
      entitys/               # LLMMessage, RequestOptions, ChatAIState
      markdown/              # Custom LaTeX markdown extensions
```

### Controller Initialization Pattern

All controllers extend `BaseController` (at `lib/chat-app/providers/base_controller.dart`), which uses a `Completer<void>` for async init. Controllers call `markReady()` after `onInit` completes. `SillyChatApp.waitAllReadyAndNotify()` awaits all controllers before showing the UI.

Controllers register with `Get.put()` in `SillyChatApp`'s constructor. Order matters — `SettingController` and `VaultSettingController` must be ready first since others depend on vault path info.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bilimited/sillyChat](https://github.com/bilimited/sillyChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
