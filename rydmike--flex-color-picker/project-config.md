---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**FlexColorPicker** is a customizable Flutter color picker package supporting multiple selection modes:
- Material Design primary/accent colors with shades
- Black & white color selection
- Custom color swatches
- HSV color wheel picker
- Material Design 3 tonal palettes

**Key Details:**
- **Version**: 3.7.2
- **SDK**: Dart 3.0+, Flutter 3.35+
- **License**: BSD 3-Clause
- **Main Dependency**: `flex_seed_scheme` v3.5.0 (Material Design 3 color utilities)

## Development Commands

### Testing
```bash
# Run all tests with coverage
flutter test --coverage

# Run a single test file
flutter test test/color_picker_test.dart

# Run tests matching a pattern
flutter test test/*_test.dart

# Run patrol tests (UI integration tests)
flutter test test/color_picker_patrol_test.dart
```

### Code Analysis
```bash
# Run static analysis
flutter analyze

# The project uses RydMike's strict lint rules (all_lint_rules.yaml)
# with strict-casts, strict-inference, and strict-raw-types enabled
```

### Building
```bash
# Get dependencies
flutter pub get

# Build example app (Web)
cd example
flutter build web

# Build example app (other platforms)
flutter build macos
flutter build windows
flutter build linux
flutter build apk
flutter build ios
```

### Example App
```bash
# Run the example/demo app
cd example
flutter run

# The example app demonstrates all picker features and configurations
# It uses Riverpod for state management and Hive for persistence
```

## Architecture Overview

### Core Widget Pattern
The main `ColorPicker` widget (`lib/src/color_picker.dart`, ~3,300 lines) is a large `StatefulWidget` with 50+ configuration parameters. It follows a **composition-based architecture** where functionality is split into specialized sub-widgets rather than using inheritance.

### Key Sub-Widgets (lib/src/widgets/)
- **MainColors**: Displays primary/accent color swatches
- **ShadeColors**: Shows shades of selected color
- **OpacitySlider**: Custom-painted opacity control with track and thumb
- **ColorCodeField**: Hex color input/display with validation
- **ColorPickerToolbar**: Copy-paste action buttons
- **RecentColors**: Recently selected colors display
- **TonalPaletteColors**: Material 3 tonal palette display
- **ColorWheelPicker**: HSV-based arbitrary color selection

### Data Models (lib/src/models/)
- **ColorPickerType** enum: `primary`, `accent`, `both`, `bw`, `custom`, `customSecondary`, `wheel`
- **ColorPickerActionButtons**: Dialog button configuration and styling
- **ColorPickerCopyPasteBehavior**: Defines copy-paste modes and color code formats

### Callback Architecture
Heavy use of `ValueChanged<Color>` callbacks for reactive state management:
- `onColorChanged`: Main color selection callback (required)
- `onColorChangeStart`/`onColorChangeEnd`: Lifecycle callbacks
- `onRecentColorsChanged`: Recent colors list updates

### Universal Widgets (lib/src/universal_widgets/)
Private, reusable utility widgets not exported from the package:
- **ContextPopupMenu**: Cross-platform context menu implementation
- **IfWrapper**: Conditional widget wrapping helper
- **DryIntrinsic**: Workaround for Flutter issue #71687

### Extensions (lib/src/color_picker_extensions.dart)
- `Color` extensions: `.alpha256`, `.alphaBlend()`, `.brighten()`, etc.
- `String` extensions: `.toColor()` for hex parsing
- These provide convenient color manipulation utilities

## Code Quality Standards

### Linting
The project uses **RydMike's strict lint rules** (v2.5.0) with all lint rules enabled, then selectively disabled. Key characteristics:

- **Strict type checking**: `always_specify_types` is enabled (unlike most Flutter projects)
- **Immutability**: All public widget classes marked `@immutable`
- **Documentation**: `public_member_api_docs` enforced for the package
- **Single quotes**: `prefer_single_quotes` over double quotes
- **No unsafe casts**: Use type checks (`is`) instead of `as` casts
- **Variance safety**: Private fields for contravariant generic types (see `unsafe_variance` lint)

### Type Safety Critical Rules
```yaml
strict-casts: true
strict-inference: true
strict-raw-types: true
```

### Common Lint Patterns
- **Always specify types** explicitly, even for local variables
- Use `is` checks instead of `as` casts (avoids `unsafe_variance` issues)
- Document all public APIs with `///` doc comments
- Follow Flutter repo style for constructors (default first, then properties, then named constructors)

### Handling `unsafe_variance`
When a generic type parameter appears in a contravariant position (like `ValueChanged<T>`), use this pattern:
```dart
class MyWidget<T> extends StatefulWidget {
  const MyWidget({required ValueChanged<T> onChanged}) : _onChanged = onChanged;
  final Function _onChanged; // Store as Function, not ValueChanged<T>

  // Cast at call site:
  void _handleChange(T value) {
    (_onChanged as ValueChanged<T>)(value);
  }
}
```

## Testing Patterns

### Test Structure (test/)
- Each source file has a corresponding `*_test.dart` file
- Tests use `flutter_test` and `patrol_finders` for UI testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rydmike/flex_color_picker](https://github.com/rydmike/flex_color_picker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
