---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Andock** (package: `com.github.andock`) is an Android application that runs Docker containers without root privileges using PRoot (user-space chroot). It's a complete Kotlin reimplementation of the udocker concept, designed specifically for Android with full internationalization support (Chinese/English).

**Key Technologies:**
- Kotlin 2.2.21 + Jetpack Compose (Material Design 3, BOM 2025.12.00)
- Hilt 2.57.2 dependency injection with AssistedInject
- Ktor 3.3.3 HTTP client (OkHttp engine)
- Room 2.8.4 database
- FlowRedux 2.0.0 state machines (containers, downloads, registries)
- PRoot v0.15 execution engine (from green-green-avk/proot)
- Coroutines 1.10.2 & Flow for async/reactive programming
- Paging 3.3.5 for Docker Hub search
- CameraX 1.5.2 + ML Kit 17.3.0 for QR code scanning
- Shizuku 13.1.5 for system service integration
- http4k 6.23.1.0 for Docker API server

## Build & Development Commands

### Build the app
```bash
./gradlew assembleDebug
```

### Run instrumented tests
```bash
./gradlew connectedAndroidTest
./gradlew connectedDebugAndroidTest --tests com.github.adocker.ImagePullAndRunTest
```

**Test requirements:** Device/emulator with ADB, JDK 17+. Tests requiring PRoot skip if binary unavailable.

### Install and run on device
```bash
export PATH="$PATH:$HOME/Library/Android/sdk/platform-tools"
adb install app/build/outputs/apk/debug/app-debug.apk
adb logcat -d | grep -i "PRootEngine\|ImagePull\|Container"
```

### PRoot Build System
PRoot is automatically compiled from source during the build process via the `proot` module:
- **Source:** Downloads PRoot v0.15 and talloc 2.4.2 automatically
- **Architectures:** arm64-v8a, armeabi-v7a, x86_64, x86
- **Output:** `libproot.so`, `libproot_loader.so`, `libproot_loader32.so` (for 64-bit archs)
- **16KB Page Alignment:** Configured for Android 15+ compatibility

Build scripts location: `proot/src/main/cpp/scripts/`

## Architecture

### Module Structure

Multi-module architecture:
- **proot/** - PRoot native build module (Android Library)
  - `src/main/cpp/CMakeLists.txt` - CMake build configuration
  - `src/main/cpp/scripts/` - Build scripts for talloc and proot
  - Downloads and compiles PRoot v0.15 + talloc 2.4.2 from source
  - Outputs: `libproot.so`, `libproot_loader.so`, `libproot_loader32.so`

- **daemon/** - Core business logic (Android Library)
  - `app/` - AppContext, AppInitializer, AppModule
  - `containers/` - Container, ContainerManager, ContainerState, ContainerStateMachine
  - `database/` - Room DB, DAOs, entities (ImageEntity, LayerEntity, ContainerEntity, etc.)
  - `engine/` - PRootEngine, PRootEnvironment
  - `images/` - ImageManager, ImageClient, ImageReference, downloader (FlowRedux)
  - `http/` - UnixHttp4kServer, TcpHttp4kServer, HttpProcessor
  - `server/` - DockerApiServer, route modules (Hilt multibinding)
  - `registries/` - RegistryManager, RegistryStateMachine (FlowRedux)
  - `search/` - SearchPagingSource, SearchRepository (Paging 3 + DataStore)
  - `io/` - File, Zip, Tailer (symlink handling, tar extraction)
  - `os/` - ProcessLimitCompat, RemoteProcess (Shizuku integration)
  - `logging/` - TimberLogger, TimberServiceProvider (SLF4J + Timber)

- **app/** - UI module (Android Application)
  - `ui/screens/` - Screen composables by feature (containers, images, search, etc.)
  - `ui/components/` - Reusable UI components
  - `ui/theme/` - Spacing.kt, IconSize.kt (Material Design 3)

**AIDL Files** (for Shizuku): `daemon/src/main/aidl/com/github/andock/daemon/os/`

### Data Flow

1. **Image Pull:** ViewModel → ImageRepository → DockerRegistryApi → download layers to `layersDir/{digest}.tar.gz` (compressed) → save ImageEntity to Room
2. **Container Creation:** ViewModel → ContainerManager → create `containersDir/{containerId}/ROOT/` → extract layers from compressed files → save ContainerEntity to Room → create Container instance
3. **Container Execution:** ViewModel → Container.start() → ContainerStateMachine transitions → PRootEngine launches process → track state via Container.state StateFlow

### Critical Architecture Details

#### Container Status Management

**IMPORTANT:** Container status is NOT stored in the database.

- **Database (`ContainerEntity`)**: Only stores static configuration (name, imageId, config, created timestamp)
- **Runtime State (`Container`)**: Each instance has a state machine tracking current state
- **UI Layer**: Directly uses `ContainerState` class names for display

**Container State Machine states:**
- `Created`, `Starting`, `Running`, `Stopping`, `Exited`, `Dead`, `Removing`, `Removed`

**Real-time State Observation:**
```kotlin
// In Composable - observe state in real-time
val containerState by container.state.collectAsState()

when (containerState) {
    is ContainerState.Running -> ShowStopButton()
    else -> ShowStartButton()
}
```

**Container API:**
```kotlin
container.getInfo() // Returns Result<ContainerEntity>
container.start()   // Start container
container.stop()    // Stop container
container.remove()  // Remove container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LukeXeon/adocker](https://github.com/LukeXeon/adocker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
