---
trigger: always_on
description: Use this rule when you are building any form UI.
---


# Tailwind CSS Styling Standards for UI Builder

## Overview

This document outlines the Tailwind CSS styling standards and patterns used throughout the OpenZeppelin UI Builder codebase. These rules ensure consistency, maintainability, and adherence to the established design system.

## Design System Foundation

### Semantic Color Tokens

**ALWAYS use semantic design tokens instead of hard-coded colors:**

**PREFERRED:**

```tsx
// Semantic tokens that adapt to themes
className = 'bg-primary text-primary-foreground';
className = 'text-muted-foreground';
className = 'border-input bg-background';
className = 'text-destructive bg-destructive/10';
```

**AVOID:**

```tsx
// Hard-coded colors
className = 'bg-blue-500 text-white';
className = 'text-gray-600';
className = 'border-gray-300';
```

**Core Semantic Tokens:**

- **Backgrounds**: `bg-background`, `bg-card`, `bg-muted`, `bg-primary`, `bg-secondary`, `bg-destructive`
- **Text**: `text-foreground`, `text-muted-foreground`, `text-primary-foreground`, `text-destructive`
- **Borders**: `border-input`, `border-destructive`
- **Interactive**: `hover:bg-accent`, `hover:text-accent-foreground`

### CSS Variable Integration

Leverage CSS variables for consistent theming:

```tsx
className = 'ring-offset-background focus-visible:ring-ring';
```

## Layout Patterns

### Responsive Grid Systems

Use responsive grid layouts with proper breakpoints:

```tsx
// Form field grids
className = 'grid grid-cols-1 gap-4 md:grid-cols-2';

// Option selector layouts
className = 'grid-cols-3 gap-4'; // Desktop
className = 'grid-cols-[auto_1fr]'; // Collapsed mode
```

### Flexbox Patterns

Standard flex patterns for common layouts:

```tsx
// Standard flex container
className = 'flex items-center gap-2';

// Full height layouts
className = 'flex min-h-screen flex-col';

// Space between items
className = 'flex justify-between items-center';

// Centered content
className = 'flex items-center justify-center';
```

### Spacing System

Consistent spacing using Tailwind's scale:

```tsx
// Container spacing
className = 'space-y-4'; // Vertical spacing between children
className = 'space-y-6'; // Larger sections
className = 'space-y-2'; // Tight spacing

// Padding patterns
className = 'p-6'; // Standard container padding
className = 'px-3 py-2.5'; // Button/input padding
className = 'px-4 py-3'; // Card content padding
```

## Component-Specific Patterns

### Button Styling

Follow the established button variant system:

```tsx
// Use predefined variants from class-variance-authority
className={cn(buttonVariants({ variant, size, className }))}

// Common button patterns
className="flex items-center gap-2 pl-6 pr-3 py-2.5 h-11" // Action buttons
className="h-full w-5 p-0 transition-opacity" // Icon buttons
```

### Input Field Styling

Standard input field patterns:

```tsx
// Base input styling
className="border-input bg-background ring-offset-background focus-visible:ring-ring placeholder:text-muted-foreground h-10 w-full rounded-md border px-3 py-2 text-sm"

// Field container
className="flex flex-col gap-2 w-full"

// Width variants
className={`${width === 'full' ? 'w-full' : width === 'half' ? 'w-1/2' : 'w-1/3'}`}
```

### Card and Container Styling

Consistent card and container patterns:

```tsx
// Card backgrounds
className = 'bg-card';

// Section containers
className = 'rounded-md border overflow-hidden';

// Content padding
className = 'px-4 py-3 border-b last:border-0';
```

## State Management with Classes

### Interactive States

Implement consistent hover and focus states:

```tsx
// Hover effects
className = 'hover:bg-muted/50 cursor-pointer transition-colors';
className =
  "hover:before:content-[''] hover:before:absolute hover:before:inset-x-0 hover:before:top-1 hover:before:bottom-1 hover:before:bg-muted/80 hover:before:rounded-lg hover:before:-z-10";

// Focus states
className = 'focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:outline-none';
```

### Conditional State Classes

Use conditional classes for component states:

```tsx
// Selected/active states
className={cn(
  'base-classes',
  isSelected ? 'bg-primary text-primary-foreground' : 'text-muted-foreground',
  disabled ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'
)}
```

### Validation States

Consistent validation and error styling:

```tsx
// Error states
className = 'border-destructive bg-destructive/10';
className = 'text-destructive text-sm font-medium';

// Helper text
className = 'text-muted-foreground text-sm';
```

## Animation and Transitions

### Standard Transitions

Use consistent transition patterns:

```tsx
// Standard transition
className = 'transition-colors duration-300 ease-in-out';

// For layout changes
className = 'transition-all duration-300 ease-in-out';

// Opacity transitions
className = 'transition-opacity';
```

### Loading States

Standard loading and animation patterns:

```tsx
// Pulse animation
className = 'animate-pulse [animation-duration:1200ms]';

// Loading opacity
className = 'opacity-30';
```

## Responsive Design Rules

### Mobile-First Approach

Always use mobile-first responsive design:

```tsx
// Mobile hidden, desktop visible
className = 'hidden sm:block';

// Responsive grid changes
className = 'grid-cols-1 md:grid-cols-2';


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenZeppelin/ui-builder](https://github.com/OpenZeppelin/ui-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
