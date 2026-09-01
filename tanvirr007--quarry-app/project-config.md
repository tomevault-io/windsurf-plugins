---
trigger: always_on
description: Welcome to **Quarry** (`app.quarry.tanvir.info`). This document outlines repository structure, architectural patterns, coding guidelines, and workflow conventions for developers and AI coding agents.
---

# AGENTS.md — Developer & AI Agent Guidelines for Quarry

Welcome to **Quarry** (`app.quarry.tanvir.info`). This document outlines repository structure, architectural patterns, coding guidelines, and workflow conventions for developers and AI coding agents.

---

## 1. Project Overview & Philosophy

**Quarry** is a modern, privacy-first Android storage analyzer and visual disk cleanup utility built with 100% Jetpack Compose and Material 3.

- **100% Offline & Private**: All storage scanning, size calculation, and file operations occur entirely on-device. No telemetry, no external network uploads.
- **Modern Android Native**: Targets Android 12.0+ (API 31+) through Android 16 (API 36), leveraging modern platform APIs, Coroutines, Flow, and Material You theming.
- **Visual Storage Breakdown**: Hardware-accelerated squarified treemaps with guaranteed minimum tile visibility, list explorers, and duplicate/large file cleaners.

---

## 2. Technical Stack & Dependencies

- **Language & Runtime**: Kotlin `2.2.21`, Java `21`, Gradle Kotlin DSL (`build.gradle.kts`)
- **Android Target**: `compileSdk = 36`, `targetSdk = 36`, `minSdk = 31`
- **UI Framework**: Jetpack Compose (BOM `2026.06.01`), Material 3, Compose Navigation `2.8.5`
- **Typography**: Google Sans Rounded embedded font family (`res/font/google_sans_rounded.ttf`)
- **Local Persistence**:
  - **Room Database** `2.8.4` (with KSP `2.2.21-2.0.5`) for file metadata indexing and cache.
  - **DataStore Preferences** `1.1.1` for user settings (theme, sort preferences, onboarding flags, scan exclusions).
- **Background Processing**: Jetpack **WorkManager** `2.10.0` + Kotlinx Coroutines `1.9.0`
- **Security**: AndroidX **Biometric** `1.2.0-alpha05` for biometric / PIN protection.
- **CI/CD**: GitHub Actions with live Telegram bot build monitoring & release dispatch.

---

## 3. Architecture & Codebase Structure

Quarry follows **Clean Architecture** and **Unidirectional Data Flow (UDF)**:

```
app/src/main/java/app/quarry/tanvir/info/
├── MainActivity.kt               # Single-activity container, theme/onboarding root
├── MainViewModel.kt              # App-wide UI state and global preferences holder
├── QuarryApp.kt                  # Application class
├── data/                         # Repositories, Room DAOs, DataStore, file scanners
│   ├── database/                 # Room database, entities, DAOs (QuarryDatabase, FileDao, FileEntity)
│   ├── model/                    # Data models, category enums, file items
│   ├── preferences/              # DataStore user preferences, theme, haptics, keep-screen-on, category visibility
│   └── repository/               # Repository implementations
├── domain/                       # Use cases and domain business logic
│   ├── analyzer/                 # Deep storage analyzer and directory breakdown
│   ├── app/                      # Installed applications and package space analysis
│   ├── cleanup/                  # Cleanup rules, category calculation & strategies
│   ├── duplicates/               # Fast hash calculation & duplicate detection engine
│   ├── file/                     # File operations (trash, delete, rename, batch actions)
│   ├── haptics/                  # Centralized vibration helper (QuarryHaptics, strength mapping, VibratorManager)
│   ├── media/                    # Native high-performance ThumbnailLoader with LRU cache
│   ├── model/                    # Domain models (StorageCategory, StorageFormatter)
│   ├── report/                   # Storage breakdown reporting & summary generation
│   ├── scanner/                  # Local storage scanners & metadata indexing
│   ├── security/                 # Biometric authentication management
│   ├── treemap/                  # Treemap layout engine (squarified algorithm, node hierarchy)
│   └── volume/                   # Storage volume manager (internal & external volumes)
├── ui/                           # Jetpack Compose UI layer
│   ├── cleanup/                  # Storage cleaner & duplicate/large file screens
│   ├── components/               # Reusable UI widgets (cards, charts, buttons, dialogs, thumbnails)
│   ├── explore/                  # Treemap canvas, file lists, category explorer, breadcrumb navigation
│   ├── home/                     # Dashboard, storage breakdown, visual category charts (Quick Insights gating, filtered categories)
│   ├── navigation/               # NavHost, screens, bottom navigation bar
│   ├── onboarding/               # First-run permission & onboarding dialogs
│   ├── settings/                 # Settings + dialogs (Appearance, Exclusions, Storage Volumes, CategoryVisibilityDialog, MiscellaneousDialog)
│   └── theme/                    # Material 3 ColorScheme, Typography, Theme setup
└── worker/                       # WorkManager workers (background scanning/cleanup)
```

---

## 4. Key Architectural Patterns

### Treemap Engine & Visualization
- **Squarified Treemap Algorithm**: `TreemapEngine` organizes directory hierarchies into squarified tiles maintaining optimal aspect ratios.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanvirr007/quarry-app](https://github.com/tanvirr007/quarry-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
