---
trigger: always_on
description: OSMEA sizer extensions - responsive, spacing, duration, padding, alignment
---


# OSMEA Sizer Extensions - Cursor Rules

## 📦 Package Overview
OSMEA Sizer Extensions provides comprehensive utilities for responsive design, spacing, animations, and UI layout in Flutter applications. This file contains 25+ extension categories with 800+ utility methods for consistent, responsive UI development.

## Import Statement
Always import the sizer extensions at the top of your files:
```dart
import 'package:osmea_components/osmea_components.dart';
```

## 🎯 Development Guidelines

### 📁 File Structure
```
packages/components/lib/src/utils/
└── sizer_extensions.dart    # Main sizer extensions file (858 lines)
```

### 🎨 Extension Categories Overview

#### 1. 📱 Core Sizer Extensions (`SizerExtension`)
- **Screen Dimensions**: `allHeight`, `allWidth`, `mediaQuery`
- **Text Scaling**: `textScaler`, `textScaleFactor`
- **Dynamic Sizing**: `dynamicWidth()`, `dynamicHeight()`
- **Value Scaling**: `lowValue`, `normalValue`, `mediumValue`, `highValue`
- **Radius Values**: `radiusNone`, `radiusLow`, `radiusNormal`, `radiusMedium`, `radiusHigh`
- **Scale Values**: `scaleLowValue`, `scaleMediumValue`, `scaleHighValue`
- **Utility Values**: `nullValue`, `infinity`
- **Dividers**: `divider1`, `divider2`, `divider3`

#### 2. ⭕ Border Radius Extensions (`BorderRadiusExtension`)
- **Basic Radius**: `borderRadiusZero`, `borderRadiusNone`, `borderRadiusLow`
- **Standard Radius**: `borderRadiusNormal`, `borderRadiusMedium`, `borderRadiusHigh`
- **Special Radius**: `borderRadiusMinStandard` (7), `borderRadiusMaxStandard` (13)

#### 3. 🔲 Border Width Extensions (`BorderWidthExtension`)
- **Standard Width**: `borderWidth` (0.5)

#### 4. ⏱️ Duration Extensions (`DurationExtension`)
- **Time Units**: `7.seconds`, `7.minutes`, `7.hours`, `7.days`, `7.weeks`
- **Micro Units**: `7.microseconds`, `7.milliseconds`
- **Zero Duration**: `7.zero`

#### 5. ⏰ Duration Values Extensions (`DurationValuesExtension`)
- **Basic Durations**: `durationZero`, `durationInstant`, `durationFast`, `durationQuick`
- **Standard Durations**: `durationNormal`, `durationMedium`, `durationSlow`, `durationVerySlow`
- **Long Durations**: `durationLong`, `durationVeryLong`
- **Animation Durations**: `animationShort`, `animationMedium`, `animationLong`, `animationSlow`
- **Delay Durations**: `delayShort`, `delayMedium`, `delayLong`
- **Timeout Durations**: `timeoutQuick`, `timeoutNormal`, `timeoutLong`, `timeoutVeryLong`

#### 6. 📐 Alignment Extensions (`AlignmentExtension`)
- **Position Alignments**: `topLeft`, `topCenter`, `topRight`, `centerLeft`, `center`, `centerRight`, `bottomLeft`, `bottomCenter`, `bottomRight`
- **Main Axis**: `start`, `end`, `centerMain`, `spaceBetween`, `spaceAround`, `spaceEvenly`
- **Cross Axis**: `crossStart`, `crossEnd`, `crossCenter`, `crossStretch`, `crossBaseline`
- **Wrap Alignment**: `wrapStart`, `wrapEnd`, `wrapCenter`, `wrapSpaceBetween`, `wrapSpaceAround`, `wrapSpaceEvenly`
- **Axis Size**: `min`, `max`
- **Text Direction**: `ltr`, `rtl`
- **Axis Direction**: `horizontal`, `vertical`

#### 7. 📦 Box Fit Extensions (`BoxFitExtension`)
- **Fit Types**: `fill`, `contain`, `cover`, `fitWidth`, `fitHeight`, `scaleDown`

#### 8. 📐 Stack Fit Extensions (`StackFitExtension`)
- **Stack Fits**: `loose`, `expand`, `passthrough`

#### 9. 📏 Spacing Extensions (`SpacingExtension`)
- **Dynamic Spacing**: `spacingZero`, `spacingLow`, `spacingNormal`, `spacingMedium`, `spacingHigh`
- **Fixed Spacing**: `spacing2`, `spacing4`, `spacing6`, `spacing8`, `spacing10`, `spacing12`, `spacing16`, `spacing20`, `spacing24`, `spacing32`, `spacing40`, `spacing48`, `spacing56`, `spacing64`

#### 10. 📐 Width Extensions (`WidthExtension`)
- **Dynamic Width**: `widthZero`, `widthLow`, `widthNormal`, `widthMedium`, `widthHigh`
- **Fixed Width**: `width1` to `width384` (1, 2, 4, 8, 12, 16, 20, 24, 32, 40, 48, 56, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 384)

#### 11. 📏 Height Extensions (`HeightExtension`)
- **Dynamic Height**: `heightZero`, `heightLow`, `heightNormal`, `heightMedium`, `heightHigh`
- **Fixed Height**: `height1` to `height640` (1, 2, 4, 8, 12, 16, 20, 24, 32, 40, 48, 56, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 384, 448, 512, 576, 640)

#### 12. 🔄 Wrap Cross Alignment Extensions (`WrapCrossAlignmentExtension`)
- **Wrap Cross**: `wrapCrossStart`, `wrapCrossEnd`, `wrapCrossCenter`

#### 13. 🎪 Animation Status Extensions (`AnimationStatusExtension`)
- **Animation States**: `dismissed`, `forward`, `reverse`, `completed`

#### 14. 🔧 Clip Behavior Extensions (`ClipBehaviorExtension`)
- **Clip Types**: `clipNone`, `clipHardEdge`, `clipAntiAlias`, `clipAntiAliasWithSaveLayer`

#### 15. ➖ Empty Widget Extensions (`EmptyWidget`)
- **Empty Boxes**: `emptySizedBox`, `emptySizedWidthBoxLow`, `emptySizedWidthBoxNormal`, `emptySizedWidthBoxHigh`
- **Height Boxes**: `emptySizedHeightBoxLow`, `emptySizedHeightBoxNormal`, `emptySizedHeightBoxMedium`, `emptySizedHeightBoxHigh`
- **Zero Boxes**: `emptySized`, `emptySizedWidthBoxZero`, `emptySizedHeightBoxZero`

#### 16. 🟫 Divider Extensions (`DividerX`)
- **Custom Divider**: `divider({Color? color})` - Creates divider with 13% indent/endIndent

#### 17. 🖼️ Image Asset Extensions (`ImageAssetX`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masterfabric-mobile/osmea](https://github.com/masterfabric-mobile/osmea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
