---
trigger: always_on
description: Flutter plugin providing a bridge to Meta's Wearables Device Access Toolkit (DAT) for integration with Meta AI Glasses (Ray-Ban Meta). Supports iOS (17.0+) and Android (API 29+).
---

# flutter_meta_wearables_dat

Flutter plugin providing a bridge to Meta's Wearables Device Access Toolkit (DAT) for integration with Meta AI Glasses (Ray-Ban Meta). Supports iOS (17.0+) and Android (API 29+).

- **pub.dev**: https://pub.dev/packages/flutter_meta_wearables_dat
- **GitHub**: https://github.com/rodcone/flutter_meta_wearables_dat
- **Example app**: https://github.com/rodcone/flutter_meta_wearables_dat/tree/main/example

## Code style

- All DAT calls go through `MetaWearablesDat` static methods — never instantiate the class
- Use `Provider`/`ChangeNotifier` for state management (see example app)
- Use `async`/`await` for all DAT operations
- Use `Stream` subscriptions for reactive state (registration, device availability, session state, errors)
- Dispose all stream subscriptions in your provider/widget `dispose()`
- Handle `PlatformException` for method channel errors

## Architecture

```
MetaWearablesDat (static API facade)
    ↓
MetaWearablesDatPlatform (abstract contract)
    ↓
MethodChannelMetaWearablesDat (method/event channel impl)
    ↓
Native: iOS (Swift) | Android (Kotlin)
```

Single import: `import 'package:flutter_meta_wearables_dat/flutter_meta_wearables_dat.dart';`

Communication:
- **Method channel** `flutter_meta_wearables_dat` — request/response calls
- **Event channels** — reactive streams for state changes

## Key types

### Enums

| Enum | Values |
|------|--------|
| `RegistrationState` | `unavailable(0)`, `available(1)`, `registering(2)`, `registered(3)` |
| `VideoCodec` | `raw('raw')`, `hvc1('hvc1')` |
| `StreamQuality` | `high('high')`, `medium('medium')`, `low('low')` |
| `StreamSessionState` | `stopping(0)`, `stopped(1)`, `waitingForDevice(2)`, `starting(3)`, `streaming(4)`, `paused(5)` |
| `PhotoCaptureFormat` | `heic('heic')`, `jpeg('jpeg')` |
| `FrameFormat` | `rawRgba`, `rawStraightRgba`, `png` |

### Classes

| Class | Key properties |
|-------|---------------|
| `StreamSessionError` | `code` (String), `message` (String), `isThermalCritical`, `isHingesClosed`, `isPermissionDenied` |
| `CameraPermissionException` | `code`, `message`, `details`, `isDeviceDisconnected`, `isPermissionDenied`, `isInternalError` |
| `CapturedPhoto` | `bytes` (Uint8List), `format` (String), `fileExtension`, `mimeType` |
| `CapturedFrame` | `bytes` (Uint8List), `width`, `height`, `format` (FrameFormat) |
| `VideoStreamSize` | `width`, `height`, `aspectRatio` |
| `VideoFrame` | `codec` (VideoCodec), `bytes` (Uint8List), `width`, `height`, `presentationTimestampUs`, `isKeyframe` |
| `BackgroundNotification` | `title`, `text`, `channelId`, `channelName`, `iconResourceName?` (Android only) |

### Error codes (StreamSessionError)

| Code | Meaning |
|------|---------|
| `internalError` | Internal SDK error |
| `deviceNotFound` | No matching device available |
| `deviceNotConnected` | Device is not connected |
| `timeout` | Operation timed out |
| `videoStreamingError` | Video stream failed |
| `permissionDenied` | Camera permission denied |
| `hingesClosed` | User folded the glasses |
| `thermalCritical` | Device overheating — streaming pauses automatically |

## API reference

```dart
// Registration
static Future<bool> startRegistration()
static Future<bool> handleUrl(String url)
static Future<bool> disconnect()
static Future<RegistrationState> getRegistrationState()
static Stream<RegistrationState> registrationStateStream()

// Permissions
static Future<bool> requestAndroidPermissions()   // No-op on iOS
static Future<bool> requestCameraPermission()
static Future<bool> getCameraPermissionStatus()

// Device monitoring
static Stream<bool> activeDeviceStream()
static Future<bool> restartActiveDeviceMonitoring()  // No-op on iOS

// Streaming
static Future<int> startStreamSession(
  String? deviceUUID, {
  double fps = 30.0,
  StreamQuality streamQuality = StreamQuality.high,
  VideoCodec videoCodec = VideoCodec.raw,
})  // Returns textureId for Texture widget
static Future<bool> stopStreamSession(String? deviceUUID)
static Stream<StreamSessionState> streamSessionStateStream()
static Stream<StreamSessionError> streamSessionErrorStream()
static Stream<VideoStreamSize> videoStreamSizeStream()

// Background streaming (opt-in; call before startStreamSession)
static Future<void> enableBackgroundStreaming({
  BackgroundNotification? androidNotification, // required on Android
})
static Future<void> disableBackgroundStreaming()
static Stream<VideoFrame> videoFramesStream()   // foreground + background

// Photo capture
static Future<CapturedPhoto> capturePhoto(
  String? deviceUUID, {
  PhotoCaptureFormat format = PhotoCaptureFormat.jpeg,
})

// Frame capture (Dart-side rasterization, no native call)
static Future<CapturedFrame?> captureStreamFrame(
  int textureId, {
  int width = 720,
  int height = 1280,
  FrameFormat format = FrameFormat.rawRgba,
})

// Mock device APIs live in the optional companion package
// `flutter_meta_wearables_dat_mock_device` — see "Testing with MockDeviceKit".
```

## Dev environment setup

### 1. Add dependency

```bash
flutter pub add flutter_meta_wearables_dat
```

### 2. iOS configuration

**Minimum deployment target:** iOS 17.0


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rodcone/flutter_meta_wearables_dat](https://github.com/rodcone/flutter_meta_wearables_dat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
