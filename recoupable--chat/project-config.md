---
trigger: always_on
description: Modern UI Component Building Standards - Comprehensive guidelines for building accessible, composable, and maintainable UI components
---


# Modern UI Component Building Standards

## 🚨 CRITICAL REQUIREMENTS (Must Follow)

### Accessibility (NON-NEGOTIABLE)

- **ALWAYS** use semantic HTML elements appropriate to the component's role
- **ALWAYS** ensure keyboard navigation and focus management
- **ALWAYS** provide proper ARIA roles/states and test with screen readers
- **NEVER** create components that aren't accessible - this is a baseline requirement
- **ALWAYS** start with semantic HTML first, then augment with ARIA if needed

### TypeScript Standards (MANDATORY)

- **ALWAYS** extend native HTML attributes: `React.ComponentProps<"div">`
- **ALWAYS** export component prop types: `export type ComponentNameProps`
- **ALWAYS** support both controlled and uncontrolled state
- **ALWAYS** wrap a single HTML element per component
- **NEVER** create components that wrap multiple elements

### Composition Patterns (ESSENTIAL)

- **ALWAYS** favor composition over inheritance
- **ALWAYS** expose clear APIs via props/slots for customization
- **ALWAYS** make components composable and reusable
- **ALWAYS** use `asChild` pattern for flexible element types
- **ALWAYS** support polymorphism with `as` prop when appropriate

## 🎯 HIGH PRIORITY PATTERNS

### Framework Standards (shadcn/ui + Radix UI)

- **MUST** use `@radix-ui/react-*` primitives for behavior and accessibility
- **MUST** use `class-variance-authority` (CVA) for component variants
- **MUST** use `@radix-ui/react-slot` for `asChild` pattern
- **MUST** use `@radix-ui/react-use-controllable-state` for state management
- **MUST** follow shadcn/ui component structure and naming conventions
- **MUST** use `cn()` utility combining `clsx` and `tailwind-merge`
- **MUST** export both component and variant functions (e.g., `Button, buttonVariants`)

### Project-Specific Dependencies

- **MUST** use `@radix-ui/react-icons` for icons (not Lucide React for UI components)
- **MUST** use `lucide-react` for application icons and illustrations
- **MUST** use `framer-motion` for animations and transitions
- **MUST** use `next-themes` for theme management
- **MUST** use `tailwindcss-animate` for CSS animations
- **MUST** use `sonner` for toast notifications
- **MUST** use `embla-carousel-react` for carousels
- **MUST** use `react-resizable-panels` for resizable layouts

### State Management

- **MUST** support both controlled and uncontrolled modes
- **MUST** use `useControllableState` pattern for state merging
- **MUST** provide `defaultValue` and `onValueChange` props
- **MUST** allow parent to control state or component to manage internally

### Styling with Tailwind

- **MUST** use `cn()` utility combining `clsx` and `tailwind-merge`
- **MUST** follow class order: base → variants → conditionals → user overrides
- **MUST** use `data-state` for visual states: `data-state="open|closed"`
- **MUST** use `data-slot` for component identification: `data-slot="button"`
- **MUST** define variants outside components to avoid recreation

### Data Attributes (REQUIRED)

- **ALWAYS** use `data-state` for component states (open/closed, loading, etc.)
- **ALWAYS** use `data-slot` for component identification
- **ALWAYS** use kebab-case naming: `data-slot="form-field"`
- **ALWAYS** enable parent-aware styling with `has-[]` selectors

## 📋 STANDARD PRACTICES

### Component Architecture

- **Primitive**: Unstyled, behavior-focused building blocks
- **Component**: Styled, immediately usable UI units
- **Pattern**: Documented solutions for common problems
- **Block**: Production-ready compositions

### API Design

- **Props API**: Stable, typed, documented with defaults
- **Children/Slots**: Support implicit and named slots
- **Render Props**: Use function-as-child when parent owns data
- **Provider/Context**: Top-level components for shared state
- **Portal**: For layering/stacking context with a11y

### Styling and Theming

- **Design Tokens**: Named, platform-agnostic values
- **Headless vs Styled**: Choose based on use case
- **Variants**: Discrete style/behavior permutations via props
- **Class Variance Authority**: For complex variant management

## 🔧 IMPLEMENTATION GUIDELINES

### TypeScript Best Practices (shadcn/ui Pattern)

```tsx
// ✅ CORRECT: shadcn/ui component structure
import { cva, type VariantProps } from "class-variance-authority";
import { Slot } from "@radix-ui/react-slot";
import { cn } from "@/lib/utils";

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive:
          "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline:
          "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary:
          "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [recoupable/chat](https://github.com/recoupable/chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
