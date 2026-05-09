---
trigger: always_on
description: > This document is for agents and LLMs to follow when maintaining, generating,
---

# User Interface Wiki - Project Standards

**Version 1.0.0**  
January 2026

> **Note:**  
> This document is for agents and LLMs to follow when maintaining, generating,  
> or refactoring code in this repository. Humans may also find it useful, but  
> guidance here is optimized for automation and consistency by AI-assisted workflows.

---

## Abstract

Standards and conventions for the User Interface Wiki, a Next.js documentation site focused on UI/UX design principles. Contains guidelines for components, styling, content, and code organization to ensure consistency and maintainability.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Naming Conventions](#2-naming-conventions)
3. [File Structure](#3-file-structure)
4. [Component Standards](#4-component-standards)
5. [CSS Standards](#5-css-standards)
6. [Content Standards](#6-content-standards)
7. [TypeScript Standards](#7-typescript-standards)
8. [Animation Standards](#8-animation-standards)
9. [Accessibility Standards](#9-accessibility-standards)
10. [Performance Standards](#10-performance-standards)
11. [External Skills](#external-skills)

---

## 1. Project Overview

### Tech Stack

| Technology | Purpose |
|------------|---------|
| Next.js 16 | Framework with App Router |
| React 19 | UI library with React Compiler |
| TypeScript | Type safety |
| Biome | Linting and formatting |
| CSS Modules | Scoped styling |
| Motion (Framer Motion) | Animation library |
| Fumadocs | MDX documentation framework |
| Zustand | State management |
| Base UI | Headless component primitives |

### Key Directories

```
/app           → Next.js pages and routes
/components    → Reusable UI components
/content       → MDX articles and demos
/icons         → SVG icon components
/lib           → Utilities, types, and stores
/public        → Static assets
/skills        → AI skill definitions (SKILL.md files)
/styles        → Global CSS and theme
```

---

## 2. Naming Conventions

**All names use kebab-case throughout the project.**

| Item | Convention | Example |
|------|------------|---------|
| Directories | kebab-case | `components/button-group/` |
| Files | kebab-case | `use-audio.ts`, `styles.module.css` |
| CSS classes | kebab-case | `.button-primary`, `.nav-item` |
| CSS variables | kebab-case | `--font-weight-medium` |
| URL slugs | kebab-case | `/12-principles-of-animation` |

**Incorrect:**

```css
.buttonPrimary { }
.NavItem { }
.button_primary { }
```

**Correct:**

```css
.button-primary { }
.nav-item { }
```

**Exception:** React component function names use PascalCase (e.g., `function Button()`).

---

## 3. File Structure

### Component Structure

Each component lives in its own directory with colocated files:

```
components/
  button/
    index.tsx          # Component implementation
    styles.module.css  # Scoped styles
```

**Rules:**
- Export components from `index.tsx` using named exports
- Colocate CSS modules with components
- Use `index.ts` barrel files only for multi-file exports (e.g., hooks)

**Incorrect:**

```tsx
// button.tsx - wrong filename
export default function Button() { ... }
```

**Correct:**

```tsx
// index.tsx
export function Button() { ... }
```

### Content Structure

MDX content lives in `/content/` with demos colocated:

```
content/
  12-principles-of-animation/
    index.mdx           # Article content
    demos/
      index.ts          # Demo barrel export
      squash-stretch/
        index.tsx
        styles.module.css
```

**Rules:**
- Each article is a directory with `index.mdx`
- Demos are colocated in `demos/` subdirectory
- Export all demos from `demos/index.ts`

---

## 4. Component Standards

### Component Definition

Use function declarations with explicit prop interfaces:

**Incorrect:**

```tsx
const Button = (props: any) => { ... }
```

**Correct:**

```tsx
interface ButtonProps {
  variant?: "primary" | "secondary" | "ghost" | "text";
  size?: "small" | "medium" | "large";
  children: ReactNode;
}

function Button({ variant = "primary", size = "medium", children }: ButtonProps) {
  return (
    <button className={clsx(styles.button, styles[variant], styles[size])}>
      {children}
    </button>
  );
}

export { Button };
```

### Client Components

Add `"use client"` directive only when necessary:

**When to use `"use client"`:**
- Using React hooks (`useState`, `useEffect`, etc.)
- Using browser APIs (`window`, `localStorage`, etc.)
- Using event handlers
- Using motion/animation libraries

**Incorrect:**

```tsx
"use client"; // Unnecessary - no client features used

function StaticCard({ title }: { title: string }) {
  return <div>{title}</div>;
}
```

### Props Pattern

Use data attributes for variants instead of multiple className conditionals:

**Correct:**

```tsx
function Callout({ type = "info", children }: CalloutProps) {
  return (
    <div className={styles.callout} data-variant={type}>
      {children}
    </div>
  );
}
```

```css
.callout[data-variant="info"] {
  background: var(--blue-a3);
}

.callout[data-variant="error"] {
  background: var(--red-a3);
}
```

### Motion Components

Use `motion.create()` for Base UI components:

**Correct:**

```tsx
import { Button as BaseButton } from "@base-ui/react/button";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raphaelsalaja/userinterface-wiki](https://github.com/raphaelsalaja/userinterface-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
