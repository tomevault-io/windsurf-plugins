---
trigger: always_on
description: The Salesforce Mobile SDK Templates repository is the **template library** for creating Salesforce mobile applications. It contains ready-to-use app templates that serve as starting points for iOS, Android, hybrid (Cordova), and React Native mobile applications.
---

# CLAUDE.md — Salesforce Mobile SDK Templates

---

## About This Project

The Salesforce Mobile SDK Templates repository is the **template library** for creating Salesforce mobile applications. It contains ready-to-use app templates that serve as starting points for iOS, Android, hybrid (Cordova), and React Native mobile applications.

**Key constraint**: These templates are **consumed by CLI tools** (`forceios`, `forcedroid`, `forcehybrid`, `forcereact`) and the SFDX plugin. Changes here affect every new app created with the Mobile SDK.

## Repository Purpose

This repository provides:

1. **Template Library** - Collection of app templates for all supported platforms and app types
2. **Template Metadata** - `templates.json` defining all available templates
3. **Template Scripts** - `install.js` and `template.js` for dependency management and customization
4. **Testing Infrastructure** - `test_template.sh` for validating template structure and buildability

## Repository Structure

```
SalesforceMobileSDK-Templates/
├── templates.json                         # Template registry (defines all templates)
├── test_template.sh                       # Template testing script
├── TESTING.md                             # Testing documentation
├── setversion.sh                          # Version update script
├── .claude/
│   └── skills/                            # Claude Code agent skills (see below)
│       └── README.md                      # Skills documentation
│
├── iOSNativeSwiftTemplate/                # Swift iOS template (most common)
│   ├── package.json                       # SDK dependencies
│   ├── install.js                         # Downloads SDK dependencies
│   ├── template.js                        # Customizes template for user
│   ├── Podfile                            # CocoaPods dependencies
│   └── <template-files>                   # Xcode project and source code
│
├── iOSNativeSwiftPackageManagerTemplate/  # Swift with SPM
├── iOSNativeSwiftEncryptedNotificationTemplate/  # Swift with notifications
├── iOSNativeLoginTemplate/                # Swift native login example
├── iOSIDPTemplate/                        # Swift Identity Provider
├── MobileSyncExplorerSwift/               # Swift MobileSync sample app
│
├── AndroidNativeKotlinTemplate/           # Kotlin Android template (most common)
│   ├── package.json                       # SDK dependencies
│   ├── install.js                         # Downloads SDK dependencies
│   ├── template.js                        # Customizes template for user
│   └── <template-files>                   # Gradle project and source code
│
├── AndroidNativeLoginTemplate/            # Kotlin native login example
├── AndroidIDPTemplate/                    # Kotlin Identity Provider
├── MobileSyncExplorerKotlinTemplate/      # Kotlin MobileSync sample app
│
├── ReactNativeTemplate/                   # React Native JavaScript template
│   ├── package.json                       # SDK and npm dependencies
│   ├── installios.js                      # iOS SDK setup
│   ├── installandroid.js                  # Android SDK setup
│   ├── template.js                        # Customizes for both platforms
│   ├── ios/                               # iOS project structure
│   └── android/                           # Android project structure
│
├── ReactNativeTypeScriptTemplate/         # React Native TypeScript template
├── ReactNativeDeferredTemplate/           # React Native deferred login
├── MobileSyncExplorerReactNative/         # React Native MobileSync sample app
│
├── HybridLocalTemplate/                   # Cordova local hybrid template
│   └── <hybrid-app-files>                 # HTML/JS/CSS app
│
└── HybridRemoteTemplate/                  # Cordova remote hybrid template
    └── <hybrid-app-files>                 # Visualforce/Communities app
```

## Template Registry (templates.json)

The `templates.json` file is the **single source of truth** for all available templates.

### Structure

```json
[
    {
        "path": "iOSNativeSwiftTemplate",
        "description": "Swift application using MobileSync, SwiftUI and Combine",
        "appType": "native_swift",
        "platforms": ["ios"]
    }
]
```

### Fields

- **path**: Directory name (must match actual directory)
- **description**: Human-readable description (shown in `listtemplates` output)
- **appType**: One of: `native`, `native_swift`, `native_kotlin`, `hybrid_local`, `hybrid_remote`, `react_native`
- **platforms**: Array of `"ios"`, `"android"`, or both

### App Types

| App Type | Description | Platforms |
|----------|-------------|-----------|
| **native** | Objective-C (iOS) or Java (Android) | Single platform |
| **native_swift** | Swift (iOS) | iOS only |
| **native_kotlin** | Kotlin (Android) | Android only |
| **hybrid_local** | Cordova app with local HTML/JS | iOS and/or Android |
| **hybrid_remote** | Cordova app loading Visualforce/Communities | iOS and/or Android |
| **react_native** | React Native app (JavaScript or TypeScript) | iOS and/or Android |

## Template Anatomy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forcedotcom/SalesforceMobileSDK-Templates](https://github.com/forcedotcom/SalesforceMobileSDK-Templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
