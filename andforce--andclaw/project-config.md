---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Andclaw is an Android automation assistant powered by Large Language Models (LLMs). It uses Android Accessibility Services to "read" UI hierarchy and performs actions based on natural language commands. Supports remote control via Telegram Bot.

## Build Commands

This is a Gradle-based Android project with multiple modules.

```bash
# Build the project
./gradlew build

# Install debug APK to connected device
./gradlew :app:installDebug

# Clean build
./gradlew clean

# Run tests
./gradlew test

# Run Android instrumented tests
./gradlew connectedAndroidTest
```

### Required Setup

1. Create `local.properties` with your API keys:
   ```properties
   kimi_key=your_kimi_api_key
   tg_token=your_telegram_bot_token  # Optional, for Telegram integration
   ```

2. The app requires these permissions at runtime:
   - **Accessibility Service**: Must be manually enabled in Settings > Accessibility
   - **Display over other apps**: For the emergency stop floating button

## Project Architecture

### Module Structure

| Module | Purpose |
|--------|---------|
| `:app` | Main application module containing AI agent logic, UI, and accessibility service |
| `:model` | Kimi API client (`KimiApiClient`) using Anthropic messages format |
| `:mdm` | Mobile Device Management - Device Owner policies, Kiosk mode, silent app install |
| `:services` | Kotlin interface definitions for cross-module communication (NOT AIDL) |
| `:commonUtils` | Common Java/Kotlin utilities shared across modules |
| `:network` | Network utilities, Retrofit factory, and file download manager |
| `:setupdesign`, `:setupcompat`, `:setupdesign_strings` | Setup Wizard UI components (from Google testdpc) |

### Module Dependencies

```
app → model, mdm, services, commonUtils
mdm → setupdesign, setupcompat, setupdesign_strings, services, commonUtils
setupdesign → setupcompat, setupdesign_strings
```

### Key Components (app module)

**AgentController** (`app/.../AgentController.kt`)
- Singleton `object` that orchestrates the AI agent loop
- Implements `ITgBridgeService` and `IAiConfigService`
- Manages LLM API calls via `Utils.callLLMWithHistory()`
- Handles Telegram bot integration for remote control (`TgBotClient`)
- Maintains conversation state in `MutableStateFlow<List<ChatMessage>>`
- Loop detection via action fingerprint tracking (`consecutiveSameCount`)

**AgentAccessibilityService** (`app/.../AgentAccessibilityService.kt`)
- Android AccessibilityService providing screen perception and interaction
- `captureScreenHierarchy()`: Captures UI hierarchy as text with element bounds
- `click(x, y)`: Simulates tap gestures using `dispatchGesture()`
- `swipe()`, `longPress()`: Gesture simulation
- `inputText()`: Text injection (SET_TEXT → clipboard paste fallback)
- `isWebViewContext()`: Detects browser/WebView for screenshot-based analysis
- `captureScreenshot()`: Takes screenshot via `takeScreenshot()` API (API 30+)
- `globalAction()`: System-level actions (back, home, recents, etc.)

**Utils** (`app/.../Utils.kt`)
- `buildAgentSystemPrompt()`: Constructs the system prompt with all action type documentation
- `callLLMWithHistory()`: Multi-provider LLM call supporting Kimi (Anthropic format) and OpenAI-compatible APIs
- `parseAction()`: Extracts JSON from LLM response, handles markdown fences and array wrapping
- Supports multimodal input (screenshot base64) for both Kimi and OpenAI providers

**DpmBridge** (`app/.../DpmBridge.kt`)
- Bridges AI actions to Device Policy Manager operations
- `execute()`: Dispatches 40+ DPM actions (lock, reboot, install/uninstall, permissions, kiosk, etc.)
- `getSafetyLevel()`: Classifies actions as SAFE / CONFIRM / DANGEROUS

**TgBotClient** (`app/.../TgBotClient.kt`)
- Telegram Bot API client using OkHttp
- `poll()`: Long-polling `getUpdates`
- `send()`, `sendPhoto()`, `sendVideo()`: Message/media delivery

### Action Types (`app/.../model/Models.kt`)

The AI returns JSON actions with these types:
- `intent` — Launch apps/activities with extras
- `click` — Tap at screen coordinates (x, y)
- `swipe` — Swipe from (x, y) to (endX, endY)
- `long_press` — Long press at coordinates
- `text_input` — Type text into focused fields
- `global_action` — System actions (back, home, recents, notifications, quick_settings)
- `screenshot` — Capture screen and save to gallery
- `download` — Download file by URL via DownloadManager
- `wait` — Wait for UI transition, then re-check screen
- `camera` — Take photo or record video (take_photo, start_video, stop_video)
- `screen_record` — Screen recording via MediaProjection (start_record, stop_record)
- `volume` — Volume control (set, adjust_up, adjust_down, mute, unmute, get)
- `dpm` — Device Policy Manager actions (Device Owner only)
- `finish` — Task completed

### AI Agent Loop

1. User provides command (text via UI, or remotely via Telegram Bot)
2. `AgentAccessibilityService.captureScreenHierarchy()` captures current UI tree
3. If in WebView/browser context, also captures screenshot for visual analysis
4. Prompt sent to LLM with: system prompt + last 12 messages history + screen data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andforce/Andclaw](https://github.com/andforce/Andclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
