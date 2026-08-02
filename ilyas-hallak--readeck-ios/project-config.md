---
trigger: always_on
description: **readeck iOS** is a native iOS client for [readeck](https://readeck.org) bookmark management. The app provides a clean, native iOS interface for managing bookmarks with features like swipe actions, search, tagging, and reading progress tracking.
---

# CLAUDE.md - readeck iOS Project Documentation

## Project Overview

**readeck iOS** is a native iOS client for [readeck](https://readeck.org) bookmark management. The app provides a clean, native iOS interface for managing bookmarks with features like swipe actions, search, tagging, and reading progress tracking.

### Key Information
- **Platform:** iOS (iPhone + iPad)
- **Language:** Swift
- **UI Framework:** SwiftUI
- **Architecture:** MVVM + Clean Architecture (3-layer: UI/Domain/Data)
- **Database:** CoreData
- **Dependencies:** Swift Package Manager
- **License:** MIT

## Architecture Summary

The project follows Clean Architecture with custom dependency injection:

```
UI Layer (SwiftUI Views + ViewModels)
    ↓
Domain Layer (Use Cases + Repository Protocols + Models)  
    ↓
Data Layer (Repository Implementations + API + CoreData)
```

### Core Components
- **Custom DI:** Protocol-based factory pattern (no external frameworks)
- **MVVM Pattern:** ViewModels handle business logic, Views handle presentation
- **Use Cases:** Single-responsibility business logic encapsulation
- **Repository Pattern:** Data access abstraction with protocols

## Project Structure

```
readeck/
├── UI/                           # SwiftUI Views & ViewModels
│   ├── Bookmarks/               # Main bookmark list
│   ├── BookmarkDetail/          # Article reader
│   ├── AddBookmark/             # Create new bookmarks
│   ├── Search/                  # Search functionality
│   ├── Settings/                # App configuration
│   ├── Labels/                  # Tag management
│   ├── Menu/                    # Navigation & tabs
│   ├── SpeechPlayer/            # Text-to-speech
│   └── Components/              # Reusable UI components
├── Domain/
│   ├── Model/                   # Core business models
│   ├── UseCase/                 # Business logic
│   ├── Protocols/               # Repository interfaces
│   └── Error/                   # Custom error types
├── Data/
│   ├── API/                     # Network layer & DTOs
│   ├── Repository/              # Data access implementations
│   ├── CoreData/                # Local database
│   └── Utils/                   # Helper utilities
└── Localizations/               # i18n strings
    ├── Base.lproj/
    ├── en.lproj/
    └── de.lproj/
```

## Key Features

### Implemented Features
- ✅ Browse bookmarks (All, Unread, Favorites, Archive by type)
- ✅ Share Extension for adding URLs from Safari/other apps
- ✅ Swipe actions for quick bookmark management
- ✅ Native iOS design with Dark Mode support
- ✅ Full iPad Support with Multi-Column Split View
- ✅ Font customization in reader
- ✅ Article view with reading time and word count
- ✅ Search functionality
- ✅ Tag/label management
- ✅ Reading progress tracking
- ✅ Offline support with auto-sync when reconnected
- ✅ Text-to-speech (Read Aloud feature)

### Planned Features (v1.1.0)
- ⏳ Bookmark filtering and sorting options
- ⏳ Collection management
- ⏳ Custom themes
- ⏳ Text highlighting in articles
- ⏳ Multiple selection for bulk actions

## Development Setup

### Requirements
- Xcode 15.0+
- iOS 17.0+ deployment target
- Swift Package Manager (dependencies auto-resolved)

### Key Dependencies
- **netfox:** Network debugging (debug builds only)
- **RswiftLibrary:** Resource management

### Build Configurations
- **Debug:** Includes netfox for network debugging
- **Release:** Production-ready build
- **URLShare Extension:** Share extension target

## Localization (Weblate Integration)

### Current Setup
The project has been converted from Apple's String Catalog (.xcstrings) to traditional .strings format for Weblate compatibility:

```
readeck/Localizations/
├── Base.lproj/Localizable.strings    # Source language (English)
├── en.lproj/Localizable.strings      # English localization
└── de.lproj/Localizable.strings      # German localization
```

### Weblate Configuration
When setting up Weblate:
- **File mask:** `readeck/Localizations/*.lproj/Localizable.strings`
- **Monolingual base:** `readeck/Localizations/Base.lproj/Localizable.strings`  
- **File format:** "iOS Strings (UTF-8)"
- **Repository:** Connect to main Git repository

### Adding New Languages
1. Create new `.lproj` directory (e.g., `fr.lproj/`)
2. Copy `Base.lproj/Localizable.strings` to new directory
3. Weblate will automatically detect and manage translations

## App State Management & Navigation

### Setup Flow & Authentication
The app uses a sophisticated setup and authentication system:

**Initial Setup Detection:**
- `AppViewModel.hasFinishedSetup` controls the main app flow
- `readeckApp.swift:19` determines whether to show setup or main app
- Setup status is persisted via `SettingsRepository.hasFinishedSetup`

**Authentication & Keychain Management:**
- `KeychainHelper` (singleton) securely stores sensitive credentials:
  - Server endpoint (`readeck_endpoint`)
  - Username (`readeck_username`) 
  - Password (`readeck_password`)
  - Authentication token (`readeck_token`)
- Access Group: `8J69P655GN.de.ilyashallak.readeck` for app group sharing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ilyas-hallak/readeck-ios](https://github.com/ilyas-hallak/readeck-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
