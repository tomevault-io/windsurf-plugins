---
trigger: always_on
description: Below are the standards and best practices for developing components in the Backpack iOS library. These standards ensure that all components are accessible, well-designed, thoroughly tested, and properly documented. Following these guidelines will help maintain consistency across the design system and provide the best experience for developers and end users.
---

# Backpack iOS Component Standards

Below are the standards and best practices for developing components in the Backpack iOS library. These standards ensure that all components are accessible, well-designed, thoroughly tested, and properly documented. Following these guidelines will help maintain consistency across the design system and provide the best experience for developers and end users.

## Technology Stack & Architecture

### SwiftUI First Approach
- SwiftUI is the standard for all new component development in Backpack iOS
- New components should be developed using SwiftUI and located in Backpack-SwiftUI/
- UIKit components should be maintained for backward compatibility, but new development should prioritize SwiftUI implementations
- Only update UIKit components when absolutely necessary - encourage users to migrate to SwiftUI component counterpart

### Project Structure
Components should follow the established folder structure:

Backpack-SwiftUI/
├── ComponentName/
│   ├── Classes/
│   │   ├── BPKComponentName.swift
│   │   ├── BPKComponentName+Style.swift (if applicable)
│   │   └── Supporting files...
│   ├── README.md
│   └── Tests/ (if component-specific tests exist)

## Accessibility Standards

Accessibility is fundamental to all Backpack components. Components must be usable by everyone, following WCAG 2.2 AA guidelines.

### Requirements

1. **VoiceOver Support**
   - All components must work properly with VoiceOver
   - Interactive elements must have appropriate accessibility labels
   - State changes must be announced to screen readers
   - Use accessibilityElement(), accessibilityLabel(), accessibilityHint(), and accessibilityValue() appropriately

2. **Keyboard Navigation & Focus Management**
   - All interactive components must be keyboard accessible
   - Focus states must be clearly visible and follow system conventions
   - Tab order must be logical and follow visual layout
   - Use focusable() and accessibilityFocused() when needed

3. **Touch Target Size**
   - Interactive elements must have a minimum touch target size of 44pt × 44pt (following Apple's HIG)
   - Ensure sufficient spacing between interactive elements

4. **Color Contrast & Visual Accessibility**
   - Text must have a contrast ratio of at least 4.5:1 against its background
   - UI controls must have a contrast ratio of at least 3:1
   - Components must work properly in both light and dark modes
   - Support high contrast accessibility settings

5. **Dynamic Type Support**
   - Components must support Dynamic Type scaling
   - Text should scale appropriately with user's font size preferences
   - Layouts must adapt gracefully to larger text sizes
   - Use `BPKDynamicLayout` and `BPKDynamicTypeLayout` when making flexible layout based on a size threshold.
   - Control the enablement of Dynamic type via BPKFont.enableDynamicType flag. Default it to sizeCategory(.large) when dynamic type is not enabled. 

6. **State Communication**
   - Error states must be communicated both visually and programmatically
   - Loading states must be properly indicated and announced
   - Selection states must be clearly visible and accessible

## API Design Principles

SwiftUI components should follow established patterns and conventions for consistency and ease of use.

### Requirements

1. **SwiftUI Best Practices**
   - Follow SwiftUI naming conventions and patterns
   - Use View protocol for all components
   - Implement proper state management with @State, @ObservedObject, etc.
   - Use ViewModifier for reusable styling behaviors
   - Provide view builder closures for customizable content when appropriate

2. **Backpack Conventions**
   - All components must start with BPK prefix (e.g., BPKBadge, BPKButton)
   - Use associated types and enums for configuration (e.g., BPKBadge.Style)
   - Style modifiers should return the same type (e.g., func badgeStyle(_ style: BPKBadge.Style) -> BPKBadge)
   - Follow the existing patterns established in other components

3. **Parameter Design**
   - Required parameters should come first in initializers
   - Optional parameters should have sensible defaults
   - Use clear, descriptive parameter names
   - Group related parameters logically

4. **Consistency**
   - Similar components should have similar APIs
   - Use consistent naming across components (e.g., style, size, variant)
   - Follow established patterns for colors, spacing, and typography

5. **Flexibility & Customization**
   - Provide style variants through enums (e.g., .normal, .strong, .success)
   - Allow customization through modifiers rather than constructor parameters
   - Support both simple and advanced use cases

## Design Token Integration

Proper use of design tokens ensures consistency and supports theming across the design system.

### Requirements

1. **Color Tokens**
   - Use BPKColor tokens exclusively (never hardcoded colors)
   - Support both light and dark mode automatically

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Skyscanner/backpack-ios](https://github.com/Skyscanner/backpack-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
