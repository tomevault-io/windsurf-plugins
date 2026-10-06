---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LockView is a fully functional Android application that provides a lockable image viewer with advanced gesture controls, theme customization, and automatic device-unlock integration. The app uses modern Android architecture with Jetpack Compose and implements comprehensive zoom, pan, unlock functionality, and persistent user preferences.

**Key Details:**
- Language: Kotlin
- UI Framework: Jetpack Compose with Material3
- Architecture: MVVM with StateFlow + Repository Pattern
- Build System: Gradle with Kotlin DSL
- Package Name: `com.zac15987.lockview`
- Min SDK: 24 (Android 7.0)
- Target SDK: 36 (Android 15)

## Development Commands

### Building
```bash
cmd.exe //c ".\gradlew.bat clean build"          # Clean and build everything
cmd.exe //c ".\gradlew.bat assembleDebug"        # Build debug APK
cmd.exe //c ".\gradlew.bat assembleRelease"      # Build release APK
cmd.exe //c ".\gradlew.bat installDebug"         # Build and install debug APK on connected device
cmd.exe //c ".\gradlew.bat bundle"               # Assemble bundles for all variants
```

### Testing
```bash
cmd.exe //c ".\gradlew.bat test"                 # Run unit tests for all variants
cmd.exe //c ".\gradlew.bat testDebugUnitTest"    # Run unit tests for debug build
cmd.exe //c ".\gradlew.bat connectedAndroidTest" # Run instrumentation tests on connected devices
cmd.exe //c ".\gradlew.bat connectedDebugAndroidTest" # Run debug instrumentation tests
cmd.exe //c ".\gradlew.bat check"               # Run all verification tasks
```

### Code Quality
```bash
cmd.exe //c ".\gradlew.bat lint"                # Run lint on the default variant
cmd.exe //c ".\gradlew.bat lintDebug"          # Run lint for debug variant
cmd.exe //c ".\gradlew.bat lintRelease"        # Run lint for release variant
cmd.exe //c ".\gradlew.bat lintFix"            # Run lint and apply safe suggestions
cmd.exe //c ".\gradlew.bat updateLintBaseline"  # Update lint baseline
```

### Other Useful Commands
```bash
cmd.exe //c ".\gradlew.bat tasks"              # List main tasks
cmd.exe //c ".\gradlew.bat tasks --all"        # List all available tasks
cmd.exe //c ".\gradlew.bat dependencies"       # Show project dependencies
cmd.exe //c ".\gradlew.bat signingReport"      # Display signing info
cmd.exe //c ".\gradlew.bat uninstallDebug"     # Uninstall debug build from device
```

**Note:** use cmd.exe //c ".\gradlew.bat"

## Architecture Overview

### MVVM Architecture with Compose
The app follows modern Android architecture patterns:
- **State Management**: `ImageViewerViewModel` with `StateFlow`
- **UI Layer**: Declarative Compose UI with state-driven updates
- **Data Layer**: `ImageViewerState` data class for centralized state
- **Gesture Handling**: Custom Compose modifiers and gesture detectors

### Key Dependencies
- **Jetpack Compose**: UI framework with Material3
- **Coil Compose**: Image loading and caching
- **AndroidX Lifecycle**: ViewModel and state management
- **DataStore Preferences**: Persistent user preferences (theme and language settings)
- **Material3 Dynamic Color**: Adaptive theming for Android 12+

### Package Structure
```
com.zac15987.lockview/
├── LockViewApplication.kt          # Custom Application class for locale initialization
├── MainActivity.kt                 # Main activity with device unlock integration
├── data/
│   ├── ImageViewerState.kt         # Image viewer state data model
│   ├── Bounds.kt                   # Boundary calculation utilities
│   ├── language/
│   │   ├── LanguagePreference.kt   # Language preference enum
│   │   └── LanguageRepository.kt   # Language persistence with DataStore + SharedPreferences
│   └── theme/
│       ├── ThemePreference.kt      # Theme preference enum
│       └── ThemeRepository.kt      # Theme persistence with DataStore
├── ui/
│   ├── components/
│   │   ├── ZoomableImage.kt        # Zoomable image with gesture handling
│   │   ├── TransformGestureDetector.kt # Custom gesture detection
│   │   └── LicensesDialog.kt       # Open source licenses and about dialog
│   ├── screens/
│   │   └── ImageViewerScreen.kt    # Main UI screen with theme and language pickers
│   └── theme/
│       ├── Theme.kt                # Material3 theme with dynamic theming
│       ├── Color.kt                # Color palette definitions
│       ├── Type.kt                 # Typography definitions
│       └── LocaleProvider.kt       # Compose context provider for immediate language switching
├── utils/
│   ├── LocaleHelper.kt             # Locale configuration and system language reset
│   └── ShareIntentHelper.kt        # Extracts the image URI from an incoming ACTION_SEND intent
└── viewmodel/
    ├── ImageViewerViewModel.kt     # Image state and business logic
    ├── SettingsViewModel.kt        # Combined theme and language state management
    ├── SettingsViewModelFactory.kt # Factory for SettingsViewModel dependency injection
    ├── ThemeViewModel.kt           # Legacy theme state management (still used)
    └── ThemeViewModelFactory.kt    # Legacy theme ViewModel factory
```

## Core Features

### Image Handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zac15987/LockView](https://github.com/zac15987/LockView) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
