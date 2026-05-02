---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Fud AI is an open-source iOS calorie tracker (SwiftUI, iOS 17.6+). Snap/speak/type a meal, an AI provider returns nutrition JSON, the user reviews it, and it lands in `FoodStore` + Apple Health. There's also a "Coach" tab — multi-turn AI chat that sees the user's full profile, weight history, and food log and answers questions like "what's my expected weight in 30 days?". Bring-your-own-key model; all data is local. No subscriptions, no sign-in, no cloud sync.

## Build, Install, Launch

The app is tested on Apoorv's physical iPhone (iPhone 16, device ID `E2095CDC-E117-527C-818A-9F741A145103`). After every change run all three commands. The Release config is intentional — it matches what users actually see.

```bash
# Build
xcodebuild -scheme calorietracker -destination 'id=E2095CDC-E117-527C-818A-9F741A145103' build

# Install
xcrun devicectl device install app --device E2095CDC-E117-527C-818A-9F741A145103 \
  ~/Library/Developer/Xcode/DerivedData/calorietracker-gyjqfuacfxocddfrskbcdsbwqhxa/Build/Products/Release-iphoneos/calorietracker.app

# Launch
xcrun devicectl device process launch --device E2095CDC-E117-527C-818A-9F741A145103 com.apoorvdarshan.calorietracker

# Pass --reset-onboarding to test the onboarding flow:
xcrun devicectl device process launch --device E2095CDC-E117-527C-818A-9F741A145103 com.apoorvdarshan.calorietracker -- --reset-onboarding
```

## Tests

`calorietrackerTests` and `calorietrackerUITests` targets exist but only contain Xcode boilerplate — there are no real tests. Verify behavior by hand on device. If you do add tests, run them with:

```bash
xcodebuild test -scheme calorietracker -destination 'id=E2095CDC-E117-527C-818A-9F741A145103'
```

## Code Review

Use Codex CLI before each PR / after each commit cluster:

```bash
codex exec review --commit <SHA> --full-auto
```

Address P1 and P2 findings. P3 is judgment-call.

## Architecture

### State / Dependency Injection

All stores use Swift's `@Observable` macro (not `ObservableObject`) and are injected with `.environment(...)` (not `.environmentObject(...)`). Created once in `calorietrackerApp.swift` and shared:

- `FoodStore` — food entries, favorites, macro aggregates
- `WeightStore` — weight entries; `addEntry` auto-syncs `profile.weightKg` to latest
- `ProfileStore` — **source of truth for `UserProfile`**. All reads/writes go through `profileStore.profile`. It listens for `.userProfileDidChange` and reloads so external writers (WeightStore, HealthKit observer) propagate to every view.
- `ChatStore` — Coach conversation history (persisted in UserDefaults as JSON, capped at last 20 messages in LLM payload)
- `NotificationManager`, `HealthKitManager`

Build setting `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor` means most types are main-actor isolated by default. New files are auto-discovered via `PBXFileSystemSynchronizedRootGroup` — **do not** edit `project.pbxproj` to register source files. (The `knownRegions` entry in pbxproj *is* edited when adding a new localization.)

### AI / LLM Routing (13 providers, 3 formats)

Two services, both route to the same 13 providers via `AIProvider.apiFormat`:

- **`GeminiService`** (`Services/GeminiService.swift`) — single-shot food/label analysis. Methods: `analyzeFood`, `analyzeTextInput`, `autoAnalyze`, `analyzeNutritionLabel`. All funnel through `callAI`.
- **`ChatService`** (`Services/ChatService.swift`) — multi-turn Coach chat. Builds a fresh system prompt every turn from the live profile + forecast + recent weights/foods, sends history + new user message.

The three API dialects are:
- **Gemini** (`.gemini`): `POST /models/{model}:generateContent` with `systemInstruction` + `contents[{role, parts}]`. API key goes in `X-goog-api-key` header, not the URL.
- **Anthropic Messages** (`.anthropic`): `POST /messages` with `system` + `messages` array, `x-api-key` header + `anthropic-version: 2023-06-01`.
- **OpenAI-compatible** (`.openaiCompatible`): `POST /chat/completions` with `messages` array (system + user/assistant). Used by OpenAI, xAI Grok, OpenRouter, Together AI, Groq, **Hugging Face** (router for open-weight models — Gemma, Qwen VL, Llama Vision), **Fireworks AI**, **DeepInfra** (open-weight hosts), **Mistral** (Pixtral vision), Ollama (local), and the **Custom (OpenAI-compatible)** provider where the user supplies their own base URL + free-form model name. OpenRouter and Hugging Face both set `supportsCustomModelName = true` so users can type any model ID alongside the preset list.

Adding a provider: add a case to `AIProvider` in `Models/AIProvider.swift`, set `baseURL`/`models`/`apiFormat`/`apiKeyPlaceholder`. If `apiFormat` is `.openaiCompatible` it works automatically; otherwise add a branch in both `GeminiService.callAI` and `ChatService.sendMessage`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apoorvdarshan/fud-ai](https://github.com/apoorvdarshan/fud-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
