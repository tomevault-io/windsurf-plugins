---
trigger: always_on
description: CameraSync is an Android application that synchronizes GPS data and date/time from your Android phone to your camera via Bluetooth Low Energy (BLE). The app supports cameras from multiple vendors (Ricoh, Sony, etc.) and can sync to **multiple cameras simultaneously** in the background.
---


# CameraSync - Claude AI Assistant Guide

## Project Overview

CameraSync is an Android application that synchronizes GPS data and date/time from your Android phone to your camera via Bluetooth Low Energy (BLE). The app supports cameras from multiple vendors (Ricoh, Sony, etc.) and can sync to **multiple cameras simultaneously** in the background.

## Project Structure

This is an Android project built with:
- **Language**: Kotlin
- **Build System**: Gradle with Kotlin DSL
- **Target Platform**: Android (tested on Pixel 9 with Android 15)
- **Hardware Target**: BLE-enabled cameras (tested with Ricoh GR IIIx and Sony Alpha cameras)

## Key Technologies & Architecture

- **Multi-Device Architecture**: Supports pairing and syncing multiple cameras simultaneously.
- **Multi-Vendor Architecture**: Uses the Strategy Pattern to support different camera brands (Ricoh, Sony, and extensible to others).
- **Bluetooth Low Energy (BLE)**: Core communication protocol using the Kable library.
- **Android Foreground Services**: Maintains connections when app is backgrounded.
- **Location Services**: Centralized GPS data collection shared across all devices.
- **Proto DataStore**: Persistent storage for paired devices using Protocol Buffers.

## Architecture Overview

### Data Layer
- `PairedDevicesRepository`: Interface for managing paired devices (add, remove, enable/disable) and global sync state
- `DataStorePairedDevicesRepository`: Proto-based persistence implementation
- `CameraRepository`: BLE scanning and connection management
- `LocationRepository`: GPS location updates from Fused Location Provider

### Domain Layer
- `PairedDevice`: Domain model for stored paired cameras
- `DeviceConnectionState`: Sealed interface for device connection states
- `Camera`: Discovered camera model with vendor information
- `CameraVendor`: Strategy interface for vendor-specific protocols
- `VendorConnectionDelegate`: **NEW** Abstraction for handling vendor-specific connection/sync lifecycles
- `DefaultConnectionDelegate`: Standard implementation of the delegate
- `CameraVendorRegistry`: Registry managing all supported camera vendors

### Service Layer
- `MultiDeviceSyncService`: Foreground service managing all device connections
- `MultiDeviceSyncCoordinator`: Core sync logic for multiple concurrent connections
- `LocationCollectionCoordinator`: Centralized location collection with device registration

### UI Layer
- `DevicesListScreen`: Main screen showing paired devices with enable/disable toggles
- `PairingScreen`: BLE scanning and pairing flow for new devices
- Material 3 design with animated connection status indicators

### Utility Layer
- `BatteryOptimizationUtil`: Utility for checking battery optimization status and creating intents to system settings
  - Supports standard Android battery optimization settings
  - Detects and provides intents for OEM-specific battery settings (Xiaomi, Huawei, Oppo, Samsung, etc.)
  - Uses two-step verification to avoid false positives on package detection
- `BatteryOptimizationChecker`: Injectable interface for battery optimization checks (with `AndroidBatteryOptimizationChecker` implementation)
  - Allows mocking in tests via `FakeBatteryOptimizationChecker`

## Multi-Vendor Architecture

CameraSync uses a **Strategy Pattern** to support cameras from multiple manufacturers without modifying core sync logic. This architecture enables adding new camera brands by implementing vendor-specific adapters.

### Key Components

1. **CameraVendor Interface** (`domain/vendor/CameraVendor.kt`)
   - Defines what each vendor must provide: GATT specification, protocol encoding/decoding, device recognition
   - Each vendor implements device identification logic based on service UUIDs, device names, or manufacturer data
   - Vendors declare their capabilities (firmware version support, geo-tagging, etc.)

2. **VendorConnectionDelegate** (`domain/vendor/VendorConnectionDelegate.kt`)
   - **New**: Encapsulates the entire connection and sync lifecycle
   - Allows vendors to implement complex handshakes (like Sony's DD30/DD31 locking) in isolation
   - `KableCameraRepository` delegates sync operations to this interface

3. **CameraVendorRegistry** (`domain/vendor/CameraVendorRegistry.kt`)
   - Central registry managing all supported vendors
   - Identifies which vendor a discovered BLE device belongs to
   - Aggregates scan filter UUIDs and device name prefixes from all vendors for efficient BLE scanning

4. **Vendor Implementations** (`vendors/` package)
   - **Ricoh**: `vendors/ricoh/` - Supports GR IIIx, GR III, and other Ricoh cameras
   - **Sony**: `vendors/sony/` - Supports Alpha series cameras (ILCE-7M4, etc.)
   - Each vendor package contains:
     - `[Vendor]GattSpec`: BLE service and characteristic UUIDs
     - `[Vendor]Protocol`: Encoding/decoding logic for date/time and GPS data
     - `[Vendor]CameraVendor`: Device recognition and capabilities
     - `[Vendor]ConnectionDelegate`: Connection lifecycle implementation

5. **Vendor-Agnostic Core**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rock3r/CameraSync](https://github.com/rock3r/CameraSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
