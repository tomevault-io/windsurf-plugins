---
trigger: always_on
description: This is a React Native/Expo admin mobile application for hotel management with order and booking management features. Built with TypeScript, React Navigation, and a bottom-tab navigation pattern.
---

# Nugen Atria Admin Mobile App - Cursor Rules

## Project Overview
This is a React Native/Expo admin mobile application for hotel management with order and booking management features. Built with TypeScript, React Navigation, and a bottom-tab navigation pattern.

---

## Tech Stack

### Core Framework
- **React Native**: 0.79.3
- **Expo**: ~53.0.11 (managed workflow)
- **React**: 19.0.0
- **TypeScript**: ~5.8.3 (strict mode enabled)

### Navigation
- **@react-navigation/native**: ^6.1.18
- **@react-navigation/native-stack**: ^6.11.0 (Stack navigation)
- **@react-navigation/bottom-tabs**: ^6.6.1 (Tab navigation)

### Development Tools
- **ESLint**: ^9.22.0 with `eslint-config-expo` and TypeScript plugin
- **Babel**: @babel/core ^7.25.2

### UI Libraries
- **@expo/vector-icons**: ^14.1.0 (MaterialIcons)
- **react-native-safe-area-context**: ~5.4.0
- **react-native-screens**: ~4.11.1

---

## Project Structure

### `/src/data`
- Mock data and fixtures for development
- Contains: `mockData.ts` with sample orders and bookings
- Use named exports for all data

### `/src/features`
- Feature-driven, domain-based organization
- Each feature directory (dashboard, orders, bookings, settings) contains:
  - `screens/` - Main screen components
  - `components/` - Feature-specific reusable components
- Features are independent and self-contained

### `/src/navigation`
- `RootNavigator.tsx` - Root stack navigator (tabs + detail screens)
- `MainTabsNavigator.tsx` - Bottom tab navigator
- `types.ts` - TypeScript param lists for type-safe navigation

### `/src/shared`
- `components/` - Shared components used across features
- `theme/` - Design tokens (colors, typography, spacing)

### `/src/types`
- `domain.ts` - Shared domain types and interfaces
- Contains: Order, Booking, OrderStatus types

---

## TypeScript Conventions

### Configuration
- Strict mode enabled in `tsconfig.json`
- Path alias: `@/*` maps to `src/*`
- Use for clean cross-feature imports

### Type Definitions
- Use `interface` for component props and domain objects
- Use `type` for unions, intersections, and param lists
- Export types with `export type` or `export interface`

### Component Props
```typescript
interface ComponentNameProps {
  item: DomainType;
  onPress: () => void;
}

export function ComponentName({ item, onPress }: ComponentNameProps) {
  // Implementation
}
```

### Navigation Types
```typescript
// Define in src/navigation/types.ts
export type RootStackParamList = {
  ScreenName: { param1: string };
  ScreenWithoutParams: undefined;
};

// Use in components
type NavigationProp = NativeStackNavigationProp<RootStackParamList>;
const navigation = useNavigation<NavigationProp>();
```

---

## Component Patterns

### Functional Components
- Always use named function declarations
- Never use default exports for components
- Use arrow functions for callbacks and handlers

```typescript
// ✅ Correct
export function MyComponent() { ... }

// ❌ Incorrect
export default function MyComponent() { ... }
const MyComponent = () => { ... }
```

### Screen Components
- Name with `Screen` suffix (e.g., `OrdersScreen.tsx`)
- Wrap content in `<ScreenContainer>` for consistent layout
- Structure: imports → component → styles

```typescript
import React from 'react';
import { StyleSheet, Text } from 'react-native';
import { ScreenContainer } from '@/shared/components/ScreenContainer';
import { colors } from '@/shared/theme/colors';

export function ExampleScreen() {
  return (
    <ScreenContainer>
      <Text style={styles.title}>Title</Text>
    </ScreenContainer>
  );
}

const styles = StyleSheet.create({
  title: {
    fontSize: 24,
    fontWeight: '700',
    color: colors.text
  }
});
```

### Reusable Components
- Define props interface above component
- Export component with named export
- Colocate styles at the bottom

```typescript
interface CardProps {
  item: DataType;
  onPress: () => void;
}

export function Card({ item, onPress }: CardProps) {
  return (/* JSX */);
}

const styles = StyleSheet.create({
  // Styles
});
```

---

## Styling Guidelines

### Theme Colors
Always use the centralized theme colors from `@/shared/theme/colors`:
```typescript
import { colors } from '@/shared/theme/colors';

// Available colors:
colors.primary      // #1E40AF - Primary brand blue
colors.background   // #F8FAFC - Light background
colors.card         // #FFFFFF - Card/surface background
colors.text         // #0F172A - Primary text
colors.muted        // #64748B - Secondary/muted text
colors.success      // #15803D - Success indicators
colors.border       // #E2E8F0 - Borders
```

### StyleSheet Pattern
- Use `StyleSheet.create()` for all styles
- Colocate styles at the bottom of the component file
- No external stylesheet files

### Standard Spacing
- Screen padding: `16`
- Card padding: `14`
- Gap between elements: `12`
- Border radius for cards: `14`
- Border width: `1`

### Font Weights
- Bold: `'700'`
- Semi-bold: `'600'`
- Regular: default (no specification needed)

### Card Pattern
```typescript
const styles = StyleSheet.create({
  card: {
    backgroundColor: colors.card,
    borderRadius: 14,
    padding: 14,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Deepanshu7-bit/nugen-atria-admin-mobile-app](https://github.com/Deepanshu7-bit/nugen-atria-admin-mobile-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
