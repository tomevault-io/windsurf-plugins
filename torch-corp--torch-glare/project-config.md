---
trigger: always_on
description: TORCH Glare is a React component library built with TypeScript, Radix UI, and Tailwind CSS. This guide explains patterns and conventions for building new components.
---

# CLAUDE.md - TORCH Glare Component Development Guide

## Project Overview

TORCH Glare is a React component library built with TypeScript, Radix UI, and Tailwind CSS. This guide explains patterns and conventions for building new components.

## Project Structure

```
TORCH-Glare/
├── apps/lib/                  # Main component library
│   ├── components/            # UI components
│   ├── hooks/                 # Custom React hooks
│   ├── layouts/               # Layout components
│   ├── providers/             # Context providers
│   └── utils/                 # Utilities (cn, types)
├── cli/                       # torch-glare CLI tool
├── plugins/                   # Tailwind plugins
└── docs/                      # Documentation
```

## Development Commands

```bash
cd apps
pnpm install
pnpm run dev          # Start dev server
pnpm run build        # Production build
pnpm run lint         # Run ESLint
```

## Component Building Patterns

### 1. File Structure

Components live in `apps/lib/components/`. Each component is a single file named in PascalCase.

### 2. Required Imports

```typescript
import React, { forwardRef } from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "../utils/cn";
import { Themes } from "../utils/types";
// For Radix-based components:
import * as PrimitiveName from "@radix-ui/react-primitive-name";
```

### 3. CVA Variant Definition

Define styles using `cva()` with variants object:

```typescript
const componentStyles = cva(
  [
    // Base styles (always applied)
    "flex items-center",
    "transition-all duration-200 ease-in-out",
    "border",
  ],
  {
    variants: {
      variant: {
        PresentationStyle: [
          "bg-background-presentation-form-field-primary",
          "border-border-presentation-action-primary",
          "hover:bg-background-presentation-form-field-hover",
          "focus-within:border-border-presentation-state-focus",
        ],
        SystemStyle: [
          "bg-black-alpha-20",
          "text-white",
          "border-[#2C2D2E]",
        ],
      },
      size: {
        S: ["h-[22px]", "text-[12px]"],
        M: ["h-[28px]", "text-[14px]"],
        L: ["h-[34px]", "text-[16px]"],
        XL: ["h-[40px]", "text-[18px]"],
      },
      error: {
        true: [
          "border-border-presentation-state-negative",
        ],
      },
      disabled: {
        true: [
          "cursor-not-allowed",
          "bg-background-presentation-action-disabled",
        ],
      },
    },
    defaultVariants: {
      variant: "PresentationStyle",
      size: "M",
    },
    compoundVariants: [
      {
        disabled: true,
        variant: "PresentationStyle",
        className: "pointer-events-none",
      },
    ],
  }
);
```

### 4. Interface Definition

Extend HTML attributes and CVA variants:

```typescript
interface Props
  extends HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof componentStyles> {
  theme?: Themes;
  // Custom props
}
```

For Radix-based components:

```typescript
interface Props
  extends React.ComponentPropsWithoutRef<typeof Primitive.Root>,
    VariantProps<typeof componentStyles> {
  theme?: Themes;
}
```

### 5. Component with forwardRef

```typescript
export const Component = forwardRef<HTMLDivElement, Props>(
  ({ className, variant, size, theme, ...props }, ref) => {
    return (
      <div
        ref={ref}
        data-theme={theme}
        className={cn(componentStyles({ variant, size }), className)}
        {...props}
      />
    );
  }
);

Component.displayName = "Component";
```

### 6. Radix UI Wrapper Pattern

```typescript
"use client"

import * as DialogPrimitive from "@radix-ui/react-dialog";

const Dialog = DialogPrimitive.Root;
const DialogTrigger = DialogPrimitive.Trigger;

const DialogContent = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof DialogPrimitive.Content>
>(({ className, children, ...props }, ref) => (
  <DialogPrimitive.Portal>
    <DialogPrimitive.Overlay className={cn("fixed inset-0 z-50 bg-black/80")} />
    <DialogPrimitive.Content
      ref={ref}
      className={cn("fixed left-[50%] top-[50%] z-50", className)}
      {...props}
    >
      {children}
    </DialogPrimitive.Content>
  </DialogPrimitive.Portal>
));
DialogContent.displayName = DialogPrimitive.Content.displayName;

export { Dialog, DialogTrigger, DialogContent };
```

### 7. Compound Component Pattern (Input)

```typescript
// Group wrapper
export const Group = forwardRef<HTMLDivElement, GroupProps>(
  ({ size, variant, error, className, ...props }, ref) => (
    <div
      ref={ref}
      className={cn(GroupStyles({ size, variant, error }), className)}
      {...props}
    />
  )
);

// Icon slot
export const Icon = ({ children, className }: IconProps) => (
  <div className={cn("flex items-center", className)} data-role="icon">
    {children}
  </div>
);

// Input element
export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ className, ...props }, ref) => (
    <input
      ref={ref}
      className={cn("flex-1 bg-transparent outline-none", className)}
      {...props}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TORCH-Corp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
