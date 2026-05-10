---
trigger: always_on
description: Anchor is a minimal check-in app ecosystem using the AT Protocol to post check-ins (called "drops") to Bluesky. The project includes multiple targets sharing a common core, built with Swift 6, using OpenStreetMap via Overpass API for location search and CoreLocation for geolocation.
---

# Anchor Project Structure Guide

## Overview

Anchor is a minimal check-in app ecosystem using the AT Protocol to post check-ins (called "drops") to Bluesky. The project includes multiple targets sharing a common core, built with Swift 6, using OpenStreetMap via Overpass API for location search and CoreLocation for geolocation.

## Project Architecture

The project follows a modular architecture with shared core components and multiple platform-specific targets, using **MV Pattern** (Model-View without ViewModels) for SwiftUI applications:

### Core Components

- **AnchorKit** - Core reusable logic module located in `AnchorKit/`
- **Shared Models & Services** - Common business logic shared across all targets
- **Stores** - `@Observable` classes for business logic operations (no ViewModels)

### Platform Targets

- **AnchorCLI** - Command-line interface executable (macOS)
- **Anchor** - SwiftUI menu bar app (macOS 14.0+)
- **AnchorMobile** - SwiftUI mobile app (iOS 26.0+)

### Key Files

#### Core Module
- [AnchorKit/Package.swift](mdc:Anchor/Anchor/AnchorKit/Package.swift) - Core module package definition
- [AnchorKit/Sources/AnchorKit/AnchorKit.swift](mdc:Anchor/Anchor/AnchorKit/Sources/AnchorKit/AnchorKit.swift) - Core module entry point

#### CLI Target
- [Package.swift](mdc:Anchor/Anchor/Package.swift) - Main package definition with executable target
- [AnchorCLI/Sources/AnchorCLI/main.swift](mdc:Anchor/Anchor/AnchorCLI/Sources/AnchorCLI/main.swift) - CLI entry point
- [AnchorCLI/Sources/AnchorCLI/CLICommands.swift](mdc:Anchor/Anchor/AnchorCLI/Sources/AnchorCLI/CLICommands.swift) - CLI command definitions

#### Menu Bar App Target
- [Anchor.xcodeproj/project.pbxproj](mdc:Anchor/Anchor/Anchor.xcodeproj/project.pbxproj) - Xcode project configuration
- [Anchor/AnchorMenubarApp.swift](mdc:Anchor/Anchor/Anchor/AnchorMenubarApp.swift) - Menu bar app entry point
- [Anchor/ContentView.swift](mdc:Anchor/Anchor/Anchor/ContentView.swift) - Main UI view

#### Mobile App Target
- [AnchorMobile/AnchorMobileApp.swift](mdc:Anchor/Anchor/AnchorMobile/AnchorMobileApp.swift) - Mobile app entry point
- [AnchorMobile/ContentView.swift](mdc:Anchor/Anchor/AnchorMobile/ContentView.swift) - Main mobile UI view

### Directory Structure

```
AnchorKit/Sources/AnchorKit/
├── Models/         ← Data models (Place, AuthCredentials, AnchorSettings)
├── Services/       ← API services (Bluesky, Overpass, CoreLocation)
├── Stores/         ← Minimal @Observable business logic classes
│   ├── AnchorStore.swift      ← Main app store (check-ins, location, auth)
│   └── SettingsStore.swift    ← Settings management (if needed)
├── ATProtocol/     ← AT Protocol client and models
└── Utils/          ← Utility functions and helpers

Anchor/Features/    ← Menu bar app feature modules (feature-based organization)
├── CheckIn/
│   └── Views/      ← SwiftUI views (@Environment(AnchorStore.self))
├── Feed/
│   └── Views/      ← Feed display views (@Query + @Environment(AnchorStore.self))
├── Nearby/
│   └── Views/      ← Nearby places views (@Environment(AnchorStore.self))
└── Settings/
    └── Views/      ← Settings UI (@Environment(SettingsStore.self))

AnchorMobile/Features/ ← Mobile app feature modules (same minimal pattern)
├── CheckIn/Views/
├── Feed/Views/
├── Nearby/Views/
└── Settings/Views/

Docs/               ← Local documentation
├── mv_pattern_guidelines.md      ← MV Pattern architecture guide
├── swift-concurrency.md          ← Swift 6 concurrency guide
├── SwiftData-api-20250627.md     ← SwiftData API reference
└── SwiftUI-api-20250627.md       ← SwiftUI API reference
```

## CLI Commands (MVP Scope)

1. **`anchor login`** - Authenticate with Bluesky using AT Protocol
2. **`anchor settings`** - Configure user preferences (default message)
3. **`anchor drop`** - Main check-in functionality with optional `--place` and `--message` parameters
4. **`anchor nearby`** - List nearby POIs with optional `--filter` parameter

## Key Technologies

### Cross-Platform
- **Swift 6** - Modern Swift with strict concurrency (see [swift-concurrency.md](mdc:Anchor/Anchor/Docs/swift-concurrency.md))
- **SwiftUI** - Declarative UI framework (see [SwiftUI-api-20250627.md](mdc:Anchor/Anchor/Docs/SwiftUI-api-20250627.md))
- **SwiftData** - Data persistence framework (see [SwiftData-api-20250627.md](mdc:Anchor/Anchor/Docs/SwiftData-api-20250627.md))
- **AT Protocol** - Bluesky backend communication
- **CoreLocation** - Location services for all platforms
- **Overpass API** - OpenStreetMap POI queries via `overpass.private.coffee`

### Platform-Specific
- **Swift ArgumentParser** - CLI command parsing (CLI only)
- **UserDefaults** - Local storage for settings and auth tokens

## Architecture Patterns

### MV Pattern (Model-View)
Follow the **MV Pattern** guidelines (see [mv_pattern_guidelines.md](mdc:Anchor/Anchor/Docs/mv_pattern_guidelines.md)):

- **No ViewModels** - Use `@Observable` classes for business logic
- **Views own UI state** - Use `@State` for temporary UI concerns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dropanchorapp/Anchor](https://github.com/dropanchorapp/Anchor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
