---
trigger: always_on
description: `flutter_platform_alert` is a Flutter plugin that bridges native platform APIs to:
---

# GitHub Copilot Instructions for flutter_platform_alert

## Project Overview

`flutter_platform_alert` is a Flutter plugin that bridges native platform APIs to:

1. **Play alert sounds** – using the system's native audio APIs.
2. **Show native alert dialogs** – using each platform's native dialog API.

### Supported Platforms and Their Native APIs

| Platform | Sound API | Dialog API |
|----------|-----------|------------|
| Android | `RingtoneManager` | `AlertDialog` (Material Design) |
| iOS | `AudioServices` | `UIAlertController` |
| macOS | `NSSound.beep()` | `NSAlert` |
| Windows | `MessageBeep()` | `MessageBox` / `TaskDialogIndirect` |
| Linux | `gtk_widget_error_bell()` | `GtkMessageDialog` |

---

## Repository Layout

```
flutter_platform_alert/
├── lib/
│   ├── flutter_platform_alert.dart        # Public export barrel file
│   └── src/
│       ├── flutter_platform_alert.dart    # Main class (MethodChannel calls)
│       ├── alert_button.dart              # AlertButton / CustomButton enums
│       ├── helpers.dart                   # Enum ↔ string conversion helpers
│       ├── window_position.dart           # AlertWindowPosition enum
│       ├── alert_options/
│       │   ├── platform_alert_options.dart  # Platform-routing options class
│       │   ├── ios_alert_options.dart
│       │   ├── android_alert_options.dart
│       │   ├── macos_alert_options.dart
│       │   └── windows_alert_options.dart
│       └── styles/
│           ├── alert_style.dart           # AlertButtonStyle enum
│           ├── icon_style.dart            # IconStyle enum
│           ├── ios_alert_style.dart       # IosAlertStyle enum
│           └── ios_button_style.dart      # IosButtonStyle enum
├── android/   # Kotlin implementation
├── ios/       # Swift implementation (SPM-compatible)
├── macos/     # Swift implementation (SPM-compatible)
├── windows/   # C++ implementation
├── linux/     # C implementation (GTK3)
├── example/   # Example Flutter app demonstrating all features
├── pubspec.yaml
└── analysis_options.yaml
```

---

## Dart / Flutter Public API

### `FlutterPlatformAlert` (static class)

All public methods are `static` and `async`. The class cannot be instantiated.

```dart
// Play the system alert sound.
static Future<void> playAlertSound({
  IconStyle iconStyle = IconStyle.none,
})

// Show a standard platform alert with predefined button sets.
// Returns the button the user clicked.
static Future<AlertButton> showAlert({
  required String windowTitle,
  required String text,
  AlertButtonStyle alertStyle = AlertButtonStyle.ok,
  IconStyle iconStyle = IconStyle.none,
  PlatformAlertOptions? options,
  AlertWindowPosition windowPosition = AlertWindowPosition.parentWindowCenter,
})

// Show a platform alert with fully customised button labels.
// Returns the button the user clicked.
static Future<CustomButton> showCustomAlert({
  required String windowTitle,
  required String text,
  IconStyle iconStyle = IconStyle.none,
  String? positiveButtonTitle,
  String? negativeButtonTitle,
  String? neutralButtonTitle,
  PlatformAlertOptions? options,
  AlertWindowPosition windowPosition = AlertWindowPosition.parentWindowCenter,
  String iconPath = '',   // Flutter asset path; only ICO format is supported on Windows
})
```

### Key Enums

| Enum | Values |
|------|--------|
| `AlertButton` | `abortButton`, `cancelButton`, `continueButton`, `ignoreButton`, `noButton`, `okButton`, `retryButton`, `tryAgainButton`, `yesButton`, `other` |
| `CustomButton` | `positiveButton`, `negativeButton`, `neutralButton`, `other` |
| `AlertButtonStyle` | `abortRetryIgnore`, `cancelTryContinue`, `ok`, `okCancel`, `retryCancel`, `yesNo`, `yesNoCancel` |
| `IconStyle` | `none`, `exclamation`, `warning`, `information`, `asterisk`, `question`, `stop`, `error`, `hand` |
| `IosAlertStyle` | `alertDialog`, `actionSheet` |
| `IosButtonStyle` | `cancel`, `destructive`, `normal` |
| `AlertWindowPosition` | `parentWindowCenter`, `screenCenter` |

### Platform-Specific Options

All options are passed through `PlatformAlertOptions`, which routes to the correct platform at runtime via `Platform.isX` checks:

```dart
PlatformAlertOptions({
  IosAlertOptions? ios,
  MacosAlertOptions? macos,
  WindowsAlertOptions? windows,
  AndroidAlertOptions? android,
})
```

Each options class exposes a `toJson()` method that serialises fields for the `MethodChannel` call.

---

## MethodChannel Protocol

The Dart layer communicates with native code over the channel named **`flutter_platform_alert`**.

### Method: `playAlertSound`
```json
{ "iconStyle": "<IconStyle.name>" }
```

### Method: `showAlert`
```json
{
  "windowTitle": "string",
  "text": "string",
  "alertStyle": "<AlertButtonStyle.stringValue>",
  "iconStyle": "<IconStyle.stringValue>",
  "position": 0,
  // ...platform options flattened from PlatformAlertOptions.toJson()
}
```

### Method: `showCustomAlert`
```json
{
  "windowTitle": "string",
  "text": "string",
  "iconStyle": "<IconStyle.name>",
  "positiveButtonTitle": "string",
  "negativeButtonTitle": "string",
  "neutralButtonTitle": "string",
  "position": 0,
  "iconPath": "/absolute/path/to/asset",
  "base64Icon": "<base64-encoded image>",
  // ...platform options flattened from PlatformAlertOptions.toJson()
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zonble/flutter_platform_alert](https://github.com/zonble/flutter_platform_alert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
