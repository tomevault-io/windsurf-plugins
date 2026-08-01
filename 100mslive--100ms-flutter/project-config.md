---
trigger: always_on
description: This file provides comprehensive guidance to Claude Code when working with the 100ms Flutter SDK repository.
---

# CLAUDE.md

This file provides comprehensive guidance to Claude Code when working with the 100ms Flutter SDK repository.

## Project Overview

This is the official **100ms Flutter SDK** repository, providing real-time audio/video conferencing, interactive live streaming, and chat capabilities for Flutter applications. The repository contains two main packages and multiple sample applications demonstrating various implementation patterns.

### Repository Structure

```
100ms-flutter/
├── packages/
│   ├── hmssdk_flutter/          # Core SDK package (v1.11.0)
│   │   ├── android/             # Android platform code (Kotlin)
│   │   ├── ios/                 # iOS platform code (Swift)
│   │   ├── lib/                 # Dart SDK implementation
│   │   └── example/             # Full-featured example app
│   │
│   └── hms_room_kit/            # Prebuilt UI package (v1.2.0)
│       ├── android/             # Android platform integration
│       ├── ios/                 # iOS platform integration
│       ├── lib/                 # Prebuilt UI components
│       └── example/             # Prebuilt UI example
│
└── sample apps/                 # 11 different sample implementations
    ├── bloc/                    # BLoC pattern example
    ├── getx/                    # GetX state management
    ├── mobx/                    # MobX state management
    ├── riverpod/                # Riverpod state management
    ├── flutter-quickstart-app/  # Simple quickstart
    ├── hms-callkit-app/         # CallKit integration
    ├── one_to_one_callkit/      # 1-to-1 calling
    ├── flutter-audio-room-quickstart/
    ├── flutter-hls-quickstart/
    ├── flutterflow-prebuilt-quickstart/
    └── flutter-meet/
```

## Development Environment

### Required Tools & Versions

#### Minimum Configuration
- **Flutter**: 3.24.0 or higher
- **Dart**: 2.16.0 or higher
- **Android**:
  - Android Studio with Android SDK
  - API Level: 24 (Android 7.0) minimum
  - Java: JDK 17 or higher
  - Kotlin: 2.0.21 or higher
  - Android Gradle Plugin (AGP): 8.9.0 or higher
  - Gradle: 8.10 or higher
  - NDK: r28 (27.2.12479018) or higher
- **iOS**:
  - macOS with Xcode 12 or higher
  - iOS 12.0+ deployment target
  - CocoaPods

#### Recommended Configuration
- **Flutter**: 3.27.0 or higher (latest stable)
- **Android API Level**: 35 or 36 (for 16KB page size support)
- **iOS**: 16.0+ deployment target
- **Xcode**: 15 or higher

### Flutter & Dart Setup

The repo uses Flutter's stable channel. Ensure your Flutter installation is up to date:

```bash
flutter --version
flutter doctor -v
```

## Package Details

### 1. hmssdk_flutter (Core SDK)

**Location**: `packages/hmssdk_flutter/`

**Purpose**: Low-level Flutter plugin providing native platform integrations for 100ms video/audio SDK.

**Key Features**:
- Real-time audio/video conferencing
- Interactive live streaming (HLS, RTMP)
- Recording (Server, Browser, HLS)
- Picture-in-Picture (PiP) mode
- CallKit & VoIP support
- Screen sharing (Android & iOS)
- Audio sharing (Android 10+)
- Chat messaging (broadcast, group, direct)
- Role-based permissions
- Session store & metadata
- Network quality monitoring
- Audio output routing (Android)
- Active speaker detection
- Video track management

**Platform Code**:
- **Android**: `android/src/main/kotlin/` - Kotlin implementation using 100ms Android SDK
- **iOS**: `ios/Classes/` - Swift implementation using 100ms iOS SDK
- **Dart**: `lib/` - Platform channel interfaces and Dart models

**Native Dependencies**:
- Android: `live.100ms:android-sdk`, `live.100ms:video-view`, `live.100ms:hls-player`
- iOS: 100ms iOS SDK via CocoaPods

**Important Files**:
- `lib/assets/sdk-versions.json` - Native SDK version configuration
- `pubspec.yaml` - Flutter package configuration
- `android/build.gradle` - Android build configuration
- `ios/hmssdk_flutter.podspec` - iOS CocoaPods spec

### 2. hms_room_kit (Prebuilt UI)

**Location**: `packages/hms_room_kit/`

**Purpose**: High-level prebuilt UI components for quick integration of video conferencing features.

**Key Features**:
- Ready-to-use video conferencing UI
- Customizable themes and layouts
- Built-in chat interface
- Participant list management
- Screen sharing UI
- Audio/video controls
- Role-based UI elements
- Settings and permissions UI

**Dependencies**:
- Uses `hmssdk_flutter` as core dependency
- Additional UI packages: `provider`, `google_fonts`, `flutter_svg`, `lottie`, etc.

**Integration**: Single widget implementation
```dart
HMSPrebuilt(
  roomCode: "your-room-code",
  hmsConfig: HMSPrebuiltOptions(userName: "User Name")
)
```

### 3. Example App

**Location**: `packages/hmssdk_flutter/example/`

**Purpose**: Comprehensive reference implementation demonstrating all SDK features.

**State Management**: Provider pattern

**Architecture**:
- `PreviewStore` - Manages preview screen state (implements `HMSPreviewListener`)
- `MeetingStore` - Manages meeting room state (implements `HMSUpdateListener`, `HMSActionResultListener`)
- `PeerTrackNode` - Per-peer state management to optimize UI updates

**Key Files**:
- `lib/main.dart` - App entry point
- `lib/data_store/meeting_store.dart` - Main meeting state
- `lib/data_store/preview_store.dart` - Preview state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [100mslive/100ms-flutter](https://github.com/100mslive/100ms-flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
