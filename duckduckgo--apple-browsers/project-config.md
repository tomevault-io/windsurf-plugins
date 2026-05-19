---
trigger: always_on
description: This is the DuckDuckGo browser for iOS and macOS, built with privacy-first principles, modern Swift patterns, and cross-platform architecture.
---


# DuckDuckGo Browser Development Rules Overview

## Project Context
This is the DuckDuckGo browser for iOS and macOS, built with privacy-first principles, modern Swift patterns, and cross-platform architecture.

**Key Directories:**
- `iOS/` - iOS browser app (UIKit + SwiftUI hybrid)
- `macOS/` - macOS browser app (AppKit + SwiftUI hybrid) 
- `SharedPackages/` - Cross-platform Swift packages

## Architecture Summary
- **Pattern**: MVVM + Coordinators + Dependency Injection
- **UI**: SwiftUI preferred, UIKit/AppKit for legacy
- **Storage**: Core Data + GRDB + Keychain for sensitive data
- **Design**: DesignResourcesKit for colors/icons (MANDATORY)
- **Testing**: >80% coverage required

## Available Rules (`.cursor/rules/`)

Development rules are stored in `.cursor/rules/`.
You MUST list all the available rules and you MUST consult the appropriate rule file before starting any work!

### Core (Always Apply)
- `anti-patterns.mdc` - What NOT to do; use with ViewModels, testing, WebView work
- `code-style.mdc` - Swift style guide
- `privacy-security.mdc` - Privacy requirements; use with network calls, analytics, credentials
- `import-hygiene.mdc` - Import management and SwiftUI preview scoping
- `logging-guidelines.mdc` - Logger usage (never print())

### Architecture & Patterns
- `architecture.mdc` - MVVM, DI patterns; use for new ViewModels
- `project-structure.mdc` - Directory layout
- `browserserviceskit-integration.mdc` - BSK integration; use for cross-platform code
- `shared-packages.mdc` - Cross-platform packages; use for cross-platform code
- `subscription-architecture.mdc` - Privacy Pro subscription

### Feature Development
- `feature-flags.mdc` + `feature-flags-addition.mdc` - Feature flags
- `abn-experiment-framework.mdc` - A/B testing
- `user-defaults-storage.mdc` - UserDefaults, @UserDefaultsWrapper; use for settings/preferences

### UI Development
- `swiftui-style.mdc` - SwiftUI + DesignResourcesKit; use for new ViewModels, UI work
- `swiftui-advanced.mdc` - Advanced SwiftUI patterns
- `design-system-designresourceskit.mdc` - Colors, typography, icons (MANDATORY)
- `webkit-browser.mdc` - WebView patterns

### Platform-Specific
- `ios-architecture.mdc` - iOS AppDependencyProvider, MainCoordinator, UIKit
- `ios-tracker-blocking-implementation.mdc` - iOS content blocking
- `macos-window-management.mdc` - macOS windows
- `macos-system-integration.mdc` - macOS system services
- `macos-singletons-removal.mdc` - Removing singletons from macOS

### Feature-Specific
- `duckplayer.mdc` + `duckplayer-userscript-integration.mdc` - DuckPlayer
- `securevault-guidelines.mdc` - Credentials/vault storage
- `app-lifecycle-state-machine.mdc` - App state management
- `network-quality-*.mdc` (4 files) - Network quality assessment

### Testing & Quality
- `testing.mdc` - Testing patterns, xcodebuild commands
- `ui-testing.mdc` - UI testing for macOS browser
- `maestro-device-selection.mdc` - Maestro test device config
- `performance-optimization.mdc` - Performance; use with network calls

### Workflow & Process
- `development-commands.mdc` - Build commands
- `pull-request.mdc` + `branch-naming-conventions.mdc` - PRs and git workflow
- `analytics-patterns.mdc` - Pixel analytics

## Quick Start Checklist

### Before Writing Any Code:
1. ✅ Read `privacy-security.mdc` - Privacy is non-negotiable
2. ✅ Check platform rules (`ios-architecture.mdc` or `macos-system-integration.mdc`)
3. ✅ Review `anti-patterns.mdc` - Avoid common mistakes
4. ✅ REMEMBER: NEVER commit, push, or run tests without explicit user permission or unless explicitly asked to

### For UI Development:
1. ✅ Use `swiftui-style.mdc` for SwiftUI components
2. ✅ MUST use DesignResourcesKit colors: `Color(designSystemColor: .textPrimary)`
3. ✅ MUST use DesignResourcesKit icons: `DesignSystemImages.Glyphs.Size16.add`

### For New Features:
1. ✅ Follow `architecture.mdc` for MVVM + DI patterns
2. ✅ Use AppDependencyProvider (iOS) or equivalent (macOS)
3. ✅ Write tests per `testing.mdc` requirements

## Critical Don'ts (from anti-patterns.mdc)
- ❌ NEVER commit, push changes, create or delete branches on git or trigger github actions without EXPLICIT user permission
- ❌ NEVER run tests without EXPLICIT user permission or if user explicitly asked to in their prompt
- ❌ NEVER use `.shared` singletons - use dependency injection instead
- ❌ NEVER hardcode colors/icons (use DesignResourcesKit)
- ❌ NEVER update UI without @MainActor
- ❌ NEVER ignore privacy implications
- ❌ NEVER force unwrap without justification
- ❌ NEVER use `print()` statements - use appropriate Logger extensions instead

## Logging Guidelines

**NEVER use `print()` in production code. ALWAYS use appropriate Logger extensions:**

**Example:** See [logging-guidelines.swift](general/logging-guidelines.swift)

**Available Logger categories:**
- `Logger.general` - General app functionality
- `Logger.network` - Network requests and responses  
- `Logger.ui` - UI updates and user interactions
- `Logger.tests` - Test-specific logging (import `os.log` in tests)

**Benefits of Logger extensions:**
- Structured logging with categories and levels
- Better performance than print() statements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
