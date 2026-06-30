---
trigger: always_on
description: Blue Falcon is a Bluetooth Low Energy (BLE) Kotlin Multiplatform library supporting iOS, Android, macOS, Raspberry Pi, Windows, and JavaScript.
---

# Blue Falcon - Copilot Instructions

Blue Falcon is a Bluetooth Low Energy (BLE) Kotlin Multiplatform library supporting iOS, Android, macOS, Raspberry Pi, Windows, and JavaScript.

## Build, Test, and Lint Commands

All commands are run from the `library/` directory using the Gradle wrapper from the project root:

```bash
# Build the library (all targets)
cd library && ../gradlew build

# Build specific targets
cd library && ../gradlew compileKotlinAndroid
cd library && ../gradlew compileKotlinIos
cd library && ../gradlew compileKotlinWindows windowsJar
cd library && ../gradlew jsBrowserProductionWebpack

# Assemble outputs
cd library && ../gradlew assemble

# Clean build artifacts
cd library && ../gradlew clean

# Tests (platform-specific)
cd library && ../gradlew linkDebugTestIosX64
cd library && ../gradlew linkDebugTestMacosArm64
cd library && ../gradlew jsTestClasses

# Publish to Maven Central (requires credentials)
cd library && ../gradlew publish
```

**Note**: The Gradle wrapper (`gradlew`) is located at the project root, but the build configuration is in the `library/` subdirectory.

## Architecture Overview

### Multiplatform Structure

Blue Falcon uses Kotlin Multiplatform with an **expect/actual** pattern to provide a unified API while implementing platform-specific Bluetooth functionality:

- **commonMain**: Contains `expect` declarations for:
  - `BlueFalcon` - Main BLE manager class
  - `BluetoothPeripheral`, `BluetoothService`, `BluetoothCharacteristic` - Core BLE entities
  - `ApplicationContext` - Platform-specific context wrapper
  - `Uuid`, `ServiceFilter` - Platform-agnostic types
  - `BlueFalconDelegate` - Event callback interface (concrete, not expect/actual)
  - `Logger` interface and `PrintLnLogger` implementation

- **Platform implementations** (`*Main` source sets):
  - `androidMain` - Uses Android BLE APIs (`BluetoothGatt`, `BluetoothAdapter`)
  - `iosMain` - Uses CoreBluetooth framework via Kotlin/Native
  - `macosMain` - Uses CoreBluetooth framework via Kotlin/Native
  - `jsMain` - Uses Web Bluetooth API
  - `windowsMain` - Uses Windows Runtime (WinRT) Bluetooth APIs via JNI (includes native C++ code in `src/windowsMain/cpp/`)
  - `rpiMain` - Raspberry Pi-specific implementation (currently disabled in build)
  - `nativeMain` - Shared code for native targets (iOS/macOS)

### Key Architectural Patterns

1. **Delegate Pattern**: `BlueFalconDelegate` provides callback methods for BLE events (discovery, connection, characteristic changes). Multiple delegates can be registered via `blueFalcon.delegates.add(delegate)`.

2. **StateFlow for State Management**:
   - `blueFalcon.peripherals` - Emits discovered peripherals as a `Set<BluetoothPeripheral>`
   - `blueFalcon.managerState` - Emits `BluetoothManagerState` (Ready/NotReady)

3. **Platform Context Abstraction**: Each platform has its own `ApplicationContext` implementation:
   - Android: Wraps Android `Context`
   - iOS/macOS: Empty wrapper (not required)
   - Windows: Empty wrapper (not required)
   - JS: Empty wrapper (not required)

4. **Auto-Discovery**: When `autoDiscoverAllServicesAndCharacteristics = true` (default), services and characteristics are automatically discovered after connection.

### Important Files

- `library/src/commonMain/kotlin/dev/bluefalcon/BlueFalcon.kt` - Main API definition (expect class)
- `library/src/commonMain/kotlin/dev/bluefalcon/BlueFalconDelegate.kt` - Event callback interface
- `library/src/androidMain/kotlin/dev/bluefalcon/BlueFalcon.kt` - Android implementation
- `library/src/nativeMain/kotlin/dev/bluefalcon/BlueFalcon.kt` - iOS/macOS implementation
- `library/src/windowsMain/kotlin/dev/bluefalcon/BlueFalcon.kt` - Windows implementation
- `library/src/windowsMain/cpp/` - Native Windows Bluetooth bridge (JNI C++ code)

## Key Conventions

### Expect/Actual Pattern

All platform-specific types must have an `expect` declaration in `commonMain` and corresponding `actual` implementations in each platform source set. When adding new functionality:

1. Add `expect` declaration in `commonMain/kotlin/dev/bluefalcon/`
2. Add `actual` implementation in each platform's `*Main/kotlin/dev/bluefalcon/`
3. Ensure all platforms compile, even if some platforms provide no-op implementations

### Logger Usage

- The `Logger` interface accepts an optional `Throwable` cause parameter on all methods
- Methods signature: `error(message: String, cause: Throwable? = null)`
- `PrintLnLogger` is the default implementation (prints to stdout)
- Pass `log = null` to `BlueFalcon` constructor to disable logging

### Platform Identifiers

`retrievePeripheral(identifier: String)` uses different identifier formats per platform:
- **Android**: MAC address format (e.g., `"00:11:22:33:44:55"`)
- **iOS/Native**: UUID format (e.g., `"XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"`)

### Windows-Specific Notes

- Windows implementation requires native DLL (`bluefalcon-windows.dll`)
- Native code is in `library/src/windowsMain/cpp/`
- Uses Windows Runtime (WinRT) APIs through JNI
- Requires Windows 10 version 1803+ and JDK 17+
- Build verification runs on `windows-latest` in CI

### Version and Publishing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Reedyuk/blue-falcon](https://github.com/Reedyuk/blue-falcon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
