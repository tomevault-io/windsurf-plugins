---
trigger: always_on
description: Reablocks is a React component library built on Tailwind CSS, providing 50+ UI components for building modern web applications. The library supports both dark and light themes and uses a powerful theming system that allows complete customization.
---

# CLAUDE.md - Reablocks AI Assistant Guide

## Project Overview

Reablocks is a React component library built on Tailwind CSS, providing 50+ UI components for building modern web applications. The library supports both dark and light themes and uses a powerful theming system that allows complete customization.

**Version:** 9.3.0
**License:** Apache-2.0
**Repository:** https://github.com/reaviz/reablocks

## Tech Stack

- **React** (>=16) - Component framework
- **TypeScript** - Type safety
- **Tailwind CSS 4** - Styling framework
- **Vite** - Build tool and dev server
- **Storybook 9** - Component development and documentation
- **Vitest** - Testing framework
- **Motion (Framer Motion)** - Animations
- **Floating UI** - Positioning for tooltips, popovers, etc.

## Directory Structure

```
src/
├── data/           # Data display components (DataSize, DateFormat, Duration, Ellipsis, Pager, Pluralize, Redact, Sort)
├── elements/       # Basic UI elements (Avatar, Badge, Button, Chip, CommandPalette, IconButton, Kbd, Loader, Skeleton)
├── form/           # Form components (Calendar, Checkbox, DateInput, Input, Radio, Range, Select, Textarea, Toggle)
├── layers/         # Overlay components (Backdrop, Callout, ConfirmDialog, ContextMenu, Dialog, Drawer, Menu, Notification, Popover, Tooltip)
├── layout/         # Layout components (Breadcrumbs, Card, Collapse, Divider, Field, List, Motion, Stepper, Tabs, Tree)
├── typography/     # Typography components (H1, H2, H3, H4, P, BlockQuote, Lead, Large, Small, Muted)
├── utils/          # Utilities (ExitListener, Overlay, Portal, Position, Theme system, hooks)
├── assets/         # SVG icons and assets
└── index.ts        # Main export file

docs/
├── blocks/         # Pre-built page templates (authentication, administration, foundation)
└── assets/         # Documentation assets
```

## Development Workflow

### Getting Started

```bash
npm install        # Install dependencies
npm start          # Start Storybook on port 9009
```

### Available Scripts

| Command | Description |
|---------|-------------|
| `npm start` | Start Storybook development server |
| `npm run build` | Full production build (JS + styles + docs) |
| `npm run build:js` | Build library with Vite |
| `npm run build:styles` | Build Tailwind CSS |
| `npm run build-storybook` | Build static Storybook |
| `npm run lint` | Run ESLint |
| `npm run lint:fix` | Fix ESLint issues |
| `npm run prettier` | Format code with Prettier |
| `npm test` | Run tests with Vitest |
| `npm run test:coverage` | Run tests with coverage |

### Requirements

- Node.js >= 22
- npm >= 10.8.2

## Code Conventions

### File Naming

- Components: `ComponentName.tsx`
- Themes: `ComponentNameTheme.ts`
- Stories: `ComponentName.story.tsx`
- Tests: `ComponentName.spec.ts` or `utils.spec.ts`
- Index files: `index.ts` (barrel exports)

### Import Paths

Use the `@/` alias for imports from the `src` directory:

```typescript
import { useComponentTheme } from '@/utils';
import { ButtonTheme } from '@/elements';
```

ESLint enforces no relative import paths (except same folder):

```typescript
// Good
import { Button } from '@/elements/Button';

// Allowed (same folder)
import { ButtonTheme } from './ButtonTheme';

// Bad
import { Button } from '../../elements/Button';
```

### Component Structure

Each component follows this pattern:

```typescript
'use client';  // For Next.js compatibility (when needed)

import React, { FC, forwardRef } from 'react';
import { twMerge } from 'tailwind-merge';
import { useComponentTheme } from '@/utils';
import { ComponentNameTheme } from './ComponentNameTheme';

export interface ComponentNameProps {
  /**
   * JSDoc comment for prop
   */
  propName?: string;

  /**
   * Theme for the component
   */
  theme?: ComponentNameTheme;
}

export const ComponentName: FC<ComponentNameProps> = ({
  propName = 'default',
  theme: customTheme,
  className,
  ...rest
}) => {
  const theme = useComponentTheme('componentName', customTheme);

  return (
    <div className={twMerge(theme.base, className)} {...rest}>
      {/* Component content */}
    </div>
  );
};
```

### Theme Structure

Every component has a corresponding theme interface and default theme:

```typescript
// ComponentNameTheme.ts
export interface ComponentNameTheme {
  base: string;
  variants: {
    primary: string;
    secondary: string;
    [key: string]: string;
  };
  sizes: {
    small: string;
    medium: string;
    large: string;
    [key: string]: string;
  };
}

export const componentNameTheme: ComponentNameTheme = {
  base: 'flex items-center',
  variants: {
    primary: 'bg-primary text-white',
    secondary: 'bg-secondary text-gray-900'
  },
  sizes: {
    small: 'text-sm p-2',
    medium: 'text-base p-4',
    large: 'text-lg p-6'
  }
};

// Also export a legacy theme for CSS variable-based theming
export const legacyComponentNameTheme: ComponentNameTheme = {
  // Uses CSS variables like var(--primary-background)
};
```

### Styling Guidelines

1. Use Tailwind utility classes exclusively

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reaviz/reablocks](https://github.com/reaviz/reablocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
