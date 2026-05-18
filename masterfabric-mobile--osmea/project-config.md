---
trigger: always_on
description: OSMEA text extensions - font, alignment, spacing, decoration with context
---


# OSMEA Text Extensions - Cursor Rules

## 📦 Package Overview
OSMEA Text Extensions provide comprehensive utility extensions for text styling in Flutter applications. These extensions work seamlessly with `OsmeaComponents.text` to provide consistent typography, spacing, and styling across the entire application.

## Import Statement
Always import OsmeaComponents at the top of your files:
```dart
import 'package:osmea_components/osmea_components.dart';
```

## Color Usage

### ✅ Use OsmeaColors
```dart
// ✅ Correct - using OsmeaColors
OsmeaComponents.text(
  'Hello',
  color: OsmeaColors.nordicBlue,
  fontSize: context.fontSizeMedium,
)

OsmeaComponents.text(
  'Primary text',
  color: OsmeaColors.textPrimary,
  fontSize: context.fontSizeLarge,
)

OsmeaComponents.text(
  'Secondary text',
  color: OsmeaColors.textSecondary,
  fontSize: context.fontSizeMedium,
)
```

### ❌ Don't Use Standard Colors
```dart
// ❌ Wrong - using standard Colors
OsmeaComponents.text(
  'Hello',
  color: Colors.blue,
  fontSize: context.fontSizeMedium,
)

OsmeaComponents.text(
  'Primary text',
  color: Colors.black,
  fontSize: context.fontSizeLarge,
)
```

## 🎯 Development Guidelines

### 📁 File Structure
```
packages/components/lib/src/
└── utils/
    └── text_extensions.dart         # Text utility extensions
```

### 🎨 Extension Categories

#### 1. **Core Typography Extensions**
- `FontWeightExtension` - Font weight utilities (thin to black)
- `TextSizeX` - Font size utilities (tiny to extra large)
- `FontStyleExtension` - Font style options (normal, italic)
- `FontFamilyExtension` - Font family selection (20+ fonts)

#### 2. **Text Layout Extensions**
- `TextAlignExtension` - Text alignment (left, right, center, justify, start, end)
- `TextOverflowExtension` - Text overflow handling (clip, fade, ellipsis, visible)
- `TextMaxLineExtension` - Line limit controls (1 to unlimited)
- `TextBaselineExtension` - Baseline alignment (alphabetic, ideographic)

#### 3. **Text Styling Extensions**
- `TextDecorationExtension` - Text decorations (none, underline, overline, lineThrough)
- `TextDecorationStyleExtension` - Decoration styles (solid, double, dotted, dashed, wavy)
- `TextCapitalizationExtension` - Text capitalization (words, sentences, characters, none)

#### 4. **Advanced Typography Extensions**
- `FontFeatureExtension` - Font features (smallCaps, oldstyleNums, liningNums, etc.)
- `FontVariationExtension` - Font variations (normal, wide, condensed, slant)
- `LetterSpacingExtension` - Letter spacing (tight to extra loose)
- `WordSpacingExtension` - Word spacing (tight to loose)
- `TextLeadingDistributionExtension` - Line height distribution (proportional, even)

### 🔧 Usage Rules

#### 1. Primary Usage Pattern with OsmeaComponents.text
```dart
// ✅ Good - Use extensions with OsmeaComponents.text
OsmeaComponents.text(
  'Page Title',
  fontSize: context.fontSizeExtraLarge,
  fontFamily: context.fontRoboto,
  fontWeight: context.semiBold,
  letterSpacing: context.letterSpacingWide,
  textAlign: context.textCenter,
  maxLines: context.maxLineTwo,
  overflow: context.ellipsis,
  color: OsmeaColors.nordicBlue,
)

// ✅ Good - Use extensions for additional styling
OsmeaComponents.text(
  'Styled text with extensions',
  fontSize: context.fontSizeMedium,
  fontFamily: context.fontMontserrat,
  fontWeight: context.medium,
  letterSpacing: context.letterSpacingNormal,
  wordSpacing: context.wordSpacingWide,
  textAlign: context.textJustify,
  decoration: context.underline,
  decorationStyle: context.solid,
  color: OsmeaColors.textPrimary,
)

// ❌ Bad - Hard-coded values instead of extensions
OsmeaComponents.text(
  'Styled text',
  fontSize: 16,
  fontFamily: 'Roboto',
  fontWeight: FontWeight.w500,
  letterSpacing: 0.5,
  wordSpacing: 1.0,
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)
```

#### 2. Font Family Selection
```dart
// ✅ Good - Use font family extensions
OsmeaComponents.text(
  'Primary text',
  fontSize: context.fontSizeLarge,
  fontFamily: context.fontRoboto,
  color: OsmeaColors.textPrimary,
)

OsmeaComponents.text(
  'Code text',
  fontSize: context.fontSizeMedium,
  fontFamily: context.fontFiraCode,
  color: OsmeaColors.textSecondary,
)

OsmeaComponents.text(
  'Display text',
  fontSize: context.fontSizeExtraLarge,
  fontFamily: context.fontPlayfairDisplay,
  color: OsmeaColors.nordicBlue,
)

// ❌ Bad - Hard-coded font names
OsmeaComponents.text(
  'Primary text',
  fontSize: context.fontSizeLarge,
  fontFamily: 'Roboto',
)
```

#### 3. Font Weight and Size
```dart
// ✅ Good - Use weight and size extensions
OsmeaComponents.text(
  'Bold heading',
  fontSize: context.fontSizeExtraLarge,
  fontWeight: context.bold,
  color: OsmeaColors.textPrimary,
)

OsmeaComponents.text(
  'Light text',
  fontSize: context.fontSizeMedium,
  fontWeight: context.light,
  color: OsmeaColors.textSecondary,
)

// ❌ Bad - Hard-coded weights and sizes
OsmeaComponents.text(
  'Bold heading',
  fontSize: 32,
  fontWeight: FontWeight.w700,
)
```

#### 4. Text Alignment and Layout
```dart
// ✅ Good - Use alignment extensions
OsmeaComponents.text(
  'Centered title',
  fontSize: context.fontSizeExtraLarge,
  textAlign: context.textCenter,
  color: OsmeaColors.textPrimary,
)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masterfabric-mobile/osmea](https://github.com/masterfabric-mobile/osmea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
