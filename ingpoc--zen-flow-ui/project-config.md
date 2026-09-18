---
trigger: always_on
description: Every component should follow the zen-flow-ui architecture pattern:
---

# Component Development Standards for Zen Flow UI

## Architecture Principles

### Component Structure
Every component should follow the zen-flow-ui architecture pattern:

```
src/components/ui/ComponentName/
├── index.ts              # Export barrel
├── ComponentName.tsx     # Main component
├── ComponentName.test.tsx # Unit tests
├── ComponentName.stories.tsx # Storybook stories (future)
└── types.ts              # Component-specific types
```

### File Organization Guidelines
- **Main Component**: Business logic and rendering
- **Types**: Interface definitions and type exports
- **Tests**: Unit tests with accessibility checks
- **Index**: Clean export interface

## Component Development Template

### Base Component Structure
```tsx
// src/components/ui/ComponentName/ComponentName.tsx
import React, { forwardRef } from 'react';
import { cn } from '@/lib/utils';
import { cva, type VariantProps } from 'class-variance-authority';
import { useZenAnimation } from '@/hooks/useZenAnimation';

// Component variants using CVA
const componentVariants = cva(
  // Base styles using design tokens
  "relative inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-zen-water focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-zen-paper text-zen-ink hover:bg-zen-cloud",
        primary: "bg-zen-water text-zen-light hover:bg-zen-water/90",
        secondary: "bg-zen-mist text-zen-light hover:bg-zen-mist/90",
        destructive: "bg-zen-accent text-zen-light hover:bg-zen-accent/90",
        outline: "border border-zen-cloud bg-transparent hover:bg-zen-cloud hover:text-zen-ink",
        ghost: "hover:bg-zen-cloud hover:text-zen-ink",
        link: "text-zen-water underline-offset-4 hover:underline",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

// Component interface
interface ComponentNameProps
  extends React.HTMLAttributes<HTMLElement>,
    VariantProps<typeof componentVariants> {
  // Custom props
  asChild?: boolean;
  loading?: boolean;
}

// Main component with forwardRef for proper ref handling
const ComponentName = forwardRef<HTMLElement, ComponentNameProps>(
  ({ className, variant, size, asChild = false, loading, children, ...props }, ref) => {
    const { animate } = useZenAnimation();
    
    // Component logic here
    
    return (
      <element
        ref={ref}
        className={cn(componentVariants({ variant, size, className }))}
        {...props}
      >
        {children}
      </element>
    );
  }
);

ComponentName.displayName = "ComponentName";

export { ComponentName, type ComponentNameProps };
```

### Type Definitions
```tsx
// src/components/ui/ComponentName/types.ts
import type { VariantProps } from 'class-variance-authority';
import type { componentVariants } from './ComponentName';

export interface ComponentNameProps
  extends React.HTMLAttributes<HTMLElement>,
    VariantProps<typeof componentVariants> {
  asChild?: boolean;
  loading?: boolean;
}

export type ComponentNameVariant = VariantProps<typeof componentVariants>['variant'];
export type ComponentNameSize = VariantProps<typeof componentVariants>['size'];
```

### Export Barrel
```tsx
// src/components/ui/ComponentName/index.ts
export { ComponentName, type ComponentNameProps } from './ComponentName';
export type { ComponentNameVariant, ComponentNameSize } from './types';
```

## Code Quality Standards

### TypeScript Requirements
- **Strict Mode**: All components must compile with TypeScript strict mode
- **Proper Interfaces**: Use proper interface inheritance from HTML attributes
- **Generic Support**: Support generic props where appropriate
- **No Any Types**: Avoid `any` types, use proper typing

### Accessibility Requirements
- **ARIA Labels**: All interactive elements need proper ARIA attributes
- **Keyboard Navigation**: Full keyboard support for all interactions
- **Focus Management**: Proper focus indicators and management
- **Screen Reader Support**: Semantic HTML and ARIA descriptions
- **Color Contrast**: Minimum 4.5:1 contrast ratio

### Performance Requirements
- **No Inline Styles**: Use CSS classes and design tokens only
- **Memoization**: Use React.memo for expensive components
- **Lazy Loading**: Support lazy loading where appropriate
- **Tree Shaking**: Ensure components are tree-shakable

## Animation Integration

### GSAP Integration Pattern
```tsx
import { useZenAnimation } from '@/hooks/useZenAnimation';
import { ZEN_TIMING, ZEN_EASING } from '@/lib/animations';

const Component = () => {
  const { animate } = useZenAnimation();
  const elementRef = useRef<HTMLElement>(null);
  
  const handleInteraction = () => {
    animate(elementRef.current, {
      scale: 1.02,
      duration: ZEN_TIMING.fast,
      ease: ZEN_EASING.out
    });
  };
  
  return (
    <element
      ref={elementRef}
      onMouseEnter={handleInteraction}
    >
      Content
    </element>
  );
};
```

### Motion Preferences

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ingpoc/zen-flow-ui](https://github.com/ingpoc/zen-flow-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
