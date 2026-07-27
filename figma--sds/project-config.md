---
trigger: always_on
description: Welcome to the Simple Design System repository! This is a comprehensive React-based design system with Figma integration using Code Connect. This guide will help you understand how to work with the codebase effectively.
---

# Simple Design System (SDS) Cursor Instructions

Welcome to the Simple Design System repository! This is a comprehensive React-based design system with Figma integration using Code Connect. This guide will help you understand how to work with the codebase effectively.

## Repository Overview

SDS is a production-ready design system featuring:

- 🎨 **Figma Integration**: Full Code Connect integration with variables, styles, and components
- ⚛️ **React Components**: Built on React Aria/Stately for accessibility
- 🎯 **TypeScript**: Fully typed component library
- 📚 **Storybook**: Comprehensive component documentation
- 🎨 **Design Tokens**: CSS custom properties with light/dark mode support
- 📱 **Responsive**: Mobile-first responsive design patterns

## Getting Started

### Development Commands

- `npm run app:dev` - Start development server at localhost:8000
- `npm run storybook` - Launch Storybook at localhost:6006
- `npm run app:build` - Build the application
- `npm run storybook:build` - Build Storybook
- `npm run script:tokens` - Sync design tokens from Figma
- `npm run script:icons` - Sync icons from Figma

### Key Resources

- [Live Storybook](https://figma.github.io/sds/storybook) - Component documentation
- [Figma Community File](https://www.figma.com/community/file/1380235722331273046/simple-design-system) - Design source

## Quick Start Checklist

Before implementing any Figma design, follow this checklist:

1. ✅ **Use Figma MCP tools** to extract design data when prompted.

- When trying to get a high level understanding of the design, you can use #get_image to get image representation of the design to contextualize what is selected in Figma
- Whenever you want more information about the specific UI elements that are selected, use #get_code
- With the get_code, get_code_connect_map, and get_variable_map tools you will receive information about patterns that should already be defined in this repository.
- If patterns dont already exist, you should ask for permission and clarify what about the design isn't already accounted for in this codebase that is present in the design.

2. ✅ **Read component APIs** in the actual TypeScript files before using
3. ✅ **Use SDS components only** - never import from `@react-aria`, `@react-stately`, etc.
4. ✅ **Use CSS variables** - never hardcode colors, spacing, or typography
5. ✅ **Use layout components** - never write custom CSS for positioning

## Importing Aliases

When importing SDS components, use aliases to ensure correct paths. These are described in the `vite.config.ts` file under `resolve.alias`:

```tsx
import { Footer, Header } from "compositions";
import {
  useAuth,
  usePricing,
  useProducts,
  AuthProvider,
  PricingProvider,
  ProductsProvider,
  AllProviders,
} from "data";
import { useMediaQuery } from "hooks";
import { IconChevronLeft } from "icons";
import { placeholder } from "images";
import { Flex, Section } from "layout";
import {
  Accordion,
  AccordionItem,
  Button,
  TextContentHeading,
  TextContentTitle,
} from "primitives";
```

## Figma to SDS Translation Guidelines

When implementing designs from Figma, follow these guidelines to ensure proper translation to the SDS component library.

## Annotations

In the MCP server response from Figma, you will see annotation attributes that provide context. The annotations can be describing behavior for the current node, or its descendants.

```jsx
<div
  data-content-annotations="Cards' button text is dynamic. They should say upgrade or downgrade if there is a current plan and it is better or worse."
  data-interaction-annotations="Card buttons should be disabled if they are the current plan and their text should say 'current plan'. The pill buttons should update the cards to show annual or monthly pricing."
/>
```

The prefix is a category for the annotation and describes the type of annotation. For example, "content" refers to text or image content annotations, and "interaction" refers to user interaction notes.

Always read these annotations carefully, as they provide important context for implementing the design correctly.

## CSS Variables & Design Tokens

All design tokens from Figma are defined as CSS variables in [src/theme.css](../src/theme.css). **Always use these variables instead of hardcoded values.**

- Colors: Use `var(--sds-color-*)` variables (e.g., `var(--sds-color-text-default-default)`)
- Spacing: Use `var(--sds-size-space-*)` variables (e.g., `var(--sds-size-space-400)`)
  - **important**: These variables can map to a prop value numerically. For example, `<Section padding="400">` corresponds to `padding: var(--sds-size-space-400)` in a Figma response.
- Typography: Use `var(--sds-typography-*)` or `var(--sds-font-*)` variables
- Border radius: Use `var(--sds-size-radius-*)` variables
- Shadows: Use `var(--sds-effects-shadows-*)` variables

### Example Usage

```css
.custom-element {
  background: var(--sds-color-background-default-default);
  color: var(--sds-color-text-default-default);
  padding: var(--sds-size-space-400);
  border-radius: var(--sds-size-radius-200);
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [figma/sds](https://github.com/figma/sds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
