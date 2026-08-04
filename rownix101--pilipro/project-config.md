---
trigger: always_on
description: **PiliPro** (also referred to as PiliPlus) is a high-performance third-party BiliBili client developed with Flutter.
---

# PiliPro - Agentic Coding Guide

## Project Overview

**PiliPro** (also referred to as PiliPlus) is a high-performance third-party BiliBili client developed with Flutter.

- **Project Type**: Flutter mobile application
- **Supported Platforms**: 
  - ✅ Android 10+ (actively maintained)
  - ✅ iOS 17+ (actively maintained)
  - ❌ Windows, Linux, macOS (DEPRECATED - no longer maintained)
- **Current Version**: 1.1.6
- **Flutter Version**: 3.41.0 (managed via FVM - see `.fvmrc`)
- **Dart SDK**: >=3.10.0

### Technology Stack

| Component | Technology |
|-----------|------------|
| State Management | GetX |
| Networking | Dio with HTTP/2 adapter (`dio_http2_adapter`) |
| Local Storage | Hive |
| Image Caching | `cached_network_image` |
| Video Player | Custom native plugin (ExoPlayer on Android, AVPlayer on iOS) |
| gRPC/Protobuf | `protobuf` (generated files in `lib/grpc/`) |
| WebView | `flutter_inappwebview` |
| Dynamic Theming | `dynamic_color` (Material 3) |
| Logging | `logger` + `catcher_2` for error reporting |

---

## Build Commands

> **Note**: This project uses **FVM** (Flutter Version Management). Use `fvm flutter` instead of `flutter` if FVM is installed.

### Development

```bash
# Install dependencies
flutter pub get

# Run app in debug mode
flutter run

# Analyze code
flutter analyze

# Format code (preserves trailing commas per analysis_options.yaml)
dart format .

# Generate code (if using build_runner)
flutter pub run build_runner build --delete-conflicting-outputs
```

### Release Builds

```bash
# Build Android APK (split per ABI)
flutter build apk --release --split-per-abi

# Build Android APK (single)
flutter build apk --release

# Build iOS
flutter build ios --release

# Build with release configuration (used in CI)
flutter build apk --release --split-per-abi --dart-define-from-file=pili_release.json
```

### Build Configuration

The project uses a PowerShell script (`lib/scripts/build.ps1`) to set build-time environment variables:
- `pili.name` - Version name
- `pili.code` - Version code (git commit count)
- `pili.hash` - Git commit hash
- `pili.time` - Build timestamp

These are accessed via `BuildConfig` class in `lib/build_config.dart`.

### Configuration Files

The project supports external configuration files for build-time variables (API keys, etc.).

**There are two types of configuration files:**

#### 1. Version Info (Auto-generated)
The `lib/scripts/build.ps1` script automatically generates:
- `pili_release.json` - Contains version name, code, hash, and build timestamp

**Do not manually edit this file** - it's regenerated on each build.

#### 2. API Keys (User-created)
Create your own configuration file for sensitive API credentials:

**Setup:**
```bash
# 1. Copy example config
cp pili_config_example.json pili_release_config.json

# 2. Edit pili_release_config.json with your API keys
{
  "BILI_APP_KEY": "your_app_key_here",
  "BILI_APP_SECRET": "your_app_secret_here"
}
```

**Automatic Injection (via build.ps1):**
When using the PowerShell build script, API keys are automatically merged:
```bash
# Script reads pili_release_config.json and merges with version info
# into pili_release.json
.\lib\scripts\build.ps1
```

**Manual Build:**
```bash
# You can also inject directly
dart-define BILI_APP_KEY=your_key BILI_APP_SECRET=your_secret

# Or use a config file
flutter build apk --release --dart-define-from-file=pili_release_config.json
```

**Important:** Configuration files are excluded from git (see `.gitignore`):
- `pili_release.json` - Auto-generated, contains version info
- `pili_release_config.json` - User API keys
- `*.env` - Environment files

These files may contain sensitive information and should NOT be committed to version control.

---

## Project Structure

```
lib/
├── build_config.dart          # Build-time configuration constants
├── main.dart                  # App entry point, theme setup, initialization
├── common/                    # Shared widgets, constants, skeleton screens, animations
│   ├── constants.dart         # App constants, API keys, style values
│   ├── skeleton/              # Loading skeleton widgets
│   └── widgets/               # Reusable UI components
│       ├── animation/         # Custom animations
│       ├── appbar/            # AppBar components
│       ├── button/            # Button widgets
│       ├── dialog/            # Dialog components
│       ├── effects/           # Visual effects
│       ├── flutter/           # Custom Flutter widget overrides
│       ├── gesture/           # Gesture recognizers
│       ├── image/             # Image widgets (NetworkImgLayer)
│       ├── image_viewer/      # Image viewing components
│       ├── interactions/      # Interactive widgets
│       ├── loading/           # Loading indicators
│       ├── player/            # Player-related widgets
│       ├── progress_bar/      # Progress indicators
│       ├── refresh/           # Pull-to-refresh
│       ├── scroll/            # Scroll behaviors
│       ├── skeleton/          # Skeleton screens
│       ├── stat/              # Statistics displays
│       ├── transition/        # Page transitions
│       └── video_card/        # Video card widgets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rownix101/PiliPro](https://github.com/rownix101/PiliPro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
