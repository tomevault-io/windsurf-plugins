---
trigger: always_on
description: OSMEA text system - OsmeaComponents.text, OsmeaTextStyle, typography
---


# OSMEA Text System - Cursor Rules

## 📦 Package Overview
OSMEA Text System provides a unified text solution through `OsmeaComponents.text` with comprehensive style selection. This system offers 20+ text variants, 15+ extension categories, and simplified component usage for consistent typography across Flutter applications.

## Import Statement
Always import OsmeaComponents at the top of your files:
```dart
import 'package:osmea_components/osmea_components.dart';
```

## 🎯 Development Guidelines

### 📁 File Structure
```
packages/components/lib/src/
├── components/text/
│   └── text.dart                    # Main text components
├── styles/
│   └── text_style.dart              # Typography system
└── utils/
    └── text_extensions.dart         # Text utility extensions
```

### 🎨 System Architecture

#### 1. **Text Extensions** (`text_extensions.dart`)
- Font families, weights, sizes, and styles
- Text alignment, decoration, and overflow
- Letter spacing, word spacing, and line height
- Font features and variations

#### 2. **Text Styles** (`text_style.dart`)
- Predefined typography system
- 20+ text variants (display, headline, body, etc.)
- Consistent spacing and sizing
- Theme integration

#### 3. **Text Components** (`text.dart`)
- Unified `OsmeaComponents.Text` component
- Style selection through variants
- Animation and interaction support
- Accessibility features

### 🔧 Usage Rules

#### 1. Color Usage
```dart
// ✅ Good - Use OsmeaColors for consistent theming
OsmeaComponents.text(
  'Primary text',
  textStyle: OsmeaTextStyle.headlineLarge(context),
  color: OsmeaColors.nordicBlue,
)

OsmeaComponents.text(
  'Secondary text',
  textStyle: OsmeaTextStyle.bodyMedium(context),
  color: OsmeaColors.thunder,
)

OsmeaComponents.text(
  'Muted text',
  textStyle: OsmeaTextStyle.captionMedium(context),
  color: OsmeaColors.steel,
)

// ❌ Bad - Using standard Colors
Text(
  'Primary text',
  style: TextStyle(color: Colors.blue),
)
```

#### 2. Primary Usage Pattern
```dart
// ✅ Good - Use OsmeaComponents.text with textStyle parameter
OsmeaComponents.text(
  'Page Title',
  textStyle: OsmeaTextStyle.headlineLarge(context),
  color: OsmeaColors.nordicBlue,
)

OsmeaComponents.text(
  'This is the main content text that explains the feature.',
  textStyle: OsmeaTextStyle.bodyMedium(context),
  textAlign: context.textLeft,
)

OsmeaComponents.text(
  'Form Field Label',
  textStyle: OsmeaTextStyle.labelMedium(context),
  fontWeight: context.medium,
)

// ❌ Bad - Using generic Text with manual styling
Text(
  'Page Title',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.w600,
    color: Colors.blue,
  ),
)
```

#### 3. Style Selection Patterns
```dart
// ✅ Good - Use OsmeaComponents.text with textStyle parameter
OsmeaComponents.text(
  'Custom styled text',
  textStyle: OsmeaTextStyle.headlineLarge(context),
  color: OsmeaColors.red,
  letterSpacing: context.letterSpacingWide,
)

// ✅ Good - Use extensions for additional styling
OsmeaComponents.text(
  'Styled text with extensions',
  textStyle: OsmeaTextStyle.bodyLarge(context),
  fontFamily: context.fontRoboto,
  fontWeight: context.medium,
  textAlign: context.textCenter,
)

// ❌ Bad - Manual TextStyle creation
Text(
  'Custom styled text',
  style: TextStyle(
    fontSize: 20,
    fontWeight: FontWeight.w500,
    color: Colors.red,
    letterSpacing: 0.5,
  ),
)
```

#### 4. Extension Usage Patterns
```dart
// ✅ Good - Use extensions for consistent styling
OsmeaComponents.text(
  'Styled text',
  textStyle: OsmeaTextStyle.bodyMedium(context),
  fontFamily: context.fontRoboto,
  fontWeight: context.medium,
  letterSpacing: context.letterSpacingNormal,
  wordSpacing: context.wordSpacingNormal,
  textAlign: context.textCenter,
  maxLines: context.maxLineTwo,
  overflow: context.ellipsis,
)

// ❌ Bad - Hard-coded values
OsmeaComponents.text(
  'Styled text',
  textStyle: OsmeaTextStyle.bodyMedium(context),
  fontFamily: 'Roboto',
  fontWeight: FontWeight.w500,
  letterSpacing: 0.0,
  wordSpacing: 0.0,
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)
```

### 📱 Responsive Typography

#### 1. Dynamic Text Sizing
```dart
// ✅ Good - Responsive text with style selection
class ResponsiveText extends StatelessWidget {
  final String text;
  
  const ResponsiveText({required this.text});
  
  @override
  Widget build(BuildContext context) {
    final screenWidth = MediaQuery.of(context).size.width;
    
    OsmeaTextVariant style;
    if (screenWidth < 600) {
      style = OsmeaTextVariant.bodyMedium;
    } else if (screenWidth < 900) {
      style = OsmeaTextVariant.bodyLarge;
    } else {
      style = OsmeaTextVariant.headlineSmall;
    }
    
    return OsmeaComponents.text(
      text,
      textStyle: OsmeaTextStyle.fromVariant(context, style),
      textAlign: context.textCenter,
      letterSpacing: context.letterSpacingNormal,
    );
  }
}
```

#### 2. Mobile-First Typography
```dart
// ✅ Good - Mobile-first approach with proper scaling
Widget buildArticleText(BuildContext context) {
  return Column(
    crossAxisAlignment: context.crossStart,
    children: [
      OsmeaComponents.text(
        'Article Title',
        textStyle: OsmeaTextStyle.headlineLarge(context),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masterfabric-mobile/osmea](https://github.com/masterfabric-mobile/osmea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
