---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CloudEdge4Tasker is an Android plugin app that integrates CloudEdge/Meari battery cameras with automation platforms like Tasker and MacroDroid. The app enables camera control and alarm management through Tasker actions and events.

## Build System & Commands

### Building the Project
```bash
./gradlew build                    # Build the entire project
./gradlew assembleDebug           # Build debug APK
./gradlew assembleRelease         # Build release APK
./gradlew clean                   # Clean build artifacts
```

### Testing
```bash
./gradlew test                    # Run unit tests
./gradlew connectedAndroidTest    # Run instrumented tests
```

### Installation
```bash
./gradlew installDebug            # Install debug APK to connected device
./gradlew installRelease          # Install release APK to connected device
```

### Build Configuration
- Gradle Plugin: 8.2.2
- Kotlin: 1.9.22
- compileSdkVersion: 34 (Android 14)
- minSdkVersion: 24 (Android 7.0)
- targetSdkVersion: 33 (Android 13)
- Architecture: armeabi-v7a, arm64-v8a
- viewBinding: enabled

## Architecture Overview

### Package Structure

```
online.avogadro.mearitaskerplugin/
├── MainActivity.java              # Minimal placeholder
├── CommonUtils.java               # Video stream ID helper
├── SplashActivity.java            # Entry point, permissions, auto-login
├── app/
│   ├── MeariApplication.java     # SDK init (partnerId=8)
│   ├── MyFirebaseMessagingService.java  # FCM push notifications
│   ├── MyMessageHandler.java     # MQTT message handling
│   ├── SharedPreferencesHelper.java    # AES-encrypted credential storage
│   └── Util.java                 # Metadata and date utilities
├── device/
│   ├── CamManager.java           # Core singleton for all camera ops
│   ├── DeviceListActivity.java   # Main screen with camera list
│   ├── DeviceListAdapter.java    # RecyclerView adapter with per-camera controls
│   ├── DeviceMonitorActivity.java # Live preview/playback
│   ├── DeviceSettingActivity.java # Per-camera settings
│   ├── DeviceCloudPlayActivity.java # Cloud storage playback
│   ├── AddDeviceActivity.java    # QR code onboarding
│   ├── SettingsActivity.java     # App preferences
│   └── TrafficManagerActivity.java # Traffic/data management
├── tasker/
│   ├── CameraResolver.kt             # Selector→camera list resolution (glob, ID, name)
│   ├── TriggerCameraLightActionHelper.kt  # Turn on light + AbstractCameraActionConfig base class + HelperHolder interface
│   ├── BasicActionHelper.kt      # Enable PIR detection
│   ├── ActivityConfigDisableAlarms.kt # Disable PIR detection
│   ├── EnableSirenActionHelper.kt    # Enable siren on all cameras
│   ├── DisableSirenActionHelper.kt   # Disable siren on all cameras
│   ├── DownloadLastCameraImageActionHelper.kt # Download alert image
│   ├── TakePictureActionHelper.kt    # Take live snapshot
│   ├── TriggerCameraSirenActionHelper.kt  # Fire siren on camera(s)
│   ├── DownloadLastCameraImageInput.kt    # Input: cameraID, cameraName
│   ├── DownloadLastCameraImageOutput.kt   # Output: image file path
│   └── events/
│       ├── ActivityConfigCameraAlarmEvent.kt  # Event config UI
│       ├── CameraAlarmRaiser.kt       # Triggers Tasker events
│       ├── CameraAlarmInfo.java       # Event data: deviceName, deviceID
│       └── AnyNotificationReceiver.java # Legacy C2DM receiver
├── user/
│   ├── LoginActivity.java        # Login screen
│   ├── RegisterActivity.java     # Account registration
│   ├── CloudStatusActivity.java  # Cloud subscription status
│   └── BuyCloudServiceActivity.java # Cloud purchase
├── bean/                          # Data models for traffic management
│   ├── TrafficNumberBean.java
│   ├── TrafficOrderBean.java
│   └── TrafficPacketBean.java
└── alipay/                        # Unused payment utilities
    ├── Base64.java
    ├── SignUtils.java
    └── PayResult.java
```

### Core Components

#### 1. CamManager (`device/CamManager.java`)
Central singleton (`CamManager.get(context)`) for all camera operations.

**Initialization & Auth:**
- `loginAndInitList(IDoSomething)` - Login with stored credentials, fetch device list
- Device list is cached with 120-second refresh interval

**Bulk operations** (operate on a list of cameras, with optional `ICameraOperationCallback`):
- `enableAllCameras(List<CameraInfo>)` - Enable PIR detection
- `disableAllCameras(List<CameraInfo>)` - Disable PIR detection
- `enableAllCameraAlarms(List<CameraInfo>)` - Enable siren alarm
- `disableAllCameraAlarms(List<CameraInfo>)` - Disable siren alarm
- `fireAllSirenAlarms(List<CameraInfo>)` - Fire sirens

**Selector-based operations** (login + resolve selector via `CameraResolver` + action):
- `enableCamerasPIR(selector, callback)` - Enable PIR detection
- `disableCamerasPIR(selector, callback)` - Disable PIR detection
- `fireSirenOnCameras(selector, callback)` - Fire siren (10s wake-up)
- `turnOnLightOnCameras(selector, callback)` - Turn on light (10s wake-up)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SimoneAvogadro/CloudEdge4Tasker](https://github.com/SimoneAvogadro/CloudEdge4Tasker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
