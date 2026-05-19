---
trigger: always_on
description: Comprehensive SCSS styling guidelines for the OpenUI React UI component library
---


# OpenUI Styling System

## Overview

OpenUI uses a comprehensive design system with SCSS utilities, CSS custom properties, and consistent component patterns. All styling must follow these guidelines to maintain design consistency and component reusability.

## Core Principles

### 1. Use cssUtils.scss for All Design Tokens

**Always import and use `cssUtils.scss` instead of hardcoded values:**

```scss
@use "../../cssUtils" as cssUtils;

// ✅ Correct - Use design tokens
.my-component {
  background-color: cssUtils.$foreground;
  color: cssUtils.$text-neutral-primary;
  padding: cssUtils.$space-m;
  border-radius: cssUtils.$radius-m;
  @include cssUtils.typography(body, default);
}

// ❌ Incorrect - Hardcoded values
.my-component {
  background-color: #ffffff;
  color: #000000;
  padding: 16px;
  border-radius: 8px;
  font-family: "Inter", sans-serif;
}
```

### 2. Component Architecture

#### File Structure

Each component should follow this structure:

```FileStructure
components/ComponentName/
├── ComponentName.tsx      # Main component
├── ComponentName.scss     # Component styles
├── index.ts              # Exports
├── dependencies.ts       # External dependencies
└── stories/              # Storybook stories
    └── ComponentName.stories.tsx
```

#### CSS Class Naming Convention

Use the `.openui-component-name` prefix with BEM-like modifiers:

```scss
.openui-button {
  // Base styles

  &-primary {
    // Primary variant styles
  }

  &-secondary {
    // Secondary variant styles
  }

  &-small {
    // Small size variant
  }

  &-large {
    // Large size variant
  }

  &__icon {
    // Component element (like icon inside button)
  }

  &--disabled {
    // Component state modifier
  }
}
```

---

## Color System

### Surface / Background Colors

| Token                  | Usage                           |
| ---------------------- | ------------------------------- |
| `$background`          | Main page/app background        |
| `$foreground`          | Card/container backgrounds      |
| `$popover-background`  | Popover/dropdown backgrounds    |
| `$overlay`             | Modal/overlay backdrop          |
| `$sunk-light`          | Lightly recessed surface        |
| `$sunk`                | Recessed surface (inputs)       |
| `$sunk-deep`           | Deeply recessed surface         |
| `$elevated-light`      | Lightly elevated surface        |
| `$elevated`            | Elevated surface                |
| `$elevated-strong`     | Strongly elevated surface       |
| `$elevated-intense`    | Intensely elevated surface      |
| `$inverted-background` | Inverted/dark backgrounds       |
| `$highlight-subtle`    | Subtle highlight (hover states) |
| `$highlight`           | Default highlight               |
| `$highlight-strong`    | Strong highlight                |
| `$highlight-intense`   | Intense highlight (selection)   |
| `$info-background`     | Informational backgrounds       |
| `$success-background`  | Success state backgrounds       |
| `$alert-background`    | Warning/alert backgrounds       |
| `$danger-background`   | Error/danger state backgrounds  |
| `$purple-background`   | Purple accent backgrounds       |
| `$pink-background`     | Pink accent backgrounds         |

```scss
.openui-card {
  background-color: cssUtils.$foreground;
}

.openui-modal-backdrop {
  background-color: cssUtils.$overlay;
}

.openui-input {
  background-color: cssUtils.$sunk;
}

.openui-error-banner {
  background-color: cssUtils.$danger-background;
}
```

### Interactive Colors

**Accent (Primary Actions):**

| Token                          | Usage                   |
| ------------------------------ | ----------------------- |
| `$interactive-accent-default`  | Primary button default  |
| `$interactive-accent-hover`    | Primary button hover    |
| `$interactive-accent-pressed`  | Primary button pressed  |
| `$interactive-accent-disabled` | Primary button disabled |

**Destructive (Danger Actions):**

| Token                                      | Usage                       |
| ------------------------------------------ | --------------------------- |
| `$interactive-destructive-default`         | Destructive ghost default   |
| `$interactive-destructive-hover`           | Destructive ghost hover     |
| `$interactive-destructive-pressed`         | Destructive ghost pressed   |
| `$interactive-destructive-disabled`        | Destructive ghost disabled  |
| `$interactive-destructive-accent-default`  | Filled destructive default  |
| `$interactive-destructive-accent-hover`    | Filled destructive hover    |
| `$interactive-destructive-accent-pressed`  | Filled destructive pressed  |
| `$interactive-destructive-accent-disabled` | Filled destructive disabled |

```scss
// Primary button example
.openui-button-primary {
  background-color: cssUtils.$interactive-accent-default;

  &:not(:disabled):hover {
    background-color: cssUtils.$interactive-accent-hover;
  }

  &:not(:disabled):active {
    background-color: cssUtils.$interactive-accent-pressed;
  }

  &:disabled {
    background-color: cssUtils.$interactive-accent-disabled;
  }
}

// Destructive button example
.openui-button-destructive {
  background-color: cssUtils.$interactive-destructive-accent-default;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thesysdev/openui](https://github.com/thesysdev/openui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
