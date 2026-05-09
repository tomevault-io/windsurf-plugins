---
trigger: always_on
description: description: CSS Modules, Tailwind CSS v4, and responsive design patterns
---

---
description: CSS Modules, Tailwind CSS v4, and responsive design patterns
globs: *.tsx, *.jsx, *.css, *.js, *.ts
---

# Styling Guidelines

## CSS Modules

### File Naming
Use kebab-case for CSS module files. Match the component name followed by `.module.css`

```
button.tsx
button.module.css
```

### Class Naming
Use camelCase for class names. Use descriptive, semantic names.

```css
.button { /* Base styles */ }
.isPrimary { /* Variant styles */ }
.isDisabled { /* State styles */ }
```

### Imports in Components
Always import CSS modules as `s`

```tsx
import s from './component-name.module.css'

function Component() {
  return <div className={s.wrapper} />
}
```

## Responsive Design

### Viewport Functions
Use custom viewport functions for responsive sizing

```css
.element {
  width: mobile-vw(150);      /* 150px at mobile viewport */
  height: mobile-vh(100);     /* 100px at mobile viewport */
  margin: desktop-vw(50);     /* 50px at desktop viewport */
  padding: desktop-vh(25);    /* 25px at desktop viewport */
}
```

### Breakpoints
Desktop breakpoint: 800px (defined in styles/config.ts)

```css
@media (min-width: 800px) {
  /* Desktop styles */
}
```

### Grid System
Use the column function for grid-based layouts

```css
.container {
  width: columns(6);          /* Span 6 columns */
  margin-left: columns(1);    /* Offset by 1 column */
}
```

## Typography

### Font Hierarchy
Use typography variables from the theme

```css
.title {
  font-size: var(--font-size-title);
  line-height: var(--line-height-title);
  font-weight: var(--font-weight-bold);
}
```

### Text Scaling
Use scale utilities with the 's' prefix

```css
.scalingText {
  --size: 1;
  font-size: s(var(--size) * 16px);   /* Scales appropriately */
}
```

## Colors and Themes

### Color Variables
Use theme colors from CSS variables

```css
.element {
  color: var(--color-text);
  background-color: var(--color-background);
  border-color: var(--color-accent);
}
```

### Theme Switching
Use theme-specific variables when needed

```css
.element {
  color: var(--theme-dark-text);
  background-color: var(--theme-dark-background);
}

[data-theme="light"] .element {
  color: var(--theme-light-text);
  background-color: var(--theme-light-background);
}
```

## Animations and Transitions

### Transition Timing
Use consistent transition variables

```css
.element {
  transition: opacity var(--transition-duration) var(--transition-ease);
}
```

### Animation Easings
Import easings from the theme

```css
.element {
  transition: transform 0.5s var(--ease-out-expo);
}
```

## Best Practices

### Performance
Prefer CPU-friendly properties (transform, opacity). Use `will-change` sparingly and only when needed.

```css
.animatedElement {
  will-change: transform, opacity;
}
```

### Organization
Group related properties together. Order properties consistently.

```css
.element {
  /* Positioning */
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
  
  /* Box model */
  display: flex;
  width: 100%;
  padding: 1rem;
  
  /* Visual */
  background-color: var(--color-background);
  border-radius: 4px;
  
  /* Typography */
  font-size: 1rem;
  color: var(--color-text);
  
  /* Animation */
  transition: all 0.3s ease;
}
```

---

# Tailwind CSS v4

## Core Changes

### CSS-first configuration
Configuration is now done in CSS instead of JavaScript. Use `@theme` directive in CSS instead of `tailwind.config.js`

```css
@import "tailwindcss";

@theme {
  --font-display: "Satoshi", "sans-serif";
  --breakpoint-3xl: 1920px;
  --color-avocado-500: oklch(0.84 0.18 117.33);
  --ease-fluid: cubic-bezier(0.3, 0, 0, 1);
}
```

### CSS import syntax
Use `@import "tailwindcss"` instead of `@tailwind` directives

- Old: `@tailwind base; @tailwind components; @tailwind utilities;`
- New: `@import "tailwindcss";`

### Package changes
- PostCSS plugin is now `@tailwindcss/postcss` (not `tailwindcss`)
- CLI is now `@tailwindcss/cli`
- Vite plugin is `@tailwindcss/vite`
- No need for `postcss-import` or `autoprefixer` anymore

### Native CSS cascade layers
Uses real CSS `@layer` instead of Tailwind's custom implementation

## Theme Configuration

### CSS theme variables
All design tokens are available as CSS variables

- Namespace format: `--category-name` (e.g., `--color-blue-500`, `--font-sans`)
- Access in CSS: `var(--color-blue-500)`
- Available namespaces:
  - `--color-*` : Color utilities like `bg-red-500` and `text-sky-300`
  - `--font-*` : Font family utilities like `font-sans`
  - `--text-*` : Font size utilities like `text-xl`
  - `--font-weight-*` : Font weight utilities like `font-bold`
  - `--tracking-*` : Letter spacing utilities like `tracking-wide`
  - `--leading-*` : Line height utilities like `leading-tight`
  - `--breakpoint-*` : Responsive breakpoint variants like `sm:*`
  - `--container-*` : Container query variants like `@sm:*` and size utilities like `max-w-md`
  - `--spacing-*` : Spacing and sizing utilities like `px-4` and `max-h-16`
  - `--radius-*` : Border radius utilities like `rounded-sm`
  - `--shadow-*` : Box shadow utilities like `shadow-md`
  - `--inset-shadow-*` : Inset box shadow utilities like `inset-shadow-xs`
  - `--drop-shadow-*` : Drop shadow filter utilities like `drop-shadow-md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tambo-ai/tambo-landing](https://github.com/tambo-ai/tambo-landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
