---
trigger: always_on
description: description: "Flutter app_ui Package Rule (Atomic Design Pattern)"
---


description: "Flutter app_ui Package Rule (Atomic Design Pattern)"
alwaysApply: true

## Overview
This rule enforces the use of the `app_ui` package components following the Atomic Design Pattern. The package provides consistent theming, spacing, and reusable components across the Flutter Launchpad project.

## 1. app_translations Package 📦

### Overview
The `app_translations` package manages localization in the application using the **slang** package. It provides type-safe, auto-generated translations for consistent internationalization across the Flutter Launchpad project.

### Implementation Rules

#### ✅ ALWAYS Use context.t for Text
**Correct Pattern:**
```dart
// Use generated translations
AppText.medium(text: context.t.welcome)
AppButton(text: context.t.submit, onPressed: () {})
```

#### ❌ NEVER Use Hardcoded Strings
**Avoid these patterns:**
```dart
// DON'T DO THIS
AppText.medium(text: "Welcome")
AppButton(text: "Submit", onPressed: () {})
```

### Adding New Translations

#### Step 1: Add Key-Value Pairs
Add translations to JSON files in the `i18n` folder within `app_translations` package:

**English (`en.json`):**
```json
{
  "login": "Login Screen",
  "welcome": "Welcome to the app",
  "submit": "Submit",
  "cancel": "Cancel",
  "loading": "Loading...",
}
```

**Other languages (e.g., `es.json`):**
```json
{
  "login": "Pantalla de Inicio de Sesión",
  "welcome": "Bienvenido a la aplicación",
  "submit": "Enviar",
  "cancel": "Cancelar",
  "loading": "Cargando...",
}
```

#### Step 2: Generate Code
After adding key-value pairs, run the generation command:
```bash
melos run locale-gen
```

#### Step 3: Use in Code
```dart

// In AppText widget
AppText.medium(text: context.t.welcome)

// In buttons
AppButton(
  text: context.t.submit,
  onPressed: () {},
)

AppText.small(text: context.t.error.validation)
```


### Troubleshooting

#### Translation Not Found
1. Verify key exists in JSON files
2. Check spelling and nested structure
3. Run `melos run locale-gen` to regenerate
4. Restart IDE/hot restart app

#### Generated Code Issues
1. Ensure JSON syntax is valid
2. Check for duplicate keys
3. Verify slang package configuration
4. Clean and rebuild project

## Package Structure
The `app_ui` package is organized using **Atomic Design Pattern**:
- 🎨 **App Themes** - Color schemes and typography
- 🔤 **Fonts** - Custom font configurations
- 📁 **Assets Storage** - Images, icons, and other assets
- 🧩 **Common Widgets** - Reusable UI components
- 🛠️ **Generated Files** - Auto-generated asset and theme files

## Atomic Design Levels

### 🛰️ Atoms (Basic Building Blocks)

#### Spacing Rules
**❌ NEVER Use Raw SizedBox for Spacing**
```dart
// DON'T DO THIS
const SizedBox(height: 8)
const SizedBox(width: 16)
const SizedBox(height: 24, width: 32)
```

**✅ ALWAYS Use VSpace and HSpace**
```dart
// DO THIS - Vertical spacing
VSpace.xsmall()    // Extra small vertical space
VSpace.small()     // Small vertical space  
VSpace.medium()    // Medium vertical space
VSpace.large()     // Large vertical space
VSpace.xlarge()    // Extra large vertical space

// Horizontal spacing
HSpace.xsmall()    // Extra small horizontal space
HSpace.small()     // Small horizontal space
HSpace.medium()    // Medium horizontal space
HSpace.large()     // Large horizontal space
HSpace.xlarge()    // Extra large horizontal space
```

#### Other Atom-Level Components
```dart
// Border radius
AppBorderRadius.small
AppBorderRadius.medium
AppBorderRadius.large

// Padding/margins
Insets.small
Insets.medium
Insets.large

// Text components
AppText.small(text: "Content")
AppText.medium(text: "Content")
AppText.large(text: "Content")

// Loading indicators
AppLoadingIndicator()
```

### 🔵 Molecules (Component Combinations)

#### Button Usage Rules
**❌ NEVER Use Raw Material Buttons**
```dart
// DON'T DO THIS
ElevatedButton(
  onPressed: () {},
  child: Text("Login"),
)

TextButton(
  onPressed: () {},
  child: Text("Cancel"),
)
```

**✅ ALWAYS Use AppButton**
```dart
// DO THIS - Basic button
AppButton(
  text: context.t.login,
  onPressed: () {},
)

// Expanded button
AppButton(
  text: context.t.submit,
  onPressed: () {},
  isExpanded: true,
)

// Disabled button
AppButton(
  text: context.t.save,
  onPressed: () {},
  isEnabled: false,
)

// Button variants
AppButton.secondary(
  text: context.t.cancel,
  onPressed: () {},
)

AppButton.outline(
  text: context.t.edit,
  onPressed: () {},
)

AppButton.text(
  text: context.t.skip,
  onPressed: () {},
)
```

## Spacing Implementation Patterns

### Column/Row Spacing
```dart
// Instead of multiple SizedBox widgets
Column(
  children: [
    Widget1(),
    VSpace.medium(),
    Widget2(),
    VSpace.small(),
    Widget3(),
  ],
)

Row(
  children: [
    Widget1(),
    HSpace.large(),
    Widget2(),
    HSpace.medium(),
    Widget3(),
  ],
)
```

### Complex Layout Spacing
```dart
// Combining vertical and horizontal spacing
Container(
  padding: Insets.medium,
  child: Column(
    spacing : EdgeInsets.all(Insets.small8),
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      AppText.large(text: context.t.title),
      VSpace.small(),
      AppText.medium(text: context.t.description),
      VSpace.large(),
      Row(
        children: [
          AppButton(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [7span/flutter-dashing-kit](https://github.com/7span/flutter-dashing-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
