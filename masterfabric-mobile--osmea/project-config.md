---
trigger: always_on
description: OSMEA color system - palette, semantic colors, theming and usage for Flutter
---


# OSMEA Color System - Cursor Rules

## 📦 Package Overview
OSMEA Color System provides a comprehensive color palette with semantic naming, Material Design integration, and consistent theming across Flutter applications. This system includes 50+ predefined colors organized into logical categories for maintainable design.

## 🎯 Workspace-wide conventions

### Primary style: black and white
Applications in this workspace should **prioritize a black-and-white (grayscale) style**. Use the gray palette (`OsmeaColors.paperWhite` through `OsmeaColors.void_`) for backgrounds, text, borders, and primary UI. Add color sparingly for emphasis, status, or primary actions.

### Orange instead of red (errors, warnings, accents)
Across **all apps**, use **orange tones** instead of red for error, warning, and accent colors.

- **Why orange:** Brand consistency; the UI uses an orange-heavy palette. This is a workspace-wide convention; apply it in every app.
- **Colors to use (OsmeaColors):**
  - **OsmeaColors.amberFlame** – error text, warning emphasis (strongest orange)
  - **OsmeaColors.sunsetGlow** – primary orange, buttons and accents
  - **OsmeaColors.goldenHour** – softer accent
  - **OsmeaColors.desertSand** – background or light accent
- **Do:** Use **OsmeaColors.amberFlame** or **OsmeaColors.sunsetGlow** for error messages, warning labels, and destructive action emphasis. Prefer the same orange palette in new components.
- **Don't:** Do **not** use **OsmeaColors.red** for error messages, warning labels, or destructive accents.

```dart
// Don't
color: OsmeaColors.red,

// Do
color: OsmeaColors.amberFlame,
```

## 🎯 Development Guidelines

### 📁 File Structure
```
packages/components/lib/src/styles/
└── colors.dart    # Main color system file
```

### 🎨 Color Categories

#### 1. 🌈 Core Colors
- `white`, `black`, `transparent` - Basic color constants
- Foundation colors for all UI elements

#### 2. ⚫ Gray Palette (MaterialColor)
- `paperWhite` to `void_` - 12 semantic gray shades
- From lightest (25) to darkest (950)
- Consistent naming: `grayMaterial[shade]`

#### 3. 🔵 Blue Palette (MaterialColor)
- `crystalBay` to `abyss` - 9 blue shades
- Primary brand colors with Nordic theme
- Consistent naming: `blueMaterial[shade]`

#### 4. 🟠 Orange Palette (MaterialColor)
- `desertSand` to `amberFlame` - 4 orange shades
- Accent colors for highlights and warnings
- Consistent naming: `orangeMaterial[shade]`

#### 5. 🟢 Green Palette (MaterialColor)
- `springLeaf` to `pineGrove` - 4 green shades
- Success states and positive actions
- Consistent naming: `greenMaterial[shade]`

#### 6. 🌗 Shadow System
- `shadowLight`, `shadowDark` - Elevation shadows
- Consistent shadow colors across themes

#### 7. 📦 Material Colors
- `grey`, `green`, `red`, `orange` - Flutter Material colors
- Standard Material Design color references

### 🔧 Usage Rules

#### 1. Semantic Color Usage
```dart
// ✅ Good - Use semantic color names
Container(
  color: OsmeaColors.nordicBlue,
  child: Text(
    'Primary Action',
    style: TextStyle(color: OsmeaColors.white),
  ),
)

Card(
  color: OsmeaColors.paperWhite,
  child: Text(
    'Content Card',
    style: TextStyle(color: OsmeaColors.thunder),
  ),
)

// ❌ Bad - Hard-coded color values
Container(
  color: Color(0xFF1B80BF),
  child: Text(
    'Primary Action',
    style: TextStyle(color: Color(0xFFFFFFFF)),
  ),
)
```

#### 2. Gray Palette Usage
```dart
// ✅ Good - Use appropriate gray shades
Container(
  decoration: BoxDecoration(
    color: OsmeaColors.snow,        // Light background
    border: Border.all(
      color: OsmeaColors.silver,    // Light border
      width: 1,
    ),
  ),
  child: Text(
    'Content',
    style: TextStyle(
      color: OsmeaColors.thunder,   // Dark text
    ),
  ),
)

// Status-based gray usage
Text(
  'Disabled text',
  style: TextStyle(
    color: OsmeaColors.steel,       // Muted text
  ),
)

Text(
  'Secondary text',
  style: TextStyle(
    color: OsmeaColors.slate,       // Medium text
  ),
)

// ❌ Bad - Random gray values
Container(
  color: Colors.grey[100],
  child: Text(
    'Content',
    style: TextStyle(color: Colors.grey[700]),
  ),
)
```

#### 3. Brand Color Usage
```dart
// ✅ Good - Use brand colors consistently
ElevatedButton(
  style: ElevatedButton.styleFrom(
    backgroundColor: OsmeaColors.nordicBlue,
    foregroundColor: OsmeaColors.white,
  ),
  onPressed: () {},
  child: Text('Primary Action'),
)

OutlinedButton(
  style: OutlinedButton.styleFrom(
    foregroundColor: OsmeaColors.nordicBlue,
    side: BorderSide(color: OsmeaColors.nordicBlue),
  ),
  onPressed: () {},
  child: Text('Secondary Action'),
)

// Accent colors for highlights
Container(
  decoration: BoxDecoration(
    color: OsmeaColors.sunsetGlow.withOpacity(0.1),
    border: Border.all(color: OsmeaColors.sunsetGlow),
  ),
  child: Text('Warning content'),
)

// ❌ Bad - Inconsistent brand colors
ElevatedButton(
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.blue[600],
    foregroundColor: Colors.white,
  ),
  onPressed: () {},
  child: Text('Primary Action'),
)
```

#### 4. Status Color Usage
```dart
// ✅ Good - Use appropriate status colors
Container(
  decoration: BoxDecoration(
    color: OsmeaColors.springLeaf,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masterfabric-mobile/osmea](https://github.com/masterfabric-mobile/osmea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
