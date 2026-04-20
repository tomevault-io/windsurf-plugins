---
trigger: always_on
description: > Full API reference: [https://wearables.developer.meta.com/llms.txt?full=true](https://wearables.developer.meta.com/llms.txt?full=true)
---

# Meta Wearables DAT SDK — AI Instructions

> Full API reference: [https://wearables.developer.meta.com/llms.txt?full=true](https://wearables.developer.meta.com/llms.txt?full=true)
>
> Developer docs: [https://wearables.developer.meta.com/docs/develop/](https://wearables.developer.meta.com/docs/develop/)


# DAT SDK Conventions (iOS)

## Architecture

The SDK is organized into three modules:
- **MWDATCore**: Device discovery, registration, permissions, device selectors
- **MWDATCamera**: StreamSession, VideoFrame, photo capture
- **MWDATMockDevice**: MockDeviceKit for testing without hardware

## Swift Patterns

- Use `async/await` for all SDK operations — the SDK is fully async
- Use `AsyncSequence` / publisher `.listen {}` for observing streams
- Annotate UI-updating code with `@MainActor`
- Never block the main thread with frame processing
- Handle errors with do/catch — the SDK throws typed errors

## Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Entry point | `Wearables.shared` | `Wearables.shared.startRegistration()` |
| Sessions | `*Session` | `StreamSession`, `DeviceStateSession` |
| Selectors | `*DeviceSelector` | `AutoDeviceSelector`, `SpecificDeviceSelector` |
| Config | `*Config` | `StreamSessionConfig` |
| Publishers | `*Publisher` | `statePublisher`, `videoFramePublisher` |

## Imports

```swift
import MWDATCore    // Registration, devices, permissions
import MWDATCamera  // StreamSession, VideoFrame, photo capture
```

For testing:
```swift
import MWDATMockDevice  // MockDeviceKit, MockRaybanMeta, MockCameraKit
```

## Key Types

- `Wearables` — SDK entry point. Call `Wearables.configure()` at launch, then use `Wearables.shared`
- `StreamSession` — Camera streaming session. Create with config + device selector
- `VideoFrame` — Individual video frame with `.makeUIImage()` convenience
- `AutoDeviceSelector` — Automatically selects the best available device
- `SpecificDeviceSelector` — Selects a specific device by identifier
- `StreamSessionConfig` — Configure video codec, resolution, frame rate
- `MockDeviceKit` — Factory for creating simulated devices in tests

## Error Handling

```swift
do {
    try Wearables.configure()
} catch {
    // Handle configuration error
}

do {
    try Wearables.shared.startRegistration()
} catch {
    // Handle registration error
}
```

## Links

- [iOS API Reference](https://wearables.developer.meta.com/docs/reference/ios_swift/dat/0.6)
- [Developer Documentation](https://wearables.developer.meta.com/docs/develop/)
- [GitHub Repository](https://github.com/facebook/meta-wearables-dat-ios)


# Camera Streaming (iOS)

Guide for implementing camera streaming and photo capture with the DAT SDK.

## Key concepts

- **StreamSession**: Main interface for camera streaming
- **VideoFrame**: Individual video frames — call `.makeUIImage()` to render
- **StreamSessionConfig**: Configure resolution, frame rate, and codec
- **PhotoData**: Still image captured from glasses

## Creating a StreamSession

```swift
import MWDATCamera
import MWDATCore

let wearables = Wearables.shared
let deviceSelector = AutoDeviceSelector(wearables: wearables)

let config = StreamSessionConfig(
    videoCodec: .raw,
    resolution: .medium,  // 504x896
    frameRate: 24
)

let session = StreamSession(
    streamSessionConfig: config,
    deviceSelector: deviceSelector
)
```

### Resolution options

| Resolution | Size |
|-----------|------|
| `.high` | 720 x 1280 |
| `.medium` | 504 x 896 |
| `.low` | 360 x 640 |

### Frame rate options

Valid values: `2`, `7`, `15`, `24`, `30` FPS.

Lower resolution and frame rate yield higher visual quality due to less Bluetooth compression.

## Observing stream state

`StreamSessionState` transitions: `stopping` → `stopped` → `waitingForDevice` → `starting` → `streaming` → `paused`

```swift
let stateToken = session.statePublisher.listen { state in
    Task { @MainActor in
        switch state {
        case .streaming:
            // Stream is active, frames are flowing
        case .waitingForDevice:
            // Waiting for glasses to connect
        case .stopped:
            // Stream ended — release resources
        case .paused:
            // Temporarily suspended — keep connection, wait
        default:
            break
        }
    }
}
```

## Receiving video frames

```swift
let frameToken = session.videoFramePublisher.listen { frame in
    guard let image = frame.makeUIImage() else { return }
    Task { @MainActor in
        self.previewImage = image
    }
}
```

## Starting and stopping

```swift
// Start streaming
Task { await session.start() }

// Stop streaming
Task { await session.stop() }
```

## Photo capture

Capture a still photo while streaming:

```swift
// Listen for photo data
let photoToken = session.photoDataPublisher.listen { photoData in
    let imageData = photoData.data
    // Convert to UIImage or save
}

// Trigger capture
session.capturePhoto(format: .jpeg)
```

## Bandwidth and quality

Resolution and frame rate are constrained by Bluetooth Classic bandwidth. The SDK automatically reduces quality when bandwidth is limited:
1. First lowers resolution (e.g., High → Medium)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebook/meta-wearables-dat-ios](https://github.com/facebook/meta-wearables-dat-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
