---
trigger: always_on
description: > Full API reference: [https://wearables.developer.meta.com/llms.txt?full=true](https://wearables.developer.meta.com/llms.txt?full=true)
---

# Meta Wearables DAT SDK — AI Instructions

> Full API reference: [https://wearables.developer.meta.com/llms.txt?full=true](https://wearables.developer.meta.com/llms.txt?full=true)
>
> Developer docs: [https://wearables.developer.meta.com/docs/develop/](https://wearables.developer.meta.com/docs/develop/)

# DAT SDK Conventions (Android)

## Architecture

The SDK is organized into three modules:

- **mwdat-core**: Device discovery, registration, permissions, device selectors
- **mwdat-camera**: StreamSession, VideoFrame, photo capture
- **mwdat-mockdevice**: MockDeviceKit for testing without hardware

## Kotlin Patterns

- Use `suspend` functions for async operations — no callbacks
- Use `StateFlow` / `Flow` for observing state changes
- Use `DatResult<T, E>` for error handling — not exceptions
- Prefer immutable collections
- Use `sealed interface` for state hierarchies

## Error Handling

The SDK uses `DatResult<T, E>` for type-safe error handling:

```kotlin
val result = Wearables.someOperation()
result.fold(
    onSuccess = { value -> /* handle success */ },
    onFailure = { error -> /* handle error */ }
)

// Or partial handling:
result.onSuccess { value -> /* handle success */ }
result.onFailure { error -> /* handle error */ }
```

Do **not** use `getOrThrow()` — always handle both paths.

## Naming Conventions

| Suffix     | Purpose                        | Example               |
| ---------- | ------------------------------ | --------------------- |
| `*Manager` | Long-lived resource management | `RegistrationManager` |
| `*Session` | Short-lived flow component     | `StreamSession`       |
| `*Result`  | DatResult type aliases         | `RegistrationResult`  |
| `*Error`   | Error sealed interfaces        | `WearablesError`      |

Methods: `get*`, `set*`, `check*`, `request*`, `observe*`

## Imports

```kotlin
import com.meta.wearable.dat.core.Wearables          // Entry point
import com.meta.wearable.dat.camera.StreamSession     // Camera streaming
import com.meta.wearable.dat.camera.types.*            // VideoFrame, PhotoData, etc.
```

For testing:

```kotlin
import com.meta.wearable.dat.mockdevice.MockDeviceKit  // MockDeviceKit
```

## Key Types

- `Wearables` — SDK entry point. Call `Wearables.initialize(context)` at startup
- `StreamSession` — Camera streaming session
- `VideoFrame` — Individual video frame with bitmap data
- `AutoDeviceSelector` — Auto-selects the best available device
- `SpecificDeviceSelector` — Selects a specific device by identifier
- `StreamConfiguration` — Configure video quality, frame rate
- `MockDeviceKit` — Factory for creating simulated devices in tests

## Links

- [Android API Reference](https://wearables.developer.meta.com/docs/reference/android/dat/0.5)
- [Developer Documentation](https://wearables.developer.meta.com/docs/develop/)
- [GitHub Repository](https://github.com/facebook/meta-wearables-dat-android)

# Camera Streaming (Android)

Guide for implementing camera streaming and photo capture with the DAT SDK.

## Key concepts

- **StreamSession**: Main interface for camera streaming
- **VideoFrame**: Individual video frames from the stream
- **StreamConfiguration**: Configure resolution, frame rate
- **PhotoData**: Still image captured from glasses

## Creating a StreamSession

```kotlin
import com.meta.wearable.dat.camera.types.StreamConfiguration
import com.meta.wearable.dat.camera.types.VideoQuality
import com.meta.wearable.dat.core.selectors.AutoDeviceSelector

val session = Wearables.startStreamSession(
    context = context,
    deviceSelector = AutoDeviceSelector(),
    streamConfiguration = StreamConfiguration(
        videoQuality = VideoQuality.MEDIUM,  // 504x896
        frameRate = 24,
    ),
)
```

### Resolution options

| Quality               | Size       |
| --------------------- | ---------- |
| `VideoQuality.HIGH`   | 720 x 1280 |
| `VideoQuality.MEDIUM` | 504 x 896  |
| `VideoQuality.LOW`    | 360 x 640  |

### Frame rate options

Valid values: `2`, `7`, `15`, `24`, `30` FPS.

Lower resolution and frame rate yield higher visual quality due to less Bluetooth compression.

## Observing stream state

`StreamSessionState` transitions: `STARTING` -> `STARTED` -> `STREAMING` -> `STOPPING` -> `STOPPED` -> `CLOSED`

```kotlin
lifecycleScope.launch {
    session.state.collect { state ->
        when (state) {
            StreamSessionState.STREAMING -> {
                // Stream is active, frames flowing
            }
            StreamSessionState.STOPPED -> {
                // Stream ended, release resources
            }
            StreamSessionState.CLOSED -> {
                // Session fully closed
            }
            else -> { /* handle other states */ }
        }
    }
}
```

## Receiving video frames

```kotlin
lifecycleScope.launch {
    session.videoStream.collect { frame ->
        // Display frame bitmap
        updatePreview(frame)
    }
}
```

## Photo capture

```kotlin
session.capturePhoto()
    .onSuccess { photoData ->
        // Handle captured photo data
        val imageBytes = photoData.data
    }
    .onFailure { error ->
        // Handle capture error
    }
```

## Bandwidth and quality


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [circus-kitchens/expo-meta-wearables-dat](https://github.com/circus-kitchens/expo-meta-wearables-dat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
