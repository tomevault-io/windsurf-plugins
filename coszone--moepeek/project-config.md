---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

This project uses **Tuist** as its build system with SPM for dependency management.

```bash
# Initial setup (install deps + generate Xcode project)
tuist install && tuist generate

# Regenerate after changing Project.swift or Package.swift
tuist generate

# Build from command line
xcodebuild -workspace MoePeek.xcworkspace -scheme MoePeek -configuration Debug build

# Open in Xcode
open MoePeek.xcworkspace
```

No tests, linting, or CI/CD are currently configured.

## Tech Stack

- **Swift 6.0+** with `SWIFT_STRICT_CONCURRENCY: complete`
- **macOS 14.0+** deployment target, LSUIElement (menu bar app, no Dock icon)
- **SwiftUI + AppKit hybrid**: SwiftUI for views, AppKit NSPanel for non-activating floating windows
- **Dependencies**: KeyboardShortcuts (sindresorhus), Defaults (sindresorhus)
- **License**: AGPL-3.0

## Architecture

### Core Data Flow

```
User Action (shortcut / mouse selection / OCR)
  → TranslationCoordinator (state machine: idle → grabbing → translating → streaming → completed/error)
    → Text Grabbing: TextSelectionManager (3-tier fallback: AX API → AppleScript → Clipboard)
    → Language Detection: LanguageDetector (NLLanguageRecognizer, auto-flips target if same as detected)
    → Translation: TranslationService protocol (OpenAI streaming API or Apple Translation on macOS 15+)
  → PopupPanelController (floating result panel at cursor)
```

### Key Patterns

- **@MainActor everywhere**: All UI controllers and coordinators are `@MainActor`-isolated. `@Observable` macro for state observation.
- **Non-activating NSPanels**: `PopupPanel` and `TriggerIconPanel` are borderless floating panels that never steal focus from the user's active app.
- **Coordinator pattern**: `TranslationCoordinator` owns all translation logic and exposes a single `State` enum consumed by views.
- **Callback wiring in AppDelegate**: `AppDelegate.setupSelectionMonitor()` wires together SelectionMonitor → TriggerIconController → TranslationCoordinator → PopupPanelController via closures.
- **3-tier text grabbing**: `AccessibilityGrabber` (AX API) → `AppleScriptGrabber` (Safari-specific) → `ClipboardGrabber` (⌘+C simulation). Each tier tried in order.
- **NSViewRepresentable over TextEditor**: `TextEditor` has implicit insets; use custom `NSTextView` wrapper with `textContainerInset = .zero` / `lineFragmentPadding = 0` when precise spacing control is needed. See `SourceTextEditor`.
- **Settings window activation**: `SettingsLink` is unreliable in LSUIElement / non-activating panel contexts. Use `NSApp.activate(ignoringOtherApps:)` → `sendAction("showPreferencesWindow:")` (private but stable selector) → fallback `openSettings()`.
- **UI spacing constants**: PopupView uses `contentHorizontalPadding` for all edge insets; new subviews should reference it instead of hardcoding.
- **@Observable and computed properties**: `@Observable` only tracks **stored properties**; computed property setters generate no observation notifications.
  - ❌ **Never** wrap "write to external state" logic as a computed property on an `@Observable` class and expose it as a SwiftUI binding — the classic symptom is a Toggle/Picker that appears frozen (the value is written, but the view never re-renders).
    - Broken: `var foo: Bool { get { ext.foo } set { ext.foo = newValue } }`
  - ✅ **Fix**: use a stored property + `didSet` to sync outward, initialized from the external source in `init()`.
    - `var foo: Bool = false { didSet { ext.foo = foo } }` + `init() { foo = ext.foo }`
  - ✅ **Safe**: read-only computed properties derived from stored properties — access tracking propagates through them correctly.

### Source Layout

| Directory | Purpose |
|-----------|---------|
| `Sources/App/` | SwiftUI app entry + AppDelegate lifecycle & wiring |
| `Sources/Core/` | Text grabbing, OCR, permissions, TranslationCoordinator |
| `Sources/Services/` | TranslationService protocol + OpenAI/Apple implementations |
| `Sources/UI/` | PopupPanel, TriggerIcon, MenuBar, Settings, Onboarding |
| `Sources/Utilities/` | Constants (Defaults keys, keyboard shortcuts), KeychainHelper, positioning |

### Internationalization (i18n)

The app supports **English** (development language) and **Simplified Chinese** (zh-Hans) via Xcode String Catalogs.

- **`Resources/Localizable.xcstrings`** — Single String Catalog containing all localized strings with en keys and zh-Hans translations.
- **SwiftUI views** use string literals as `LocalizedStringKey` (automatic lookup).
- **Non-UI code** (errors, coordinators) uses `String(localized:)` for runtime localization.
- **`SupportedLanguages`** in `Constants.swift` uses `Locale.current.localizedString(forIdentifier:)` for dynamic language names.
- Strings that should **NOT** be localized: API technical labels (`"Base URL:"`, `"API Key:"`, `"Model:"`), LLM system prompts, provider IDs, copyright/license text, brand name "MoePeek".

### Critical Files

- **`TranslationCoordinator.swift`** — Central orchestrator with state machine; all translation flows route through here

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cosZone/MoePeek](https://github.com/cosZone/MoePeek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
