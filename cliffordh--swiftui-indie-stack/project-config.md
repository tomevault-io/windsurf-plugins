---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Koala Starter Kit is a production-ready iOS app template with offline-first architecture. It supports three configuration modes controlled by `AppConfiguration`:

- **Local Mode** (`useFirebase = false`): No Firebase required, device-based identity, UserDefaults storage, local streak tracking
- **Cloud Anonymous Mode** (`useFirebase = true, enableAuth = false`): Firebase backend with anonymous accounts, full Firestore sync, no sign-in UI
- **Cloud Full Auth Mode** (`useFirebase = true, enableAuth = true`): Full Firebase Auth with Apple/Google Sign-In visible in Settings

## Build Commands

```bash
# Build iOS app
xcodebuild -project ios/KoalaStarterKit.xcodeproj -scheme KoalaStarterKit -sdk iphonesimulator build

# Run tests
xcodebuild test -project ios/KoalaStarterKit.xcodeproj -scheme KoalaStarterKit -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 15'
```

Firebase Functions (when using cloud mode):
```bash
cd firebase-functions/functions
npm install
npm run serve    # Local emulators
npm run deploy   # Deploy to Firebase
npm run logs     # View logs
```

## For AI Assistants

When working on this codebase, follow these guidelines:

### Adding New Features

1. **Follow the canonical pattern** - See `Sources/Library/` for the best example
2. **Use consistent folder structure**:
   ```
   Sources/YourFeature/
   ├── Models/      # Codable structs
   ├── ViewModels/  # ObservableObject classes
   └── Views/       # SwiftUI views
   ```
3. **See ARCHITECTURE.md** for detailed patterns and code templates

### Code Patterns

- **ViewModels**: Use `@Observable` macro with plain properties (iOS 17+). Instance-based ViewModels use `@State` in the owning view; singletons are accessed as plain `var` properties.
- **Singletons**: Use `static let shared` pattern for app-wide state
- **Analytics**: Always add `Analytics.trackScreenView()` in view `.task` modifiers
- **Feature flags**: Check `AppConfiguration` before using optional features

### Conditional Firebase

Always guard Firebase code with both compile-time and runtime checks:

```swift
#if canImport(Firebase)
if AppConfiguration.useFirebase {
    // Firebase-specific code
}
#endif
```

### What NOT to Do

- Don't refactor unrelated code when fixing bugs
- Don't add features beyond what was requested
- Don't change the folder structure without explicit approval
- Don't remove the `#if canImport()` guards

## Architecture

### Key Singletons

| Singleton | Purpose | File |
|-----------|---------|------|
| `AuthManager.shared` | Authentication | `Auth/AuthManager.swift` |
| `PaywallManager.shared` | Subscriptions | `Paywall/PaywallManager.swift` |
| `SettingsViewModel.shared` | User settings | `User/SettingsViewModel.swift` |
| `StreakDataProvider.shared` | Streak display | `Streak/ViewModels/StreakViewModel.swift` |
| `FirestoreManager.shared` | Firestore ops | `User/FirestoreManager.swift` |

### Data Flow

**Local Mode**: UserDefaults → ViewModel → View

**Cloud Mode**:
- Writes: View → ViewModel → FirestoreManager → Firestore
- Reads: Firestore → Listener → ViewModel → View

### Library/CMS System

Content is fetched from a GitHub repository:
1. `LibraryViewModel` fetches `index.json` from `AppConfiguration.libraryIndexURL`
2. Articles are filtered by publish/expiry dates
3. Content cached by `LibraryCacheManager`
4. Markdown rendered using MarkdownUI

## Repository Structure

```
koala-starter-kit/
├── ios/
│   ├── KoalaStarterKit.xcodeproj
│   ├── Sources/                    # Main app source
│   │   ├── App/                    # Entry point, config
│   │   ├── Auth/                   # Authentication
│   │   ├── User/                   # Settings, Firestore
│   │   ├── Streak/                 # Streak system
│   │   │   ├── Models/
│   │   │   ├── ViewModels/
│   │   │   └── Views/
│   │   ├── Paywall/                # RevenueCat
│   │   ├── Analytics/              # TelemetryDeck
│   │   ├── Library/                # GitHub CMS (canonical example)
│   │   │   ├── Models/
│   │   │   ├── ViewModels/
│   │   │   └── Views/
│   │   ├── TabBar/                 # Navigation
│   │   ├── Onboarding/             # Onboarding flow
│   │   ├── UI/                     # Theme, components
│   │   ├── Utilities/              # Helpers
│   │   └── Assets.xcassets
│   └── Widget/                     # Widget extension
├── firebase-functions/             # Backend (optional)
├── content/                        # CMS articles
├── ARCHITECTURE.md                 # Detailed patterns
├── CUSTOMIZATION.md                # Branding guide
└── README.md
```

## Configuration

All feature flags and API keys in `ios/Sources/App/AppConfiguration.swift`:

| Flag | Purpose |
|------|---------|
| `useFirebase` | Enable Firebase backend |
| `enableAuth` | Show sign-in UI (requires Firebase) |
| `useRevenueCat` | Enable subscriptions |
| `useTelemetryDeck` | Enable analytics |
| `enableStreaks` | Enable streak feature |
| `enableLibrary` | Enable CMS feature |
| `enableWidgets` | Enable widgets |
| `enableAppReview` | Enable App Store review prompts |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cliffordh/swiftui-indie-stack](https://github.com/cliffordh/swiftui-indie-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
