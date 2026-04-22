---
trigger: always_on
description: UI component patterns for Radix UI, Tailwind CSS 4, and component architecture specific to lambda-curry/forms
---


You are an expert in Radix UI, Tailwind CSS 4, class-variance-authority (CVA), and accessible component design for the lambda-curry/forms component library.

# UI Component Patterns

## Core Principles
- Build accessible-first components using Radix UI primitives
- Use Tailwind CSS 4 with modern CSS features and performance optimizations
- Follow consistent component architecture patterns
- Implement variant patterns with class-variance-authority
- Follow the Slot pattern for polymorphic components
- Focus only on components that exist in the lambda-curry/forms library

## Project-Specific Component Stack
Based on your actual dependencies and components:

### Radix UI Components Used
- `@radix-ui/react-alert-dialog`
- `@radix-ui/react-avatar`
- `@radix-ui/react-checkbox`
- `@radix-ui/react-dialog`
- `@radix-ui/react-dropdown-menu`
- `@radix-ui/react-label`
- `@radix-ui/react-popover`
- `@radix-ui/react-radio-group`
- `@radix-ui/react-scroll-area`
- `@radix-ui/react-separator`
- `@radix-ui/react-slot`
- `@radix-ui/react-switch`
- `@radix-ui/react-tooltip`

### Existing UI Components
- Badge, Button, Checkbox Field
- Command, Data Table
- Date Picker, Dropdown Menu
- Form components, Label
- OTP Input, Popover
- Radio Group, Select
- Separator, Switch
- Table, Text Field, Textarea

## Tailwind CSS 4 Features & Patterns

### Modern CSS Custom Properties
```typescript
// ✅ Tailwind 4 - CSS custom properties for theming
const themeClasses = {
  // Use CSS custom properties directly
  colors: {
    primary: 'bg-primary text-primary-foreground',
    secondary: 'bg-secondary text-secondary-foreground',
    destructive: 'bg-destructive text-destructive-foreground',
    muted: 'bg-muted text-muted-foreground',
  },
  // Modern container queries
  container: 'container @container',
  // CSS Grid improvements
  grid: 'grid grid-cols-[repeat(auto-fit,minmax(250px,1fr))]',
};
```

### Tailwind 4 Performance Optimizations
```typescript
// ✅ Use Tailwind 4's improved arbitrary value syntax
const modernClasses = cn(
  // Improved arbitrary values
  'w-[clamp(200px,50vw,400px)]',
  // Container queries
  '@md:grid-cols-2 @lg:grid-cols-3',
  // Modern CSS functions
  'h-[max(200px,20vh)]',
  'gap-[max(1rem,2vw)]'
);
```

## Component Architecture Pattern

### Standard Component Structure
```typescript
// ✅ Consistent pattern used in lambda-curry/forms
import * as React from 'react';
import { Slot } from '@radix-ui/react-slot';
import { type VariantProps, cva } from 'class-variance-authority';
import { cn } from '@/lib/utils';

// Variant pattern following your existing components
const buttonVariants = cva(
  // Base classes optimized for Tailwind 4
  'inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 [&_svg]:pointer-events-none [&_svg]:size-4 [&_svg]:shrink-0',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground shadow hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground shadow-sm hover:bg-destructive/90',
        outline: 'border border-input bg-background shadow-sm hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground shadow-sm hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-9 px-4 py-2',
        sm: 'h-8 rounded-md px-3 text-xs',
        lg: 'h-10 rounded-md px-8',
        icon: 'h-9 w-9',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

// React 19 pattern (no forwardRef needed)
export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

export const Button = ({ className, variant, size, asChild = false, ...props }: ButtonProps) => {
  const Comp = asChild ? Slot : 'button';
  return (
    <Comp
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  );
};
Button.displayName = 'Button';
```

## Radix UI Integration Patterns

### Dropdown Menu Pattern (Based on Your Components)
```typescript
import * as React from 'react';
import * as DropdownMenuPrimitive from '@radix-ui/react-dropdown-menu';
import { Check, ChevronRight, Circle } from 'lucide-react';
import { cn } from '@/lib/utils';

// Following your existing dropdown-menu.tsx pattern
const DropdownMenu = DropdownMenuPrimitive.Root;
const DropdownMenuTrigger = DropdownMenuPrimitive.Trigger;
const DropdownMenuGroup = DropdownMenuPrimitive.Group;
const DropdownMenuPortal = DropdownMenuPrimitive.Portal;
const DropdownMenuSub = DropdownMenuPrimitive.Sub;
const DropdownMenuRadioGroup = DropdownMenuPrimitive.RadioGroup;

// Content with Tailwind 4 optimizations
const DropdownMenuContent = React.forwardRef<
  React.ElementRef<typeof DropdownMenuPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof DropdownMenuPrimitive.Content>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lambda-curry) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
