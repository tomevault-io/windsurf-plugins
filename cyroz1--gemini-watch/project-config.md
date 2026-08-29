---
trigger: always_on
description: This file gives coding agents the project-specific context needed to work on Gemini Watch safely and consistently. Read it before making changes.
---

# AGENTS.md

## Purpose

This file gives coding agents the project-specific context needed to work on Gemini Watch safely and consistently. Read it before making changes.

Gemini Watch is a standalone watchOS SwiftUI app that talks directly to the Google Gemini API. It has no iPhone companion app, no server, no third-party package manager, and no analytics SDK. Keep changes small, native, and watch-first.

## Repository Layout

```text
.
├── README.md
├── LICENSE
├── .gitignore
└── gemini-watch/
    ├── gemini-watch.xcodeproj/
    └── gemini-watch Watch App/
        ├── AppSettingsStore.swift
        ├── Branding.swift
        ├── ChatViewModel.swift
        ├── ContentView.swift
        ├── ConversationListView.swift
        ├── GeminiService.swift
        ├── MarkdownParser.swift
        ├── MessageView.swift
        ├── Models.swift
        ├── PersistenceManager.swift
        ├── Secrets.plist.example
        ├── SettingsView.swift
        ├── Speaker.swift
        ├── gemini_watchApp.swift
        └── Assets.xcassets/
```

Important project characteristics:

- Native SwiftUI watchOS app.
- watchOS deployment target is 11.0.
- Swift version is 5.0 in project settings.
- The Watch App target uses `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`.
- The project uses Xcode's file system synchronized root group for the watch app source folder. Adding a Swift file under `gemini-watch/gemini-watch Watch App/` should normally be picked up by Xcode without manually editing the project file.
- There are currently no Swift Package Manager, CocoaPods, Carthage, or npm dependencies.

## Build And Run

Primary workflow:

1. Open `gemini-watch/gemini-watch.xcodeproj` in Xcode 16 or later.
2. Select the `gemini-watch Watch App` scheme.
3. Choose an Apple Watch simulator or paired watch.
4. Build and run with Cmd-R.

CLI workflow, when full Xcode is selected:

```bash
xcodebuild -list -project gemini-watch/gemini-watch.xcodeproj
xcodebuild \
  -project gemini-watch/gemini-watch.xcodeproj \
  -scheme "gemini-watch Watch App" \
  -destination 'platform=watchOS Simulator,name=Apple Watch Series 10 (46mm)' \
  build
```

If `xcodebuild` reports that the active developer directory is Command Line Tools, the machine needs full Xcode selected, for example:

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

Do not add a package manager or external build system just to make local validation easier.

## Secrets And API Keys

The Gemini API key is loaded from `Secrets.plist` in the app bundle.

- `Secrets.plist` must not be committed.
- `Secrets.plist.example` is the committed template and should remain safe for public repos.
- `.gitignore` already excludes `Secrets.plist`.
- Do not add fallback keys, sample real keys, logging of keys, or hard-coded API credentials.
- If API-key handling changes, preserve a clear missing-key error path for users.

The expected plist key is:

```text
GEMINI_API_KEY
```

## Architecture

The app follows a lean MVVM-style structure:

```text
ConversationListView
  -> ContentView
      -> ChatViewModel
          -> GeminiService
          -> PersistenceManager
          -> AppSettingsStore
      -> MessageView
          -> MarkdownParser
          -> Speaker
```

Core responsibilities:

- `gemini_watchApp.swift`: app entry point, creates shared `AppSettingsStore` and `Speaker`, injects them through the SwiftUI environment, and requests notification authorization.
- `ConversationListView.swift`: root navigation, conversation list, search, pin/unpin, delete, settings sheet, and new-chat creation.
- `ContentView.swift`: main chat UI, message scrolling, input bar, quick-reply chips, edit flow, stop/regenerate actions, and error display.
- `ChatViewModel.swift`: conversation state, user/model messages, streaming orchestration, cancellation, haptics, persistence calls, suggestions, and local reply notifications.
- `GeminiService.swift`: Google Gemini API client, streaming SSE parsing, model listing, system prompt and temperature plumbing, web-search grounding, and API error mapping.
- `Models.swift`: `Message`, `Conversation`, `ConversationMetadata`, `GroundingSource`, and `AppSettings`.
- `PersistenceManager.swift`: file-backed conversation storage, metadata index, UserDefaults-backed settings, and legacy migration.
- `SettingsView.swift`: model picker, speech speed, temperature, feature toggles, web search, system prompt, and clear-all action.
- `MessageView.swift`: message bubble rendering, markdown/math/code display, citations sheet, TTS trigger, context menu, and streaming indicator.
- `MarkdownParser.swift`: lightweight markdown/math/code parsing with precompiled regexes and a small parse cache.
- `Speaker.swift`: `AVSpeechSynthesizer` wrapper and markdown cleanup for speech output.
- `Branding.swift`: shared Gemini gradient and sparkle mark.

## Data Flow

Message send flow:

1. `ContentView` submits text to `ChatViewModel.sendMessage`.
2. The view model appends a user `Message` and persists the current conversation.
3. `ChatViewModel.processRequest` reads current settings from `AppSettingsStore`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyroz1/gemini-watch](https://github.com/cyroz1/gemini-watch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
