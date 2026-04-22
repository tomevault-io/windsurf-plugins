---
trigger: always_on
description: Last updated: 2026-03-24
---

# Repository Guidelines

Last updated: 2026-03-24

## Project Snapshot

- App: `Pindrop` (menu bar macOS app, `LSUIElement` behavior)
- Stack: Swift 5.9+, SwiftUI, SwiftData, Swift Testing, XCTest UI tests
- Platform target: macOS 14+
- Main dependency path: `Pindrop.xcodeproj` + SwiftPM
- Entry points: `Pindrop/PindropApp.swift`, `Pindrop/AppCoordinator.swift`

## Source Layout

- App code: `Pindrop/`
- Services: `Pindrop/Services/`
- UI: `Pindrop/UI/`
- Persistence models: `Pindrop/Models/`
- Utilities/logging: `Pindrop/Utils/`
- Tests: `PindropTests/`
- Test doubles: `PindropTests/TestHelpers/`
- Build automation: `justfile`, `scripts/`, `.github/workflows/`

## Required Local Tooling

- Xcode with command-line tools (`xcodebuild`)
- `just` for all routine workflows: `brew install just`
- Optional: `swiftlint`, `swiftformat`, `create-dmg`
- Apple Developer signing configured in Xcode for signed local/release builds; CI recipes use explicit unsigned overrides

## Build and Run Commands

Prefer `just` recipes over ad-hoc shell commands.

```bash
just build                 # Debug build (ALWAYS use this when testing builds)
just build-release         # Release build
just export-app            # Developer ID export for distribution
just dmg                   # Signed DMG for distribution
just test                  # Unit test plan
just test-integration      # Integration test plan (opt-in)
just test-ui               # UI test plan
just test-all              # Unit + integration + UI
just test-coverage         # Unit tests with coverage
just dev                   # clean + build + test
just ci                    # clean + unsigned build + unsigned test + unsigned release build
just run                   # open Xcode project
just xcode                 # open Xcode project
```

Direct focused test commands:

```bash
xcodebuild test -project Pindrop.xcodeproj -scheme Pindrop -testPlan Unit -destination 'platform=macOS'
xcodebuild test -project Pindrop.xcodeproj -scheme Pindrop -testPlan UI -destination 'platform=macOS'
xcodebuild test -project Pindrop.xcodeproj -scheme Pindrop -destination 'platform=macOS' -only-testing:PindropTests/AudioRecorderTests
xcodebuild test -project Pindrop.xcodeproj -scheme Pindrop -destination 'platform=macOS' -only-testing:PindropTests/AudioRecorderTests/testStartRecordingRequestsPermission
```

## Coding Conventions

- Follow existing file header style (`Created on YYYY-MM-DD`)
- Use `final class` for services and most concrete implementations
- Actor isolation pattern: services are usually `@MainActor`
- Known exception: hotkey internals with Carbon/event constraints
- Use `@Observable` for reactive services where compatible
- `SettingsStore` intentionally uses `ObservableObject` + `@AppStorage`
- Keep import groups consistent with existing files

## Service Patterns

- Dependency injection via initializer arguments (avoid hidden globals)
- Protocol abstractions for hardware/system boundaries
- Example protocol seam: `AudioCaptureBackend` in `Pindrop/Services/AudioRecorder.swift`
- Keep async boundaries explicit (`async` / `async throws`)
- Avoid fire-and-forget tasks unless they are UI/lifecycle orchestration

## Error Handling

- Define domain errors as `enum ...: Error, LocalizedError`
- Keep user-facing messaging in `errorDescription`
- Catch at boundaries, log with context, then rethrow typed errors when possible
- Do not swallow errors with empty catch blocks

## Localization

- **String Catalogs**: `Pindrop/Localization/Localizable.xcstrings` (in-app copy) and `Pindrop/Localization/InfoPlist.xcstrings` (privacy strings, bundle display name). Both are in the app target’s **Copy Bundle Resources**. The top-level `Localization/` tree is the source of truth for the YAML-first pipeline.
- **Runtime API**: `localized("English key", locale: locale)` in `Pindrop/AppLocalization.swift` now resolves through generated stable-key metadata before falling back to `Bundle`; `SettingsStore.selectedAppLocale` drives UI locale and `SettingsStore.selectedAppLanguage` drives dictation/transcription language.
- **New user-facing strings**: Add an entry to the YAML source tree under `Localization/`, then run `just l10n-sync` so the catalogs and generated Swift stay in sync.
- **New language (locale)**: Add the locale with `just l10n-add-locale <locale>` (or edit `Localization/locales.yml`), then populate the relevant `Localization/app/*.yml` and `Localization/infoplist/*.yml` files before syncing.
- **Interface vs dictation language**: The General settings UI now separates interface language from dictation language. Keep `AppLocale`-driven UI locale changes away from `AppLanguage`/transcription behavior.
- **AI enhancement prompts**: `AIEnhancementSettingsView` localizes default prompts for display; if the user saves without editing, the **localized** prompt text can be persisted and sent to the API—expect models to follow non-English system prompts, or keep defaults in English if you change that flow.
- **Localization tooling**: Use `just l10n-import-current`, `just l10n-sync`, and `just l10n-lint`. The old `scripts/translate_xcstrings.py` helper is obsolete.

## Logging

- Use `Log` categories from `Pindrop/Utils/Logger.swift`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [watzon/pindrop](https://github.com/watzon/pindrop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
