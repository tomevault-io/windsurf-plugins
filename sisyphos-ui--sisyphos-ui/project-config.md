---
trigger: always_on
description: **Sisyphos UI is a PURE Design System:**
---

# Sisyphos UI Design System - Cursor Rules

## 🎯 Design Philosophy

**Sisyphos UI is a PURE Design System:**
- **Components + Tokens ONLY** - No utilities, no state management, no business logic
- **Pure Design System Focus** - Focus on UI components and design tokens
- **Framework Agnostic** - Works with any state management (Redux, Zustand, Context, etc.)
- **No React Integration** - No ThemeProvider, users manage their own state

---

## 📦 Package Structure

```
packages/
├── core/              # Theme system + Design tokens
├── button/            # Component packages
├── input/             # Component packages (future)
└── ui/                # Umbrella package (re-exports all)
```

**Rule:** Every component must be in its own package under `packages/`.

---

## 🧩 Component Development Rules

### 1. Component Structure (MANDATORY)

Every component MUST follow this structure:

```
packages/component-name/
├── src/
│   ├── ComponentName.tsx      # React component
│   ├── ComponentName.scss     # Styles (uses tokens)
│   ├── ComponentName.stories.tsx  # Storybook (optional)
│   └── index.ts                # Exports
├── package.json
├── tsconfig.json
└── tsup.config.ts
```

### 2. TypeScript Rules

#### Component File Structure (MANDATORY):

```tsx
/**
 * ComponentName
 * @description Brief description
 */
import React, { useMemo } from "react";
import "./ComponentName.scss";

// ==================== TYPES ====================
export interface ComponentNameProps {
  /** Description with JSDoc */
  variant?: "primary" | "secondary";
  color?: "primary" | "success" | "error" | "warning" | "info";
  size?: "xs" | "sm" | "md" | "lg" | "xl";
  className?: string;
  children?: React.ReactNode;
  disabled?: boolean;
}

// ==================== COMPONENT ====================
export const ComponentName: React.FC<ComponentNameProps> = ({
  variant = "primary",
  color = "primary",
  size = "md",
  className = "",
  children,
  disabled = false,
}) => {
  // Use useMemo for class name building
  const componentClasses = useMemo(() => {
    return [
      "sisyphos-componentname",
      `sisyphos-componentname--${variant}`,
      `sisyphos-componentname--${color}`,
      `sisyphos-componentname--${size}`,
      disabled && "sisyphos-componentname--disabled",
      className,
    ]
      .filter(Boolean)
      .join(" ");
  }, [variant, color, size, disabled, className]);

  return <div className={componentClasses}>{children}</div>;
};

ComponentName.displayName = "ComponentName";
```

#### TypeScript Requirements:
- ✅ Always use `React.FC` or `React.forwardRef`
- ✅ Always set `displayName`
- ✅ Always document props with JSDoc comments
- ✅ Use `useMemo` for class name building
- ✅ Extend native HTML props when applicable: `React.InputHTMLAttributes<HTMLInputElement>`
- ✅ Use semantic prop names: `color="primary"` not `color="blue"`

### 3. SCSS Rules (CRITICAL - NO HARD-CODED VALUES)

#### MANDATORY: Use Design Tokens

```scss
/* ✅ CORRECT - Use tokens */
@use "@sisyphos-ui/core/tokens/variables" as *;
@use "@sisyphos-ui/core/tokens/mixins" as *;

.sisyphos-componentname {
  /* Spacing - Use SCSS variables */
  padding: $spacing-md;
  margin: $spacing-s;
  
  /* Typography - Use SCSS variables */
  font-size: $base-size;
  font-weight: $font-weight-medium;
  line-height: $line-height;
  
  /* Colors - Use CSS variables (theme-aware) */
  color: var(--sisyphos-color-neutral-dark, #212b36);
  background: var(--sisyphos-color-neutral-lighter, #fff);
  border-color: var(--sisyphos-color-neutral-darker, #919eab33);
  
  /* Border Radius - Use SCSS variables */
  border-radius: $radius-md;
  
  /* Transitions - Use SCSS variables */
  transition: all $duration-s;
  
  /* Z-Index - Use SCSS variables */
  z-index: $z-index-overlay;
  
  /* Opacity - Use SCSS variables */
  opacity: var(--sisyphos-opacity-xs, 0.6);
}

/* ❌ WRONG - Hard-coded values */
.sisyphos-componentname {
  padding: 16px;              /* ❌ Use $spacing-md */
  color: #212b36;             /* ❌ Use var(--sisyphos-color-neutral-dark) */
  border-radius: 12px;        /* ❌ Use $radius-md */
  font-size: 16px;            /* ❌ Use $base-size */
  z-index: 1000;              /* ❌ Use $z-index-overlay */
}
```

#### Color Usage Rules:

```scss
/* ✅ CORRECT - Semantic colors with CSS variables */
color: var(--sisyphos-color-primary, #ff7022);
background: var(--sisyphos-color-success, #22c55e);
border-color: var(--sisyphos-color-error-dark, #FB3748);

/* For neutral colors (backgrounds, borders, text) */
color: var(--sisyphos-color-neutral-dark, #212b36);
background: var(--sisyphos-color-neutral-lighter, #fff);
border-color: var(--sisyphos-color-neutral-darker, #919eab33);

/* ✅ CORRECT - With fallback values */
color: var(--sisyphos-color-primary, #ff7022); /* Default value as fallback */

/* ❌ WRONG - Hard-coded colors */
color: #007bff;              /* ❌ Use CSS variable */
background: #ffffff;         /* ❌ Use var(--sisyphos-color-neutral-lighter) */
border-color: #dadfe3;       /* ❌ Use var(--sisyphos-color-neutral-darker) */
```

#### Spacing Rules:

```scss
/* ✅ CORRECT - Use SCSS spacing tokens */
padding: $spacing-xs;        /* 8px */
padding: $spacing-s;         /* 10px */
padding: $spacing-md;        /* 16px */
padding: $spacing-lg;        /* 24px */


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sisyphos-ui/sisyphos-ui](https://github.com/sisyphos-ui/sisyphos-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
