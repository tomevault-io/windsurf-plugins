---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Maintenance Rules
- This CLAUDE.md is a living document. After any major architectural change, refactor, or new convention, update the relevant sections immediately.
- When I say “update CLAUDE.md”, revise only the changed parts and keep the file concise.
- **Self-update on every commit**: after each `git commit`, review whether the commit changed architecture, conventions, or build/network/platform wiring; if so, update the relevant CLAUDE.md sections in the same change. A `PostToolUse` hook in `.claude/settings.json` injects this reminder after commits run inside Claude Code.

## Project Overview

**DeconstructChinese** — Kotlin Multiplatform Compose app for Chinese character translation and learning. Targets Android, iOS, Web (JS/WASM). Translates text via an OpenAI-compatible LLM provider (Qwen on Alibaba DashScope by default), stores vocabulary locally with frequency tracking.

### Technology Stack

- **KMP**: Kotlin 2.3, Compose Multiplatform 1.10
- **Network**: Ktor Client 3.0 (OkHttp on Android, Darwin on iOS)
- **State**: ViewModel + StateFlow, Multiplatform Settings for persistence
- **Translation**: `TranslationService` interface over OpenAI-compatible chat/completions; Qwen (`qwen-plus` on DashScope) is the wired default; Doubao (`seed-2-0-lite-260228`) and OpenRouter adapters also present but unused
- **Build**: Gradle 8.11 with version catalog (libs.versions.toml)
- **Audio**: Platform-specific TTS (Android `TextToSpeech`, iOS `AVSpeechSynthesizer`; web stub)
- **Speech Input**: Hold-to-record via `SpeechRecognizer` expect/actual (Android `android.speech`, iOS `SFSpeechRecognizer`)
- **API Key**: `Secrets` expect/actual `defaultApiKey` — Android pulls from `BuildConfig.QWEN_API_KEY` (build reads `qwen.apiKey` from `local.properties`), iOS uses generated source via `generateIosSecrets` Gradle task, Web returns `""`. The provider key is **bundled**; there is no user-facing API-key entry. `AppSettings.apiKey` returns `defaultApiKey`.

### Target Platforms

| Platform | Min SDK | Target SDK | Details |
|----------|---------|------------|---------|
| Android | 29 | 36 | OkHttp client, Google Play Services on Android |
| iOS | 13+ | Arm64 + SimulatorArm64 | Darwin (native) HTTP client |
| Web | N/A | Modern browsers | JS and WASM targets (audio TTS not implemented) |

## Build Commands

### Android
```bash
# Debug build + install
./gradlew :composeApp:assembleDebug
./gradlew :composeApp:installDebug

# Run on connected device/emulator
./gradlew :composeApp:installDebug

# Tests
./gradlew :composeApp:connectedAndroidTest

# Signed release bundle for Play Store (needs keystore.properties at repo root)
./gradlew :composeApp:bundleRelease   # -> composeApp/build/outputs/bundle/release/composeApp-release.aab
```

**Release signing**: `signingConfigs.release` reads `storeFile`/`storePassword`/`keyAlias`/`keyPassword` from `keystore.properties` (repo root, **gitignored**, alongside the `*.jks`). If absent, release builds are unsigned. Uses Play App Signing (upload key).

**versionCode**: auto-derived from the git commit count (`git rev-list --count HEAD` via config-cache-safe `providers.exec`) — every commit bumps it by 1, no manual edits. Release builds need full git history (not a shallow clone); don't squash already-released history or the count can regress below a code Play has accepted. `versionName` stays manual.

### iOS
Open `/iosApp` in Xcode and run via IDE (KMP bridging through framework in `composeApp/build/` after Gradle sync).

### Web
```bash
# WASM (faster, modern browsers)
./gradlew :composeApp:wasmJsBrowserDevelopmentRun

# JS (slower, older browser support)
./gradlew :composeApp:jsBrowserDevelopmentRun
```

### Common
```bash
# Shared code tests
./gradlew commonTest

# Full test suite
./gradlew test
```

## Architecture

### State Management (ViewModel Pattern)

**TranslatorViewModel** holds UI state, owns coroutine scope (viewModelScope):
- `translationState`: Current translation (Idle, Loading, Success, Error — sealed class)
- `inputText`: User input with 800ms debounce before translate
- `toEnglish`: Direction toggle
- `useSimplified`: Traditional vs simplified preference
- `savedVocabulary`: StateFlow from VocabularyStore
- `isPlaying`: Audio playback status
- `recordingPhase`: `RecordingPhase` enum (`Idle`/`Armed`/`Listening`) driven by `SpeechRecognizer.results` flow
- `snackbarMessage`: SharedFlow for speech errors (non-fatal, shown as snackbar)
- `onSharedText(String)`: entry point from `IncomingText` bus; auto-sets direction by detecting Han chars
- `startRecording()` / `stopRecording()`: wraps SpeechRecognizer; locale derived from `toEnglish` + `useSimplified`

`TranslationState` sealed class lives in `model/TranslationResult.kt` alongside `TranslationResult`, `VocabularyItem`, `Language`.

ViewModel created once per app lifecycle; state flows collected in Compose via `collectAsStateWithLifecycle()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mettyoung/deconstruct-chinese](https://github.com/mettyoung/deconstruct-chinese) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
