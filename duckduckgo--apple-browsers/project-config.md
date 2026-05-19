---
trigger: always_on
description: DuckDuckGo.xcworkspace/          # Main workspace (ALWAYS open this)
---


# Project Structure & Organization

## Workspace Structure

### Root Level
```
DuckDuckGo.xcworkspace/          # Main workspace (ALWAYS open this)
├── iOS/                         # iOS app target
├── macOS/                       # macOS app target
├── SharedPackages/              # Cross-platform Swift packages
├── fastlane/                    # CI/CD automation
└── README.md                    # Main project documentation
```

### iOS App Structure
```
iOS/
├── DuckDuckGo/                  # Main iOS app
│   ├── AppDelegate.swift        # App lifecycle
│   ├── MainViewController.swift # Primary browser interface
│   ├── BrowserTab.swift        # Tab state and WebKit integration
│   ├── AIChat/                 # AI chat integration
│   ├── AppLifecycle/           # App lifecycle management
│   ├── Autofill/               # Form autofill features
│   ├── Bookmarks/              # Bookmark management
│   ├── BrowsingMenu/           # Browser menu UI
│   ├── Configuration/          # App configuration
│   ├── DataImport/             # Data import utilities
│   ├── HealthKitReporting/     # Health data reporting
│   ├── MainWindow/             # Main window controllers
│   ├── Subscription/           # Premium features
│   ├── SyncPrompt/             # Sync feature prompts
│   ├── TabSwitcher/            # Tab switching UI
│   └── WebView/                # Web view management
├── Core/                       # iOS-specific shared utilities
├── AutofillCredentialProvider/ # Password autofill extension
├── PacketTunnelProvider/       # VPN network extension
├── OpenAction/                 # Share sheet integration
├── Widgets/                    # Home screen widgets
└── Configuration/              # Build configurations
```

### macOS App Structure
```
macOS/
├── DuckDuckGo/                 # Main macOS app
│   ├── AppDelegate.swift       # App lifecycle
│   ├── MainWindow.swift        # Primary window controller
│   ├── BrowserTabViewController.swift # Web view management
│   ├── AIChat/                 # AI chat integration
│   ├── Autofill/               # Form autofill features
│   ├── Bookmarks/              # Bookmark management UI
│   ├── Downloads/              # Download handling
│   ├── NavigationBar/          # URL bar and navigation
│   ├── NetworkProtection/      # VPN integration
│   ├── Preferences/            # Settings and preferences
│   ├── Subscription/           # Premium features
│   ├── SyncPrompt/             # Sync feature prompts
│   ├── TabBar/                 # Tab management UI
│   └── WebView/                # Web view management
├── DuckDuckGoVPN/              # Standalone VPN app
├── NetworkProtectionSystemExtension/ # System-level VPN
├── DuckDuckGoDBPBackgroundAgent/ # Data Broker Protection
├── DuckDuckGoNotifications/    # System notifications
└── Configuration/              # Build configurations
```

## Dependencies and Packages

### Primary Dependency: BrowserServicesKit
```swift
// ✅ CORRECT - Always use BrowserServicesKit for shared functionality
import BrowserServicesKit

// Features provided by BrowserServicesKit:
// - Content blocking and privacy protection
// - Bookmarks and history management
// - Secure credential storage
// - Autofill functionality
// - Navigation handling
// - User script injection
// - Privacy configuration
// - Sync functionality
```

### Shared Packages
```
SharedPackages/
├── AIChat/                     # AI chat functionality
├── BrowserServicesKit/         # Core browser services
├── DataBrokerProtectionCore/   # Data broker protection
├── DesignResourcesKitIcons/    # Shared icon resources
├── Onboarding/                 # User onboarding experience
├── UIComponents/               # Reusable UI components
└── VPN/                        # VPN functionality
```

### Package Dependencies
```swift
// ✅ CORRECT - Use shared packages for cross-platform features
import DesignResourcesKitIcons
import UIComponents
import BrowserServicesKit

// ❌ INCORRECT - Don't duplicate functionality across platforms
// Keep platform-specific code in iOS/ and macOS/ directories only
```

## Build Configuration

### Xcode Workspace Setup
```swift
// ✅ CORRECT - Always open workspace, not individual projects
// Open: DuckDuckGo.xcworkspace
// Don't open: iOS/DuckDuckGo-iOS.xcodeproj or macOS/DuckDuckGo-macOS.xcodeproj
```

### Build Requirements
```
iOS:
- Xcode 15.0 or later
- Swift 5.9 or later
- iOS 15.0+ deployment target
- Valid Apple Developer account
- Provisioning profiles for extensions

macOS:
- Xcode 15.0 or later
- Swift 5.9 or later
- macOS 11.4+ deployment target
- Developer ID certificate (for notarization)
- System extension entitlements
```

### Configuration Files
```
iOS/Configuration/
├── Configuration.xcconfig         # Base configuration
├── Configuration-Alpha.xcconfig   # Alpha build settings
├── Configuration-Debug.xcconfig   # Debug build settings
└── BuildNumber.xcconfig          # Build number management

macOS/Configuration/
├── Base.xcconfig                 # Base configuration
├── Debug.xcconfig                # Debug build settings
├── Release.xcconfig              # Release build settings
└── AppStore.xcconfig             # App Store specific
```

## Development Setup

### Initial Setup
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
