---
trigger: always_on
description: React best practices and patterns for this codebase
---


# React Development Guidelines

## Core Principles

1. **Function Components Only**: No class components - use hooks for state and lifecycle
2. **TypeScript First**: Every component, prop, and hook must be properly typed
3. **Composition Over Inheritance**: Build complex UIs from simple, composable pieces
4. **Performance by Default**: Consider performance implications in initial implementation
5. **Accessibility Always**: Every interactive element must be keyboard and screen reader accessible

## Component Architecture

### File Structure

```
src/
├── components/
│   ├── ui/           # Reusable UI components (Button, Input, Card)
│   ├── layout/       # Layout components (Header, Footer, Sidebar)
│   └── features/     # Feature-specific components
├── hooks/            # Custom React hooks
├── utils/            # Utility functions
├── types/            # Shared TypeScript types
└── assets/           # Images, fonts, etc.
```

### Component Organization

```typescript
// 1. Imports (in order)
import { useState, useEffect, type FC } from 'react';
import { z } from 'zod';
import clsx from 'clsx';

// 2. Type definitions
interface ComponentProps {
  // Props interface
}

// 3. Schema definitions (if needed)
const PropsSchema = z.object({
  // Validation schema
});

// 4. Component definition
export const Component: FC<ComponentProps> = (props) => {
  // 5. Hooks
  const [state, setState] = useState();

  // 6. Event handlers
  const handleClick = () => {};

  // 7. Effects
  useEffect(() => {}, []);

  // 8. Render
  return <div />;
};

// 9. Display name (for debugging)
Component.displayName = 'Component';
```

## State Management Patterns

### Local State

```typescript
// Simple state for UI-only concerns
const [isOpen, setIsOpen] = useState(false);

// Complex state with reducer for business logic
const [state, dispatch] = useReducer(reducer, initialState);
```

### Lifted State

```typescript
// Lift state to lowest common ancestor
export function Parent() {
  const [sharedState, setSharedState] = useState();

  return (
    <>
      <ChildA state={sharedState} />
      <ChildB onUpdate={setSharedState} />
    </>
  );
}
```

### Global State (Context)

```typescript
// Create context with proper typing
const StateContext = createContext<StateValue | undefined>(undefined);

// Provider with value memoization
export const StateProvider: FC<{ children: ReactNode }> = ({ children }) => {
  const [state, setState] = useState();

  const value = useMemo(
    () => ({ state, setState }),
    [state]
  );

  return <StateContext.Provider value={value}>{children}</StateContext.Provider>;
};

// Custom hook with error boundary
export const useAppState = () => {
  const context = useContext(StateContext);
  if (!context) {
    throw new Error('useAppState must be used within StateProvider');
  }
  return context;
};
```

## Performance Optimization

### Memoization Rules

```typescript
// Memo for expensive components
export const ExpensiveComponent = memo(({ data }: Props) => {
  return <ComplexVisualization data={data} />;
});

// useCallback for stable function references
const handleSubmit = useCallback((data: FormData) => {
  // Process data
}, [dependency]);

// useMemo for expensive calculations
const processedData = useMemo(() => {
  return expensiveCalculation(rawData);
}, [rawData]);
```

### Code Splitting

```typescript
// Route-based splitting
const Dashboard = lazy(() => import('@/pages/Dashboard'));

// Component-based splitting for heavy components
const HeavyChart = lazy(() => import('@/components/HeavyChart'));

// With loading boundary
<Suspense fallback={<Spinner />}>
  <HeavyChart data={data} />
</Suspense>
```

### List Optimization

```typescript
// Always use stable, unique keys
items.map((item) => <Item key={item.id} {...item} />)

// Virtualize long lists (100+ items)
import { FixedSizeList } from 'react-window';

<FixedSizeList
  height={600}
  itemCount={items.length}
  itemSize={50}
>
  {({ index, style }) => (
    <div style={style}>
      <Item {...items[index]} />
    </div>
  )}
</FixedSizeList>
```

## Styling with TailwindCSS

### Class Name Organization

```typescript
// Use clsx for conditional classes
import clsx from 'clsx';

<div
  className={clsx(
    // Base styles first
    'rounded-lg border p-4',
    // Conditional styles
    {
      'border-blue-500 bg-blue-50': isActive,
      'border-gray-300 bg-white': !isActive,
    },
    // Size variants
    {
      'text-sm': size === 'small',
      'text-base': size === 'medium',
      'text-lg': size === 'large',
    },
    // State styles
    'hover:shadow-md focus:outline-none focus:ring-2',
    // Override with className prop
    className
  )}
/>
```

### Component Variants with CVA

```typescript
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  // Base styles
  'inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700',
        secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
        ghost: 'hover:bg-gray-100',
      },
      size: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stevekinney/react-performance](https://github.com/stevekinney/react-performance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
