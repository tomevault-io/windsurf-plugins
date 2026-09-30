---
trigger: always_on
description: Instructions for coding agents (Claude Code, Codex and others) working in this repository.
---

# settings_ui: agent instructions

Instructions for coding agents (Claude Code, Codex and others) working in this repository.
`CLAUDE.md` only imports this file (`@AGENTS.md`); edit this file, and keep `CLAUDE.md` as that one line.

## About this project

`settings_ui` is a published Flutter package (pub.dev: `settings_ui`, current version: `4.0.0`) that renders native-looking settings screens for iOS, macOS, Windows, Android, Linux, Fuchsia and the web from a single API. It is used in production by thousands of apps, so treat every public API and default-look change as a breaking change for someone.

- Requires Flutter >=3.44 and Dart >=3.12.
- Built on the decoupled [`material_ui`](https://pub.dev/packages/material_ui) and [`cupertino_ui`](https://pub.dev/packages/cupertino_ui) packages. In `lib/`, `test/` and `example/`, import `package:material_ui/material_ui.dart` and `package:cupertino_ui/cupertino_ui.dart` (plus non-design libraries such as `package:flutter/widgets.dart`, `foundation.dart`, `services.dart`). Never import `package:flutter/material.dart` or `package:flutter/cupertino.dart`: their `Theme`/`CupertinoTheme` are different classes, so the package would stop seeing the app theme.
- Apps that have not moved to material_ui stay on `settings_ui ^3.0.1`.

## Commands

```bash
flutter pub get

# Format (CI fails on unformatted code). `flutter format` no longer exists.
dart format .
dart format --output=none --set-exit-if-changed .   # CI check

# Lint
flutter analyze .

# All unit/widget tests, as CI runs them
flutter test --coverage --test-randomize-ordering-seed random

# One group or test (the files in test/settings_tests have no main())
flutter test test/widget_test.dart --name "CupertinoSettingsSwitch"

# Example app, and its integration test (needs a running device, simulator or emulator)
cd example && flutter run
# Open a screen in a style and brightness directly (example/lib/utils/launch_options.dart):
# options screen, platform, page, theme, from the web URL's query, the initial route
# (--route, or #/... on the web; its path is the screen) or --dart-define
cd example && flutter run -d chrome   # then /?screen=split-view&platform=macOS&theme=dark
cd example && flutter run -d macos --route '/split-view?platform=windows&page=system'
cd example && flutter run -d macos --dart-define=SCREEN=macos --dart-define=THEME=dark
cd example && flutter test integration_test/integration_test.dart -d <device-id>
cd example && flutter test integration_test/split_view_flows_test.dart -d <device-id>
```

## Architecture

### Platform dispatch

Every public widget (`SettingsSection`, `SettingsTile`) is a thin dispatcher. At build time it reads `SettingsTheme.of(context).platform` and returns one of six implementations:

- `iOS` → iOS style (`platforms/ios_*`)
- `macOS` → macOS System Settings style (`platforms/macos_*`)
- `windows` → Windows 11 (Fluent) style (`platforms/fluent_*`)
- `android`, `fuchsia` → Android style (`platforms/android_*`)
- `linux` → GNOME (libadwaita) style (`platforms/adwaita_*`)
- `web` → web style (`platforms/web_*`)

```
lib/src/tiles/
  settings_tile.dart              ← dispatcher
  platforms/
    android_settings_tile.dart
    ios_settings_tile.dart
    web_settings_tile.dart
    macos_settings_tile.dart
    fluent_settings_tile.dart
    adwaita_settings_tile.dart
    cupertino_settings_switch.dart  ← public, used by the iOS tile
    macos_settings_switch.dart      ← public, used by the macOS tile
    fluent_settings_switch.dart     ← public, used by the Windows tile
    adwaita_settings_switch.dart    ← public, used by the GNOME tile
    adwaita_symbolic_icons.dart     ← AdwaitaPanDownIcon public; go-next internal
```

Same pattern for `lib/src/sections/`. `lib/src/list/settings_list.dart` resolves the platform, brightness and default padding.

### Theme propagation

`SettingsList` resolves the platform and brightness, gets the style defaults from `ThemeProvider.getTheme()`, merges the user's `lightTheme`/`darkTheme` (`SettingsThemeData`) over them, and pushes the result down through `SettingsTheme` (an `InheritedWidget`). Tiles and sections read `SettingsTheme.of(context).themeData`; never pass theme values through constructors.

`ThemeProvider` (`lib/src/utils/theme_provider.dart`) holds the defaults: Android and web derive colors from the Material 3 `ColorScheme`; iOS, macOS, Windows and GNOME use fixed system colors (iOS system colors, `NSColor` label colors, WinUI theme resources from `lib/src/utils/fluent_tokens.dart`, libadwaita colors) and ignore `ColorScheme`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yako-dev/flutter-settings-ui](https://github.com/yako-dev/flutter-settings-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
