---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**atst** (`a(i)-text-select-translate`) is a tiny macOS menu-bar translator written in pure Swift + SwiftUI + AppKit. No third-party deps. macOS 13+, Swift 5.9+ (Package.swift). The product is shipped as a self-signed `.app` inside a DMG.

## Commands

```bash
# Build for debug + run inline (fastest dev loop)
swift run atst

# Compile only
swift build

# Package a fully-formed .app bundle (release config, icons, Info.plist,
# ad-hoc codesign). Output: .build/atst.app
bash Scripts/build-app.sh

# Same as above, then wrap in a compressed DMG with /Applications symlink.
# Output: .build/atst.dmg
bash Scripts/build-dmg.sh

# Cut a release. Reads ## vX.Y.Z sections from both CHANGELOG files,
# builds DMG, tags, pushes tag, creates GitHub release titled "vX.Y.Z"
# with bilingual notes auto-stitched from CHANGELOG + sha256 in body.
# Refuses on dirty tree / missing CHANGELOG section / duplicate tag.
bash Scripts/release.sh v0.1.3
```

There is **no test suite**. Verify changes by `swift build` (catches type errors), then `bash Scripts/build-app.sh && open .build/atst.app` for runtime smoke tests.

## High-level architecture

### Two parallel translation flows

The app has two top-level user actions that take very different code paths:

1. **Selection translation** (`⌥D`) → `AppDelegate.translateSelection` → `viewModel.translateSelection` → fans out across every enabled `TranslationProvider` in parallel. The UI shows API rows (Google / Microsoft) stacked above an AI row.

2. **Screenshot translation** (`⌥S`) → `AppDelegate.translateScreenshot` → forks on `screenshotUseVisionOCR`:
   - **OCR ON (default)**: `VisionOCRService` → text → reuse `translateSelection` (so screenshots also get the multi-provider UI)
   - **OCR OFF, or OCR returned no text**: fall back to AI vision via `ScreenshotVisionService` (the legacy single-source path)
   - **AI vision unavailable**: surface `AppError.aiDisabledForVision` with a recovery hint

Anything you change to selection translation automatically benefits screenshot-with-OCR. Don't try to unify the AI vision path into `TranslationProvider` — image input has fundamentally different message shape and constraints, and `Google` / `Microsoft` can't accept images anyway.

### `TranslationProvider` protocol

In `Sources/atst/Translation/TranslationProvider.swift`. Three implementations:

- `OpenAIProvider` — OpenAI-compatible Chat Completions, SSE streaming, multimodal-aware (text path only; screenshot path lives separately). Owns the XML tag protocol (`<atst-result>` / `<atst-item>` / `<atst-phonetic>` / `<atst-desc>` / `<atst-translatable>`).
- `GoogleProvider` — unofficial `translate-pa.googleapis.com` endpoint. Public API key baked in. HTML entity decoder required for the response.
- `MicrosoftProvider` — unofficial Edge translator endpoint. JWT auth via `MicrosoftAuthToken` actor (cached, auto-refresh with 30s buffer, 401 retry).

All providers return `AsyncThrowingStream<TranslationProviderEmission, Error>` so streaming (AI) and one-shot (API) flows share the same surface. `TranslatorViewModel` doesn't know which kind it's driving.

### Per-segment state

`TranslationState.text(TextSegments)` is the main translation state. `TextSegments` carries an array of `ProviderSegment` for API rows + an optional `ProviderSegment` for AI. Each segment has its own lifecycle (`loading` / `streaming` / `success` / `failure`) and is updated independently as its provider's stream emits. Screenshot AI vision uses its own state cases (`screenshotLoading` / `screenshotStreaming` / `screenshotSuccess`) — they're not crammed into `text(...)`.

When adding a provider: implement `TranslationProvider`, surface it in `TranslatorViewModel.makeProvider(for:)`, add an `APIProviderEntry` default in `AppConfiguration.defaultAPIProviders`.

### Cache schema (v2)

`TranslationCache` keys are per-segment:

- AI: `v2 | ai | <model> | <targetLang> | p<0/1> | e<0/1> | <normalize(text)>` (phonetic/explanation toggles invalidate)
- API: `v2 | <providerId> | <targetLang> | <normalize(text)>`

Untranslatable results (AI: `<atst-translatable>false</atst-translatable>`; API: source == result heuristic) skip the cache. Legacy `v1|…` keys are auto-purged on launch.

### Bilingual UI

Every user-facing string uses `L.pick("English", "中文")` (in `Sources/atst/Support/L.swift`). `L.override` is set from `AppConfiguration.uiLanguage` (`.auto` follows system, `.english` / `.chinese` lock). `AppDelegate` keeps the override in sync via the `settingsStore.$configuration` sink. **Don't** introduce raw user-facing strings — they break the language toggle.

### Tooltip mechanics

`FloatingPanelController` owns a single live `NSPanel` reused across translations. Sizing is driven from SwiftUI (`readSize` in `TranslationResultView`) into `setContentSize` with top-left anchoring (see the `TooltipPanel` subclass) so animations don't fight a moving y-origin.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itaober/atst](https://github.com/itaober/atst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
