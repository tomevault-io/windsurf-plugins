---
trigger: always_on
description: enableSystem
---

# Cursor Rules for UI Library Project

# Project Structure

## Atomic Design Structure

- All components must be organized following atomic design principles in `src/components`:
  - `atoms/` - Basic building blocks
    - `button/`
    - `input/`
    - `icon/`
    - `typography/`
  - `molecules/` - Simple combinations of atoms
    - `form-field/`
    - `search-bar/`
    - `card-header/`
  - `organisms/` - Complex combinations of molecules
    - `navigation/`
    - `form/`
    - `card/`
    - `data-table/`
    - `modal/`
    - `sidebar/`

Note: Templates and Pages are not part of the UI library as they are application-specific components that should be implemented in the consuming Next.js application.

## Component Structure

Each component should have its own directory with the following structure:

- `component-name/`
  - `index.ts` (exports)
  - `component-name.tsx` (main component)
  - `component-name.test.tsx` (tests)
  - `component-name.stories.tsx` (Storybook)
  - `component-name.types.ts` (TypeScript types)
  - `component-name.module.css` (if needed)

## Utility Structure

All utility files should use kebab-case naming:

- `hooks/` - Custom React hooks
  - `use-hook-name.ts`
- `utils/` - Utility functions
  - `utils-name.ts`
- `constants/` - Constant values
  - `constants-name.ts`
- `types/` - Type definitions
  - `types-name.ts`
- `styles/` - Global styles
  - `styles-name.css`
- `config/` - Configuration files
  - `config-name.ts`

# Component Rules

## Atomic Design Principles

### Atoms

- Smallest possible components
- Cannot be broken down further
- Examples: buttons, inputs, icons, typography
- Should be highly reusable
- Should have minimal props
- Should be stateless when possible
- Should be the foundation of the design system

### Molecules

- Combinations of atoms
- Form simple, functional units
- Examples: form fields, search bars, card headers
- Should handle simple state
- Should be focused on a single responsibility
- Should be composed of atoms
- Should be reusable across different organisms

### Organisms

- Complex combinations of molecules
- Form distinct sections of an interface
- Examples: navigation bars, forms, cards, data tables
- Can handle complex state
- Can manage their own data fetching
- Should be composed of molecules and atoms
- Should be the highest level of abstraction in the library
- Should be flexible enough to be used in various application contexts
- Should not contain application-specific logic

Note: Templates and Pages are not part of the UI library as they are application-specific components that should be implemented in the consuming Next.js application.

- Use functional components with TypeScript
- Implement proper prop types using TypeScript interfaces
- Use React.memo() for performance optimization when needed
- Follow atomic design principles
- Implement proper error boundaries
- Use proper accessibility attributes (aria-\*)
- Implement proper keyboard navigation
- Use proper semantic HTML elements
- Use kebab-case for component file names and directories
- Use PascalCase for component names in code

# Accessibility Rules

## Core Accessibility Requirements

- All components MUST be fully accessible and follow WCAG 2.1 Level AA guidelines
- Every component must be keyboard navigable and operable
- All interactive elements must have proper focus management
- All form controls must have associated labels
- All images must have meaningful alt text
- All color combinations must meet WCAG contrast requirements
- All components must be screen reader friendly

## ARIA Implementation

- Use semantic HTML elements over ARIA when possible
- When ARIA is needed:
  - Use correct ARIA roles and attributes
  - Ensure all required ARIA attributes are present
  - Validate ARIA attribute values
  - Test with screen readers
- Common ARIA patterns:
  - `aria-label` for elements without visible text
  - `aria-labelledby` for elements with associated labels
  - `aria-describedby` for additional descriptions
  - `aria-expanded` for expandable elements
  - `aria-controls` for elements that control others
  - `aria-live` for dynamic content updates

## Keyboard Navigation

- All interactive elements must be focusable
- Implement proper tab order
- Support keyboard shortcuts where appropriate
- Handle keyboard events (Enter, Space, Escape, etc.)
- Provide visible focus indicators
- Support arrow key navigation for complex components
- Implement skip links for main content

## Screen Reader Support

- Use proper heading hierarchy (h1-h6)
- Provide meaningful text alternatives
- Use proper landmark roles
- Implement proper live regions
- Test with multiple screen readers
- Ensure proper reading order
- Provide proper announcements for dynamic content

## Color and Contrast

- Maintain minimum contrast ratio of 4.5:1 for normal text
- Maintain minimum contrast ratio of 3:1 for large text
- Don't rely solely on color to convey information
- Support high contrast mode
- Test with color blindness simulators
- Provide alternative indicators for color-coded information

## Form Accessibility

- All form controls must have visible labels
- Provide clear error messages
- Support autocomplete where appropriate
- Implement proper validation feedback
- Use proper input types
- Support required field indicators

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sergii-melnykov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
