---
trigger: always_on
description: This document provides AI assistants with an overview of the eqTrainer codebase, conventions, and development workflows.
---

# CLAUDE.md — eqTrainer Codebase Guide

This document provides AI assistants with an overview of the eqTrainer codebase, conventions, and development workflows.

## Project Overview

**eqTrainer** is a cross-platform Flutter application for ear-training / critical-listening practice. Users listen to audio processed through a parametric EQ, then identify which frequency band was boosted or cut.

- **Version:** 2.3.0+250917
- **Framework:** Flutter (stable, 3.35.3+), Dart ≥ 3.0.0
- **Supported platforms:** Android 7+, iOS 14+, Windows (Vista+), macOS 11+, Linux

---

## Repository Structure

```
eqTrainer/
├── lib/
│   ├── main.dart              # App bootstrap, provider wiring, Hive init
│   ├── theme_data.dart        # Material 3 theme helpers & ReactiveElementData
│   ├── features/              # Feature-first UI modules
│   │   ├── config/            # Session parameter configuration page
│   │   ├── import/            # Audio file import workflow
│   │   ├── playlist/          # Playlist management page
│   │   ├── result/            # Post-session results display
│   │   ├── session/           # Core training session (UI + logic)
│   │   ├── settings/          # App settings page
│   │   └── main_page.dart     # Root navigation/tab controller
│   └── shared/                # Cross-feature code
│       ├── model/             # Data models: AudioClip, AudioState, SettingData
│       ├── player/            # Audio engine: PlayerIsolate, EQ filters
│       ├── repository/        # IAudioClipRepository + Hive implementation
│       ├── service/           # Business logic services
│       └── widget/            # Reusable UI widgets
├── assets/
│   ├── fonts/                 # PretendardVariable.ttf
│   ├── icon/                  # App icon
│   └── translations/          # en.yaml, ko.yaml (easy_localization)
├── android/ ios/ macos/ windows/ linux/  # Platform-specific runners
├── .github/workflows/build.yml           # CI/CD (5-platform builds)
├── pubspec.yaml
├── analysis_options.yaml
├── CONTRIBUTING.md
└── README.md
```

---

## Architecture

### State Management

The app uses **Provider** with `ChangeNotifier` throughout. All providers are registered at the root `MultiProvider` in `main.dart`:

| Provider | Type | Purpose |
|---|---|---|
| `NavBarProvider` | ChangeNotifier | Bottom nav state |
| `AudioState` | ChangeNotifier | Audio backend & output device selection |
| `AppDirectories` | Provider | App support directory paths |
| `AudioClipRepository` | Provider | Hive-backed clip storage |
| `IAudioClipRepository` | Provider | Interface alias for DI flexibility |
| `AudioClipService` | Provider | File import / clip management |
| `PlaylistService` | Provider | Playlist operations & enabled-clip queries |
| `ImportWorkflowService` | Provider | File-picker import flow |
| `SessionParameter` | ChangeNotifier | Session config (band, gain, Q, filter type, threshold) |
| `SessionStore` | ChangeNotifier | Session runtime state & results |
| `SessionController` | Provider | Orchestrates session launch & answer submission |

### Feature Layer (`lib/features/`)

Each feature is a self-contained module with:
- `*_page.dart` — top-level page widget
- `widget/` or `widgets/` — feature-local widgets
- `data/` — local state/data classes (if any)
- `model/` — feature-specific models (if any)
- `index.dart` — barrel export

### Shared Layer (`lib/shared/`)

| Sub-directory | Content |
|---|---|
| `model/` | `AudioClip` (Hive model), `AudioState` (backend/device), `SettingData` (Hive settings) |
| `player/` | `PlayerIsolate` (audio engine in a Dart isolate), `PeakingEqNode`, `PeakingEqFilter` |
| `repository/` | `IAudioClipRepository` interface + `AudioClipRepository` (Hive impl) |
| `service/` | `AudioClipService`, `PlaylistService`, `ImportWorkflowService`, `UpgraderService` |
| `widget/` | `DeviceDropdown`, `InteractionLock`, `MaxWidthCenterBox` |

---

## Key Patterns & Conventions

### Naming

- **Files:** `snake_case.dart`
- **Classes:** `PascalCase`
- **Private members:** `_camelCase` prefix
- **Barrel exports:** every module exposes an `index.dart`

### Audio Engine (`PlayerIsolate`)

Audio runs in a **Dart isolate** via `coast_audio`. Communication uses a sealed class hierarchy:

```dart
sealed class PlayerHostRequest { ... }
class PlayerHostRequestStart extends PlayerHostRequest { ... }
class PlayerHostRequestSetEQ extends PlayerHostRequest { ... }
// etc.
```

Never call audio APIs directly from the UI. Always go through `PlayerIsolate` methods (`launch`, `setEQ`, `setEQFreq`, `setEQGain`, `seek`, etc.).

### Session Flow

```
SessionController.launchSession()
  → playlistService.listEnabledClipPaths()
  → player.launch()
  → sessionStore.initFrequency()     ← FrequencyCalculator.compute()
  → SessionController.initSession()  ← picks random answer freq
  → sessionStore.setSessionState(SessionState.ready)

User submits answer →
SessionController.submitAnswer()
  → sessionStore.applySubmission()   ← updates score & per-band stats
  → adjusts sessionParameter.startingBand if threshold reached
  → SessionController.initSession()  ← next round
```

### Persistence (Hive CE)

- Boxes: `backendBox`, `miscSettingsBox`, `audioClipBox`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [potatosalad775/eqTrainer](https://github.com/potatosalad775/eqTrainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
