---
trigger: always_on
description: providesTags: ['Todo'],
---


# MobileLauncher LT - Development Rules & Guidelines

This document outlines the essential rules and guidelines for developing with the MobileLauncher LT boilerplate. Follow these rules to maintain consistency, scalability, and code quality across the project.

## 🏗️ Architecture Rules

### 1. Feature-First Structure
- **Always organize code by business features**, not technical layers
- Each feature must be self-contained with its own:
  - `components/` - Feature-specific UI components
  - `screens/` - Screen components
  - `hooks/` - Custom hooks for business logic
  - `store/` - Redux slice and selectors
  - `api/` - RTK Query endpoints
  - `services/` - Business logic services
  - `types/` - TypeScript type definitions
  - `index.ts` - Barrel exports

**Example:**
```
src/features/auth/
├── components/
│   ├── login-form.tsx
│   └── index.ts
├── screens/
│   ├── login-screen.tsx
│   └── index.ts
├── hooks/
│   ├── use-auth.ts
│   └── index.ts
├── store/
│   ├── auth-slice.ts
│   ├── auth-selector.ts
│   └── index.ts
├── api/
│   ├── auth.api.ts
│   └── index.ts
├── services/
│   ├── auth.service.ts
│   └── index.ts
├── types/
│   ├── index.ts
├── index.ts
```

### 2. Shared Resources
- **Extract common functionality** to global directories (`src/ui/`, `src/services/`, `src/utils/`)
- **Never duplicate code** - if used in 2+ features, move to shared
- **Use absolute imports** with `#root/` prefix for clean imports
- **Expo ecosystem**: use expo.dev ecosystem for packages and yarn to install packages.

**Example:**
```typescript
// ❌ Wrong - duplicated across features
// features/auth/components/button.tsx
// features/settings/components/button.tsx

// ✅ Correct - shared component
// src/ui/components/button.tsx
import { Button } from '#root/ui/components/button';

// ✅ Correct - shared utility
// src/utils/format-date.ts
import { formatDate } from '#root/utils/format-date';
```

## 🎨 UI Component Rules

### 3. Component Design
- **Use Restyle for all styling** - never use StyleSheet directly
- **Create type-safe components** with proper TypeScript interfaces
- **Export prop interfaces** for reusability
- **Use React.memo** for performance optimization
- **Follow single responsibility principle**
- **Memorzie Functions**: use useCallback for functions
- **Avoid inline functions**: extract functions outside render or use useCallback
- **Avoid inline styling**: use theme values and component variants instead

**Example:**
```typescript
// ✅ Correct - Performance optimized component
import React, { useCallback } from 'react';
import { createBox } from '@shopify/restyle';
import type { Theme } from '#root/ui/style/theme';

const StyledButton = createBox<Theme>();

export interface ButtonProps {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

const _Button = ({ title, onPress, variant = 'primary', disabled = false }: ButtonProps) => {
  const handlePress = useCallback(() => {
    if (!disabled) {
      onPress();
    }
  }, [onPress, disabled]);

  return (
    <StyledButton
      backgroundColor={variant === 'primary' ? 'primary' : 'secondary'}
      padding="md"
      borderRadius="md"
      onPress={handlePress}
      opacity={disabled ? 0.5 : 1}
    >
      <Text color="white" textAlign="center">
        {title}
      </Text>
    </StyledButton>
  );
};

export const Button = React.memo(_Button);
```

### 4. Theme Usage
- **Always use theme values** for colors, spacing, typography
- **Use borderRadius theme values**: `none`, `xs`, `sm`, `md`, `lg`, `xl`, `2xl`, `3xl`, `full`
- **Use "full" for circles** instead of calculating half width/height
- **Never hardcode values** in theme props

**Example:**
```typescript
// ❌ Wrong - hardcoded values
<Box 
  backgroundColor="#3B82F6" 
  padding={16} 
  borderRadius={8}
  width={50}
  height={50}
/>

// ✅ Correct - theme values
<Box 
  backgroundColor="primary" 
  padding="md" 
  borderRadius="md"
  width={50}
  height={50}
  borderRadius="full" // For circles
/>
```

### 5. Component Variants
- **Create multiple variants** for each component (size, type, state)
- **Use consistent naming**: `buttonTypeVariant`, `buttonSizeVariant`
- **Define variants in theme** using Restyle's variant system

**Example:**
```typescript
// Theme definition
export const buttonVariants = createVariant<Theme, 'buttonVariants', 'variant'>({
  property: 'variant',
  themeKey: 'buttonVariants',
});

// Component usage
<Button 
  title="Save" 
  buttonTypeVariant="primary" 
  buttonSizeVariant="large"
  onPress={handleSave} 
/>
```

## 🔄 State Management Rules

### 6. Redux Patterns
- **Use Redux Toolkit** for all state management
- **Create feature slices** with proper action creators
- **Use createSelector** for derived state
- **Keep state normalized** and flat
- **Use RTK Query** for all API calls

**Example:**
```typescript
// Redux slice
const todosSlice = createSlice({
  name: 'todos',
  initialState: { todos: [], loading: false },
  reducers: {
    addTodo: (state, action) => {
      state.todos.push(action.payload);
    },
    toggleTodo: (state, action) => {
      const todo = state.todos.find(t => t.id === action.payload);
      if (todo) todo.completed = !todo.completed;
    },
  },
});

// Selector
export const selectCompletedTodos = createSelector(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chohra-med/expo_boilerplate](https://github.com/chohra-med/expo_boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
