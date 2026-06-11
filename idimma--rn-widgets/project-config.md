---
trigger: always_on
description: **Package**: `@idimma/rn-widget`
---

# rn-widgets Codebase Analysis

**Package**: `@idimma/rn-widget`
**Version**: 0.2.0
**Analysis Date**: February 2026
**Status**: Refactored - All issues from v0.1.x addressed

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Component Inventory](#component-inventory)
4. [Dependencies Analysis](#dependencies-analysis)
5. [Security Concerns](#security-concerns)
6. [Code Quality Issues](#code-quality-issues)
7. [Deficiencies](#deficiencies)
8. [Tight Coupling Problems](#tight-coupling-problems)
9. [Recommendations](#recommendations)

---

## Overview

This is a React Native component library designed primarily for Expo-based projects. It provides a set of styled UI components with a shorthand prop system for rapid development.

### Strengths
- Intuitive shorthand props (`p`, `m`, `br`, `bw`, etc.)
- Theme support with light/dark modes
- Consistent styling API across components
- Good breadth of components

### Weaknesses
- Tight coupling to Expo ecosystem
- Zero test coverage
- Components crash without optional dependencies
- Multiple code quality issues

---

## Architecture

### Directory Structure

```
rn-widgets/
├── src/
│   ├── components/           # UI components
│   │   ├── Text/
│   │   ├── View.tsx
│   │   ├── Button.tsx
│   │   ├── Touch.tsx
│   │   ├── TextField.tsx
│   │   ├── Icon.tsx
│   │   ├── image/
│   │   ├── Spinner.tsx
│   │   ├── Tabs.tsx
│   │   ├── Empty.tsx
│   │   ├── KeyboardAvoidingView.tsx
│   │   ├── Select.js
│   │   ├── SelectInput.js
│   │   ├── Timeline/
│   │   └── WeekPicker/
│   ├── context/              # Theme provider
│   ├── helper/               # Utilities & types
│   │   ├── index.ts
│   │   ├── styles.view.ts
│   │   ├── styles.text.ts
│   │   ├── @types/          # Type definitions
│   │   ├── @type/           # DUPLICATE type definitions
│   │   └── data/
│   ├── common/              # Unexported components (dead code)
│   └── __tests__/           # Empty test directory
├── lib/                     # Build output
└── example/                 # Example app
```

### Export Strategy

All exports go through `src/index.tsx`:
- Multiple aliases for backward compatibility (e.g., `Touch`, `TouchOpacity`, `Pressable`, `Press`, `TouchableOpacity`)
- Barrel exports from helper modules
- Context provider exported as `WidgetProvider`

### Styling System

Uses two main functions:
- `viewStyler()` - Converts view props to React Native styles
- `flattenStyle()` - Converts text props to styles

Both consume theme colors from context via `useRnWidgetContext()`.

---

## Component Inventory

### Exported Components (11)

| Component | File | Purpose | Dependencies |
|-----------|------|---------|--------------|
| View | View.tsx | Container with animations, gradients, safe area | react-native-animatable, expo-linear-gradient, react-native-safe-area-context |
| Text | Text/index.tsx | Styled text | None (uses context) |
| Button | Button.tsx | Styled button | Touch, View |
| Touch | Touch.tsx | Pressable with haptics | expo-haptics |
| TextField | TextField.tsx | Advanced input | @expo/vector-icons, Select |
| Icon | Icon.tsx | Icon wrapper | @expo/vector-icons |
| Image | image/index.tsx | Smart image | expo-image, react-native-lightbox-v2 |
| Spinner | Spinner.tsx | Loading indicator | None |
| Empty | Empty.tsx | Empty state | Icon, Spinner |
| Tabs | Tabs.tsx | Tab navigation | None (uses context) |
| KeyboardAvoidingView | KeyboardAvoidingView.tsx | Keyboard handling | None |

### Unexported Components (Dead Code in `common/`)

- OTP.js
- Switch.js
- Modal.js
- Carousel.js
- Header.js
- DropDownSearch.js
- TabBar.js
- Container.js
- Row.js
- EmptyImage.js
- InputPhone.js
- LoaderIndicator.js
- NotificationHandler.js
- TextInputField.js
- BottomSheet.js

**Impact**: ~15+ components sitting unused, increasing bundle size.

---

## Dependencies Analysis

### Production Dependencies (3)

```json
{
  "react-native-animatable": "1.4.0",      // Animation library
  "react-native-lightbox-v2": "0.9.0",     // Image lightbox (⚠️ unmaintained since 2018)
  "react-native-raw-bottom-sheet": "3.0.0" // Bottom sheet for Select
}
```

### Peer Dependencies

| Dependency | Required | Used In |
|------------|----------|---------|
| react | Yes | All |
| react-native | Yes | All |
| react-native-safe-area-context | Yes | View, WidgetProvider |
| @expo/vector-icons | Optional* | Icon, TextField |
| expo-haptics | Optional* | Touch |
| expo-image | Optional* | Image |
| expo-linear-gradient | Optional* | View |
| react-native-gesture-handler | Optional | Not used |
| react-native-reanimated | Optional | Not used |

*Marked optional but **components crash without them**.

### Dev Dependencies of Concern

| Dependency | Issue |
|------------|-------|
| lodash@4.17.21 | Used at runtime for simple operations |
| react-native-size-matters@0.4.2 | Used at runtime but listed as dev dependency |

---

## Security Concerns

### 1. CRITICAL: External URL Execution
**Location**: `src/components/image/index.tsx:32`
```typescript
const generateRandomImageUrl = (width, height) =>
  `https://loremflickr.com/${width}/${height}/people?random=${Math.floor(Math.random() * 1000)}`;
```
**Risk**: Fetches images from external service without user consent or ability to disable.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Idimma/rn-widgets](https://github.com/Idimma/rn-widgets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
