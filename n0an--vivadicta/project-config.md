---
trigger: always_on
description: This file provides guidance for coding agents working in this repository.
---

# AGENTS.md

This file provides guidance for coding agents working in this repository.
If you don't know answer - it's ok to say that you don't know. It's not necessary to please me and be very polite with me. I'm not pleased if you always agree with everything I'm saying. I'm pleased when you provide correct answers, even if I don't like these answers.

## Skills

Canonical custom skills live in `.agents/skills/` as directories containing `SKILL.md`. `.claude/skills/` mirrors only Claude-compatible skills for compatibility and should contain only symlinks to `.agents/skills/`. Codex-only skills do not need a `.claude` mirror.

## Git Commit & PR Guidelines
- NEVER commit and push without asking the user first - always ask "do we need to commit and push at the moment?" before executing git commit or git push commands, unless the user explicitly invokes `$pr` or `$prcdx`
- Treat `$pr` and `$prcdx` as explicit approval to create a branch if needed, commit, push, and open the PR without separate confirmation
- When user asks to commit/push, ALWAYS run `git status` first instead of relying on chat context — files may have been modified externally
- When opening a PR, use `$prcdx` in Codex. In Claude Code, use `/pr`.

## App Store Connect

- **App ID**: `6758147238`
- **App Name**: VivaDicta - Speech to Text
- **Bundle ID**: `com.antonnovoselov.VivaDicta`
- **Submission quirk**: `asc review submissions-create` creates an empty draft — it does NOT add the version item. Either use `asc submit create` for simple submissions, or follow `submissions-create` with `items-add` + `submissions-submit`. When in doubt, tell the user to submit via the "Add for Review" button in App Store Connect.

## Build Commands

- Build: `xcodebuild -scheme VivaDicta -configuration Debug -workspace ./VivaDicta.xcodeproj/project.xcworkspace -destination 'platform=iOS Simulator,name=iPhone 17 Pro Max,OS=26.4' build 2>&1 | xcsift`
- Run tests: `xcodebuild -scheme VivaDicta -configuration Debug -workspace ./VivaDicta.xcodeproj/project.xcworkspace -destination 'platform=iOS Simulator,name=iPhone 17 Pro Max,OS=26.4' test 2>&1 | xcsift`
- Run single test: `xcodebuild -scheme VivaDicta -configuration Debug -workspace ./VivaDicta.xcodeproj/project.xcworkspace -destination 'platform=iOS Simulator,name=iPhone 17 Pro Max,OS=26.4' test -only-testing:VivaDictaTests/TestClassName/testMethodName 2>&1 | xcsift`

## App Overview

VivaDicta is an iOS voice transcription app with on-device (WhisperKit, Parakeet) and cloud transcription, AI text processing via multiple providers, and CloudKit sync with a companion macOS app (VivaDictaMac).

## Architecture

### Core Flow: Recording → Transcription → AI Processing → Storage

1. **RecordView/RecordViewModel** — records audio via AVAudioRecorder
2. **TranscriptionManager** — routes to on-device (WhisperKit/Parakeet) or cloud provider
3. **AIService** — AI text processing using the mode's active preset. Builds system/user messages via `PromptsTemplates`, sends to cloud providers or Apple Foundation Model
4. **Text Processing Pipeline** — multi-stage: raw text → word replacements → custom vocabulary → AI processing → output filter → paragraph formatting → text insertion formatting. See `documentation/text-processing-pipeline.md`
5. **Transcription** (SwiftData) — persisted with `text`, `enhancedText`, audio file reference, and linked `TranscriptionVariation` records

### Data Model: Transcription + Variations (Dual-Write Pattern)

- **`Transcription`** — SwiftData model with `text` (original), `enhancedText` (latest AI output cache), and `@Relationship(deleteRule: .cascade) var variations: [TranscriptionVariation]?`
- **`TranscriptionVariation`** — each AI-generated output stored separately with `presetId`, `text`, `aiModelName`, `processingDuration`, etc.
- **Dual-write**: When AI processes text, both `transcription.enhancedText` and a `TranscriptionVariation` are written. When a new variation is generated from TranscriptionDetailView, `enhancedText` is updated to the latest result.
- **`enhancedText`** serves as a "latest AI output" cache read by: list row preview, search predicate, Spotlight indexing, Shortcuts/App Intents, clipboard operations
- **`VariationMigrationService`** — one-time migration of legacy `enhancedText` values to "regular" variations on first launch

### Preset System

- **`Preset`** — in-memory value type with `id`, `name`, `instructions`, `icon`
- **`PresetCatalog`** — static catalog of built-in presets (Regular, Summary, Action Points, Professional, Casual, Email, Chat, Coding, Rewrite, translations, Assistant) with stable UUIDs
- **`PresetManager`** — manages active presets, merges built-in + custom, handles mode-to-preset mapping
- **`CustomRewritePreset`** — SwiftData model for user-created presets, synced via CloudKit
- **`PresetSyncService`** — bridges UserDefaults-stored preset selections with SwiftData/CloudKit for cross-device sync of edited built-in presets

### AI Processing (AIService)

- **`AIService`** — central service for all AI text processing. Uses `VivaMode` for per-mode configuration (provider, model, preset)
- **`PromptsTemplates`** — unified system prompt template used by all providers (cloud and Apple FM)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [n0an/VivaDicta](https://github.com/n0an/VivaDicta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
