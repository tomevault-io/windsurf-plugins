---
trigger: always_on
description: iOS 17+ SwiftUI app for ADHD-friendly task management. Core UX: **one card at a time** — avoid lists when a single card view will do.
---

# Nudge — Copilot Instructions

## Product Intent
iOS 17+ SwiftUI app for ADHD-friendly task management. Core UX: **one card at a time** — avoid lists when a single card view will do.

## Architecture Overview

**Targets:** `Nudge` (main), `NudgeShareExtension`, `NudgeWidgetExtension` (Live Activity). All share data via App Group `group.com.tarsitgroup.nudge`.

**Bundle ID:** `com.tarsitgroup.nudge`

**App lifecycle (NudgeApp.swift):** 4-tier routing: Intro → Auth → Onboarding → ContentView. `ModelContainer` is **nil until auth completes** (per-user store: `nudge_{userID}.store` in App Group). Falls back to in-memory on corruption. On foreground: resets daily counters, ingests share extension items, resurfaces expired snoozes, schedules notifications, syncs CloudKit.

**Key services** are singletons (`HapticService.shared`, `AIService.shared`, `SoundService.shared`, etc.). **Exception:** `NudgeRepository` is NOT a singleton — instantiated per-use from a `ModelContext`.

**Environment injection at root:** `AppSettings`, `AccentColorSystem`, `PenguinState`, `AuthSession` — all `@Observable`, injected via `.environment()`. `modelContainer` is injected only after auth, not at root.

## Data Layer (SwiftData)

- **Models:** `NudgeItem`, `BrainDump` (1-to-many), `NudgyWardrobe`. Schema includes all three.
- **Views never touch `ModelContext` directly** — always go through `NudgeRepository(modelContext:)`.
- **Enum storage:** Models store enums as raw strings (`statusRaw`, `actionTypeRaw`). `#Predicate` must compare raw strings:
  ```swift
  // ✅ #Predicate<NudgeItem> { $0.statusRaw == "active" }
  // ❌ #Predicate<NudgeItem> { $0.status == .active }  — won't compile
  ```
- **⚠️ ActionType raw values are UPPERCASE** (`"CALL"`, `"TEXT"`, `"EMAIL"`, `"LINK"`); all other enums use lowercase.
- **Optional dates in predicates:** Cannot unwrap `Date?` inside `#Predicate`. Fetch broader, filter in-memory (see `fetchCompletedToday()`, `resurfaceExpiredSnoozes()`).
- **`createFromBrainDump()` does NOT call `save()`** — caller must batch-save. All other create methods auto-save.

## Auth Flow

- **3 auth methods:** Apple Sign In, email/password (local-only, SHA-256 hash in Keychain), debug bypass (`-skipAuth` launch arg).
- `AuthSession` state machine: `.checking` → `.signedOut(reason:)` | `.signedIn(UserContext)`.
- `AppSettings` scopes per-user keys with `{userID}:` prefix. Some keys (quiet hours, `hasSeenIntro`) are global.
- For two-way bindings in views: `@Bindable var settings = settings`.

## View Patterns

- **No ViewModel for most views** — work directly with `NudgeRepository` + `@State`. Only `BrainDumpView` has a dedicated `BrainDumpViewModel` (`@Observable`, `Phase` enum state machine with `.tag: String` for `.onChange()`).
- **Sheet presentation:** Brain dump → `.fullScreenCover`. Others → `.sheet` with `.presentationDetents([.medium])`.
- **`.preferredColorScheme(.dark)`** enforced at TabView level.
- **Deep links:** `nudge://brainDump`, `nudge://quickAdd`, `nudge://viewTask?id=`, `nudge://markDone?id=`, `nudge://snooze?id=`, `nudge://allItems`, `nudge://settings`, `nudge://chat` — handled in `ContentView.handleDeepLink(_:)`.
- **3 tabs:** `.nudgy` (penguin home), `.nudges` (task cards), `.you` (profile/settings).

## Design System

All colors from `DesignTokens` in `Nudge/Core/Constants.swift`. **Never use raw color literals.**
- Canvas: `#000000` (OLED). Cards: `#1C1C1E` @ 80% opacity + 0.5px border. Use `DarkCard` component.
- Accent colors are **status-driven**: blue=active, green=done, amber=stale(3+days), red=overdue.
- Typography: `AppTheme` static properties (from `Core/Theme/AppTheme.swift`), not raw `.font()`.
- Spacing: 4pt grid via `DesignTokens.spacingXS/SM/MD/LG/XL/XXL/XXXL`.
- Animation: `AnimationConstants` — springs for motion, `.easeOut` for fades, never `.linear`. Respect `@Environment(\.accessibilityReduceMotion)`.
- Mark constant enums as `nonisolated` for safe cross-concurrency access.

## Accessibility

Every custom view must use helpers from `Core/Accessibility/VoiceOverHelpers.swift`:
- `.nudgeAccessibility(label:hint:traits:)`, `.nudgeAccessibilityElement(label:hint:value:)`, `.nudgeAccessibilityAction(name:action:)`.
- Use `.scaledPadding()` / `.scaledFrame()` from `AccessibilityScaling.swift` for Dynamic Type.

## Haptics & Sound

Every interaction maps to a specific `HapticService` method (`.swipeDone()`, `.micStart()`, etc.) — never call `UIImpactFeedbackGenerator` directly. Sound via `SoundService`. Both pre-warmed at launch. **Exception:** Share Extension creates its own feedback generator (no access to singletons).

## AI Integration

**Dual-provider architecture:**
- **Apple Foundation Models** (`FoundationModels` framework, on-device) via `AIService` — uses `@Generable` structs with `@Guide` annotations. Guarded by `#if canImport(FoundationModels)`.
- **OpenAI API** (GPT-4o) via `NudgyConfig` for conversational quality — key from `Secrets.xcconfig` → `Info.plist`.
- **`NudgyEngine.shared`** is the **only** AI entry point for views. Never call sub-engines (`NudgyConversationManager`, `NudgyDialogueEngine`, etc.) directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aimran6775/Nudgy](https://github.com/aimran6775/Nudgy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
