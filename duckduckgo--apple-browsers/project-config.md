---
trigger: always_on
description: The DuckDuckGo iOS design system is implemented through **DesignResourcesKit (DRK)**, a shared Swift package that contains our design tokens, type styles, colors, and design system elements.
---


# DuckDuckGo iOS Design System & DesignResourcesKit (DRK)

## Overview

The DuckDuckGo iOS design system is implemented through **DesignResourcesKit (DRK)**, a shared Swift package that contains our design tokens, type styles, colors, and design system elements.

**Repository**: [https://github.com/duckduckgo/DesignResourcesKit](https://github.com/duckduckgo/DesignResourcesKit)

**Figma Designs**: [🖱️ iOS & iPadOS Components](https://www.figma.com/file/GzGKD6gR24AHoUqVykX1ah/%F0%9F%93%B1-iOS-%26-iPadOS-Components?type=design&node-id=3938%3A23329&mode=design&t=0fuiNF84nnV5zExC-1)

### What DRK Contains

✅ **Currently Included**:
- **Type styles and typography** (based on system styles)
- **Semantic color system** (with light/dark mode support)
- **Design tokens and foundations**

🔄 **Future Expansion**:
- **Reusable components** (when patterns emerge)
- **Advanced interaction patterns**

❌ **Not Included**:
- **Icons** (remain in iOS app directly for now)

## ⚠️ Critical Rule: Don't Break the Design System

> **If you take only one thing away from this documentation**: 
> **Don't add new colors or type styles outside of the design system without reading the guidelines below.**

Breaking the design system:
- **Undermines consistency** across the app
- **Creates maintenance debt** with scattered styles
- **Breaks accessibility** features like dynamic type
- **Fragments the user experience**

## Typography System

### Philosophy

Our typography system is **based on system styles** rather than hardcoded sizes. This ensures:
- **Automatic dynamic type support** for accessibility
- **Consistent scaling** across different user preferences
- **Platform-appropriate styling** that feels native

### UIKit Usage

DRK defines **static functions on UIFont** for all typography.

**Example:** See [uikit-typography-usage.swift](design-system-designresourceskit/uikit-typography-usage.swift)

#### Available Typography Styles

**Example:** See [uikit-typography-styles.swift](design-system-designresourceskit/uikit-typography-styles.swift)

#### Best Practices for UIKit

**Example:** See [uikit-typography-best-practices.swift](design-system-designresourceskit/uikit-typography-best-practices.swift)

### SwiftUI Usage

DRK provides **view modifiers and extensions** for SwiftUI that should be used instead of direct font access.

**Example:** See [swiftui-typography-usage.swift](design-system-designresourceskit/swiftui-typography-usage.swift)

#### Available SwiftUI Typography Modifiers

**Example:** See [swiftui-typography-modifiers.swift](design-system-designresourceskit/swiftui-typography-modifiers.swift)

#### SwiftUI Code Review Guidelines

**When reviewing PRs**: Look for `.font()` usage as a red flag.

**Example:** See [swiftui-code-review-red-flags.swift](design-system-designresourceskit/swiftui-code-review-red-flags.swift)

### Emergency Escape Hatch (Avoid!)

**For legacy layout fixes only**: If you absolutely must disable dynamic type, there's a deliberately obtusely named function:

```swift
// ❌ LAST RESORT: Only for fixing legacy layouts
let fixedFont = UIFont.daxFontOutsideOfTheDesignSystemToFixLegacyLayoutBreakage()
```

**Important Notes**:
- This function **may not exist** in current DRK versions
- If you need it, you must **revert the commit** that removed it: [Commit 971979d](https://github.com/duckduckgo/DesignResourcesKit/pull/1/commits/971979d3dcd95567b9812b800eb22ab1611ce3a5)
- This is **deliberately annoying** to discourage usage
- **Always prefer** fixing the layout to support dynamic type instead

## Color System

### Semantic Color Approach

Our color system uses **semantic naming** rather than literal colors (e.g., "primary text" instead of "black"). This enables:
- **Automatic dark mode support**
- **Future theme flexibility**
- **Accessibility compliance**
- **Consistent visual hierarchy**

### Color Categories

#### Text Colors
**UIKit Example:** See [colors-text-uikit.swift](design-system-designresourceskit/colors-text-uikit.swift)

**SwiftUI Example:** See [colors-text-swiftui.swift](design-system-designresourceskit/colors-text-swiftui.swift)

#### Background Colors
**Example:** See [colors-background.swift](design-system-designresourceskit/colors-background.swift)

#### Control Colors
```swift
// UIKit
button.backgroundColor = UIColor(designSystemColor: .controlsFillPrimary)
button.backgroundColor = UIColor(designSystemColor: .controlsFillSecondary)

// SwiftUI
Button("Action") { }
    .foregroundColor(Color(designSystemColor: .controlsFillPrimary))
    .background(Color(designSystemColor: .controlsFillSecondary))
```

#### Button-Specific Colors
```swift
// UIKit
primaryButton.backgroundColor = UIColor(designSystemColor: .buttonPrimaryBackground)
primaryButton.setTitleColor(UIColor(designSystemColor: .buttonPrimaryText), for: .normal)

secondaryButton.backgroundColor = UIColor(designSystemColor: .buttonSecondaryBackground)
secondaryButton.setTitleColor(UIColor(designSystemColor: .buttonSecondaryText), for: .normal)

// SwiftUI
Button("Primary Action") { }
    .foregroundColor(Color(designSystemColor: .buttonPrimaryText))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
