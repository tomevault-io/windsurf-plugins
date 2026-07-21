---
trigger: always_on
description: This guide provides instructions and context for AI agents working with the Backpack Design System codebase.
---

# AI Agents Guide for Backpack Design System

This guide provides instructions and context for AI agents working with the Backpack Design System codebase.

## Project Overview

**Backpack** is Skyscanner's design system - a collection of design resources, reusable components, and guidelines for creating consistent user interfaces across Skyscanner's products.

- **Repository**: Skyscanner/backpack
- **Language**: TypeScript/JavaScript (React components)
- **Package Manager**: pnpm
- **Build System**: Webpack, Gulp + custom scripts
- **Styling**: SCSS with BEM methodology
- **Documentation**: [skyscanner.design](https://www.skyscanner.design/)

## Key Architecture Patterns

### Component Structure
- Each component lives in `packages/backpack-web/src/bpk-component-{name}/`
- Components follow the pattern: `BpkComponentName`
- All components are prefixed with `Bpk`
- Examples: `BpkButton`, `BpkCard`, `BpkChip`

### Package Organization
```
packages/
├── bpk-component-{name}/          # Individual React components
├── bpk-mixins/                    # SCSS mixins and utilities
├── bpk-stylesheets/               # Compiled CSS
└── bpk-tokens/                    # Design tokens
```

### File Naming Conventions
- React components: PascalCase (e.g., `BpkButton.tsx`)
- SCSS files: PascalCase (e.g., `BpkButton.module.scss`)
- Test files: `{ComponentName}.test.tsx`
- Story files: `{ComponentName}.stories.tsx`

## Development Guidelines

### Code Standards
- **TypeScript**: All new components must be written in TypeScript
- **Props Interface**: Define clear TypeScript interfaces for all component props
- **Default Props**: Use default parameters for optional props
- **Accessibility**: All components must meet WCAG 2.2 AA standards
- **Testing**: Jest + React Testing Library for unit tests
- **Storybook**: All components must have corresponding stories

### SCSS Guidelines
- Use BEM methodology for CSS class naming
- All classes prefixed with `bpk-`
- Example: `.bpk-button`, `.bpk-button--large`, `.bpk-button__icon`
- Import design tokens from `bpk-tokens`
- Use SCSS mixins from `bpk-mixins`

### Component API Patterns
```typescript
// Standard prop patterns
interface BpkComponentProps {
  children?: React.ReactNode;
  className?: string;
  onClick?: (event: React.MouseEvent) => void;
  // ... component-specific props
}

// Common prop naming
- size: 'small' | 'default' | 'large'
- variant: 'primary' | 'secondary' | 'destructive'
- disabled: boolean
- loading: boolean
```

## Build and Development Commands

```bash
# Install dependencies
pnpm install

# Build all packages
pnpm run build

# Run tests
pnpm test

# Run Storybook
pnpm run storybook

# Lint code
pnpm run lint

# Type check
pnpm run typecheck
```

## Component Development Workflow

1. **Create Component Package**
   - Follow existing package structure in `packages/backpack-web/src/bpk-component-{name}/`
   - Include: component file, tests, stories, SCSS, and package.json

2. **Component Implementation**
   - Write TypeScript React component with proper props interface
   - Include proper JSDoc comments
   - Implement accessibility features (ARIA labels, keyboard navigation)
   - Follow existing component patterns

3. **Styling**
   - Create SCSS file with BEM naming
   - Use design tokens from `bpk-tokens`
   - Follow responsive design principles
   - Support RTL languages

4. **Testing**
   - Unit tests with React Testing Library
   - Accessibility tests
   - Visual regression tests (if applicable)

5. **Documentation**
   - Storybook stories showing all variants
   - Comprehensive prop documentation
   - Usage examples

## Design Tokens and Typography

Backpack uses design tokens and typography mixins for consistent styling across all components. The token system is built on top of `@skyscanner/bpk-foundations-web` and provides access to all design system values.

### Design Token Architecture

Design tokens are centralized in `packages/bpk-mixins/_tokens.scss` which forwards all tokens from the foundations package:

```scss
@forward '@skyscanner/bpk-foundations-web/tokens/base.default';
```

### Importing Tokens and Typography

Always import both tokens and typography at the top of your SCSS files:

```scss
@use '../../bpk-mixins/tokens';
@use '../../bpk-mixins/typography';
```

### Typography Mixins

Backpack provides a comprehensive set of typography mixins for consistent text styling. Use these instead of setting font properties manually. There are more options available in `bpk-mixins/typography`.

#### Text Size Mixins
```scss
.my-component {
  // Size-based typography
  &__small-text {
    @include typography.bpk-text-xs;    // Extra small
    @include typography.bpk-text-sm;    // Small
    @include typography.bpk-text-base;  // Base/default
    @include typography.bpk-text-lg;    // Large
    @include typography.bpk-text-xl;    // Extra large
    @include typography.bpk-text-xxl;   // 2x large
    @include typography.bpk-text-xxxl;  // 3x large
  }
}
```

### Design Token Usage

#### Color Tokens
```scss
.my-component {
  // Text colors
  color: tokens.$bpk-text-primary-day;
  color: tokens.$bpk-text-secondary-day;
  color: tokens.$bpk-text-disabled-day;
  color: tokens.$bpk-text-on-dark-day;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Skyscanner/backpack](https://github.com/Skyscanner/backpack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
