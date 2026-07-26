---
trigger: always_on
description: Provides styling through two distinct layers:
---

# AGENTS.md — `@faststore/components`

> Inherits from [`/AGENTS.md`](../../AGENTS.md) (root). This file adds context specific to the `@faststore/components` library.

## Purpose

`@faststore/components` is the **structure layer** of the FastStore design system. It builds component structure and functionality **without styling**. Styling lives in `@faststore/ui`.

When working in this package, follow these conventions in addition to the root principles.

## Two-Package Architecture

FastStore components are split across two complementary packages:

### `@faststore/components` (Structure Layer — this package)

**Responsibilities:**

- JSX structure and markup
- Component logic, hooks, and utilities
- Data attributes for styling hooks
- Semantic HTML for accessibility
- **No default styles applied**

**Key features:**

- All components accept `className` prop for custom styling
- Extensive use of data attributes for:
  - Component identification: `data-fs-{component-name}`
  - Internal structures: `data-fs-{component-name}-{internal-structure}`
  - Component states: `data-fs-{component-name}-{state}={value}`

### `@faststore/ui` (Styling Layer — sibling package)

Provides styling through two distinct layers:

1. **Structural Styles (Brandless):** minimum necessary styling for components to function.
2. **Themification:** additional customizable styles via theme templates (Soft Blue, Midnight, etc.).

**Global Tokens** ensure consistency across colors, typography, spacing, borders, and shadows. Always use global tokens to maintain compatibility with Themification.

## Atomic Design Pattern

`@faststore/components` follows Atomic Design hierarchically:

- **Atoms** (`src/atoms/`): basic building blocks — Button, Icon, Input, Label
- **Molecules** (`src/molecules/`): composite components — Accordion, CartItem, SearchInput
- **Organisms** (`src/organisms/`): complex feature components — ImageGallery, PriceRange, ProductShelf

**Key benefit:** when you update an atom like `Button`, all components that use it (like `IconButton`) automatically inherit those updates.

## Data Attributes Naming Convention

Follow this strict pattern:

| Context            | Pattern                                    | Example                            |
| ------------------ | ------------------------------------------ | ---------------------------------- |
| Root element       | `data-fs-{component-name}`                 | `data-fs-badge`                    |
| Internal structure | `data-fs-{component-name}-{structure}`     | `data-fs-badge-wrapper`            |
| Component state    | `data-fs-{component-name}-{state}={value}` | `data-fs-button-variant="primary"` |

**Example — `Button` data attributes:**

```typescript
// packages/components/src/atoms/Button/Button.tsx

export type Variant = 'primary' | 'secondary' | 'tertiary'
export type Size = 'small' | 'regular'
export type IconPosition = 'left' | 'right'

<button
  ref={ref}
  data-fs-button                          // Root identifier
  data-fs-button-inverse={inverse}        // State
  data-fs-button-size={size}              // Size variant
  data-fs-button-loading={loading}        // Loading state
  data-fs-button-variant={variant}        // Variant state
  disabled={disabled}
  data-testid={testId}
  {...otherProps}
>
  <div data-fs-button-wrapper>            // Internal structure
    {loading && (
      <p data-fs-button-loading-label>    // Internal structure
        {loadingLabel}
        <Loader variant={variant === 'primary' && !inverse ? 'light' : 'dark'} />
      </p>
    )}
    {!!icon && iconPosition === 'left' && (
      <span data-fs-button-icon>{icon}</span>
    )}
    {children && <span>{children}</span>}
    {!!icon && iconPosition === 'right' && (
      <span data-fs-button-icon>{icon}</span>
    )}
  </div>
</button>
```

## Implementing a New Component

### Step 1: Categorize the component

- **Atoms:** basic elements that can't be broken down further (Button, Input, Icon)
- **Molecules:** combinations of atoms serving a specific purpose (SearchInput, ProductCard)
- **Organisms:** complex sections combining molecules/atoms (CartSidebar, ImageGallery)

> Tip: if unsure, consult the FastStore team. Generally, if it uses other components, it's at least a molecule.

### Step 2: Create the structure (`@faststore/components`)

**Location:** `packages/components/src/{category}/{ComponentName}/`

Files to create:

- `ComponentName.tsx` — component implementation
- `index.ts` — exports

**Example `index.ts`:**

```typescript
// packages/components/src/atoms/Badge/index.ts
export { default } from "./Badge";
export type { BadgeProps } from "./Badge";
```

**Implementation guidelines:**

1. **Reuse existing components** whenever possible.
2. **Use semantic HTML** for accessibility and readability.

   ```tsx
   // Good
   <button>Click me</button>
   <nav>...</nav>

   // Avoid
   <div onClick={...}>Click me</div>
   ```

3. **Accessibility first:** provide ARIA attributes, full keyboard navigation, logical focus order, screen reader compatibility, sufficient color contrast.
4. **Define props interface** as `{ComponentName}Props`:

   ```typescript
   export interface BadgeProps {
     variant?: "success" | "warning" | "danger";
     size?: "small" | "big";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vtex/faststore](https://github.com/vtex/faststore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
