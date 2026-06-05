---
trigger: always_on
description: A minimal Android app for building on discontinued Meta Portal touch and TV devices. Demonstrates UI controls, runtime permissions, camera preview, and audio recording/playback — preconfigured for Portal hardware constraints.
---

# Portal Sample App

A minimal Android app for building on discontinued Meta Portal touch and TV devices. Demonstrates UI controls, runtime permissions, camera preview, and audio recording/playback — preconfigured for Portal hardware constraints.

## Quick Start: Portal Development Skill

The easiest way to work with this project is to load the **portal** skill from [hzdb](https://github.com/meta-quest/agentic-tools). It covers Portal hardware constraints, design requirements, toolchain setup, the build/deploy loop, and debugging — all in one self-contained skill:

```bash
npx -y @meta-quest/hzdb --version          # Install hzdb (requires Node.js 20+)
hzdb mcp install <your-tool>               # Connect to your AI coding tool
```

Load it with `/read-skill portal` or by mentioning Portal when hzdb's MCP server is connected.

If you prefer to work without the skill, the rest of this file contains everything you need.

## Project Structure

```
PortalSampleApp/
  app/
    src/main/
      java/com/meta/portal/sampleapp/
        MainActivity.kt          # Single-activity Compose app with scrollable showcase
        UiElementsSection.kt     # Material 3 controls: buttons, text field, slider, checkbox, switch, radio, dropdown, card
        PermissionsSection.kt    # Runtime permission requests with granted/denied status indicators
        CameraSection.kt        # CameraX live preview with front/back camera selection
        AudioRecorderSection.kt  # Audio recording (MediaRecorder) and playback (MediaPlayer)
        ui/theme/                # Compose theme (Color.kt, Theme.kt, Type.kt)
      res/
        values/strings.xml       # All user-facing strings
        drawable/                # Launcher icon vectors
      AndroidManifest.xml        # Permissions: CAMERA, LOCATION, CONTACTS, RECORD_AUDIO
    build.gradle.kts             # App module: dependencies, SDK config
  gradle/libs.versions.toml      # Version catalog (Compose, CameraX, etc.)
  build.gradle.kts               # Root build file
  settings.gradle.kts            # Repository and module config
```

## Build & Deploy

Prerequisites: JDK 17, Android Studio, Portal with ADB enabled and USB-C connected.

```bash
./gradlew assembleDebug                                    # Build debug APK
adb install app/build/outputs/apk/debug/app-debug.apk     # Install on Portal
adb shell am start -n com.meta.portal.sampleapp/.MainActivity  # Launch
```

If hzdb is installed:
```bash
hzdb app install app/build/outputs/apk/debug/app-debug.apk
hzdb app launch com.meta.portal.sampleapp
```

## Architecture

Single-activity Compose app. `MainActivity` extends `ComponentActivity` — all UI is Jetpack Compose inside `setContent { }`.

### UI Structure

```
MainActivity (ComponentActivity)
  └─ SampleAppTheme (Material 3)
      └─ Scaffold + TopAppBar
          └─ ShowcaseScreen (vertically scrollable Column)
              ├─ UiElementsSection()
              ├─ PermissionsSection()
              ├─ CameraSection()
              └─ AudioRecorderSection()
```

### Key Composables

| Composable | Purpose |
|---|---|
| `ShowcaseScreen()` | Root scrollable column containing all demo sections |
| `UiElementsSection()` | Interactive Material 3 controls (buttons, text field, slider, checkbox, switch, radio buttons, dropdown, card) |
| `PermissionsSection()` | Requests camera, location, contacts, audio permissions individually or all at once |
| `PermissionRow()` | Reusable row showing permission name, granted/denied badge, and request button |
| `CameraSection()` | CameraX preview with camera enumeration and front/back selection via filter chips |
| `CameraViewfinder()` | AndroidView wrapping CameraX PreviewView with lifecycle-aware binding |
| `AudioRecorderSection()` | State machine for recording audio (MediaRecorder) and playing it back (MediaPlayer) |

## Key Dependencies

| Dependency | Purpose |
|---|---|
| `androidx.activity:activity-compose` | Compose integration with ComponentActivity |
| `androidx.compose.material3` | Material 3 UI components and theming |
| `androidx.camera:camera-*` | CameraX for camera preview (camera-core, camera2, lifecycle, view) |
| `androidx.lifecycle:lifecycle-runtime-ktx` | Lifecycle-aware coroutines |

Versions are managed in `gradle/libs.versions.toml`.

## Supported Portal Devices

| Device | minSdkVersion | Connection |
|---|---|---|
| Portal (1st and 2nd gen) | 28 / 29 | USB-C |
| Portal Mini | 29 | USB-C |
| Portal+ (1st and 2nd gen) | 28 / 29 | USB-C |
| Portal Go | 29 | USB-C (under rubber cover) |
| Portal TV | 29 | USB-C |

## Platform Constraints (Portal)

- **No Google Mobile Services (GMS):** Maps, Sign-In, Push, In-App Billing, Firebase will not function. Use non-GMS alternatives.
- **SDK versions:** Set `minSdkVersion` to 28 and `targetSdkVersion` to 29 for maximum compatibility. Portal runs older AOSP.
- **Contacts/Credentials:** Access to device contacts and device credentials (e.g. Facebook) is not possible.
- **Available features via standard Android permissions:** Camera, Microphone, Speaker, Bluetooth, Network, Touch input & keyboard, Storage write.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meta-quest/portal-samples](https://github.com/meta-quest/portal-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
