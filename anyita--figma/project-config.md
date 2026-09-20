---
trigger: always_on
description: This is an Expo-based React Native application with TypeScript support, utilizing:
---

# AGENTS.md - React Native Development Specification

## Project Overview

This is an Expo-based React Native application with TypeScript support, utilizing:

- **React Native**: 0.79.6
- **React**: 19.0.0
- **Expo**: ~53.0.22
- **TypeScript**: ~5.8.3
- **Expo Router**: File-based routing
- **Theme System**: Light/Dark mode support

## Directory Structure

```
/
├── app/                    # File-based routing (Expo Router)
│   ├── (tabs)/            # Tab-based screens
│   ├── _layout.tsx        # Root layout
│   └── +not-found.tsx     # 404 page
├── components/            # Reusable components
│   ├── ui/               # Low-level UI primitives
│   └── [ComponentName].tsx
├── constants/            # App constants and configuration
├── hooks/               # Custom React hooks
├── assets/             # Static assets (images, fonts)
└── scripts/           # Build and utility scripts
```

## Component Architecture

### 1. Component Classification

**Theme Components** (Themed*)
- Components that support light/dark themes
- Example: `ThemedText`, `ThemedView`
- Must extend base component props
- Must support `lightColor` and `darkColor` props

**UI Primitives** (components/ui/)
- Platform-specific implementations allowed
- File naming: `ComponentName.tsx` + `ComponentName.ios.tsx`
- Low-level, reusable building blocks

**Feature Components**
- Business logic components
- Should compose UI primitives and theme components

### 2. Component Structure Template

```typescript
import React from 'react';
import { StyleSheet, ViewStyle, TextStyle } from 'react-native';
import { useThemeColor } from '@/hooks/useThemeColor';

// Props interface with clear typing
export interface ComponentNameProps {
  // Required props first
  title: string;

  // Optional props with defaults
  variant?: 'primary' | 'secondary';
  disabled?: boolean;

  // Style props
  style?: ViewStyle;

  // Theme support for themed components
  lightColor?: string;
  darkColor?: string;

  // Event handlers
  onPress?: () => void;

  // Children if applicable
  children?: React.ReactNode;
}

export function ComponentName({
  title,
  variant = 'primary',
  disabled = false,
  style,
  lightColor,
  darkColor,
  onPress,
  children,
}: ComponentNameProps) {
  const backgroundColor = useThemeColor(
    { light: lightColor, dark: darkColor },
    'background'
  );

  return (
    // Component JSX
  );
}

const styles = StyleSheet.create({
  container: {
    // Styles here
  },
});
```

## Coding Standards

### 1. TypeScript Guidelines

**Required Configurations:**
- Strict mode enabled
- Explicit return types for complex functions
- Interface over type for object definitions
- Props interfaces exported

```typescript
// ✅ Good
export interface UserProfileProps {
  user: User;
  onUpdate: (user: User) => Promise<void>;
}

// ❌ Avoid
export type UserProfileProps = {
  user: any;
  onUpdate: Function;
}
```

**Generic Typing:**
```typescript
// ✅ Generic components
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

export function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  // Implementation
}
```

### 2. File Naming Conventions

- **Components**: PascalCase (`UserProfile.tsx`)
- **Hooks**: camelCase with 'use' prefix (`useUserData.ts`)
- **Utils**: camelCase (`dateHelpers.ts`)
- **Constants**: PascalCase (`Colors.ts`, `Config.ts`)
- **Types**: PascalCase (`UserTypes.ts`)

### 3. Import Organization

```typescript
// 1. React imports
import React from 'react';
import { useState, useEffect } from 'react';

// 2. React Native imports
import { View, Text, StyleSheet } from 'react-native';

// 3. Third-party library imports
import { useRouter } from 'expo-router';

// 4. Local imports (using path aliases)
import { useThemeColor } from '@/hooks/useThemeColor';
import { Button } from '@/components/ui/Button';
import { Colors } from '@/constants/Colors';

// 5. Type-only imports last
import type { User } from '@/types/UserTypes';
```

### 4. Function and Variable Naming

- **Functions**: camelCase (`getUserProfile`)
- **Components**: PascalCase (`UserProfile`)
- **Constants**: SCREAMING_SNAKE_CASE (`API_ENDPOINTS`)
- **Boolean variables**: Prefixed with `is`, `has`, `can`, `should` (`isVisible`, `hasPermission`)

## Styling Guidelines

### 1. StyleSheet Usage

**Always use StyleSheet.create:**
```typescript
// ✅ Good
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#ffffff',
  },
});

// ❌ Avoid inline styles for static styles
<View style={{ flex: 1, backgroundColor: '#ffffff' }} />
```

### 2. Theme Integration

**Use theme hooks:**
```typescript
const backgroundColor = useThemeColor(
  { light: Colors.light.background, dark: Colors.dark.background },
  'background'
);
```

**Conditional styling:**
```typescript
const dynamicStyles = StyleSheet.create({
  button: {
    backgroundColor: disabled ? Colors.light.disabled : Colors.light.primary,
    opacity: disabled ? 0.6 : 1,
  },
});
```

### 3. Responsive Design

```typescript
import { Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');

const styles = StyleSheet.create({
  container: {
    width: width * 0.9,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Anyita/figma](https://github.com/Anyita/figma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
