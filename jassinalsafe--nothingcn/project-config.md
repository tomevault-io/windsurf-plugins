---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint checks

## Project Architecture

This is NothingCN - an open source Next.js 15 component showcase built with React 19, TypeScript, and Tailwind CSS 4. The goal is to create a comprehensive library of creative, reusable components that developers can easily copy and use in their projects.

### Key Architecture Patterns

- **App Router Structure**: Uses Next.js App Router with pages organized in `src/app/`
- **Component-First Design**: All UI components are built with Radix UI primitives and use the compound component pattern
- **Variant System**: Components use `class-variance-authority` for consistent styling variants
- **Theme System**: CSS custom properties for theming with built-in dark mode support
- **Utility-First Styling**: Tailwind CSS with `cn()` utility function for class merging

### Core Dependencies

- **UI Primitives**: Radix UI for accessible, unstyled components
- **Styling**: Tailwind CSS 4 with `clsx` and `tailwind-merge`
- **Icons**: Lucide React
- **Code Highlighting**: Prism React Renderer for syntax highlighting
- **Fonts**: Geist Sans and Geist Mono

### Component Structure

All reusable components are in `src/components/ui/` and follow these patterns:
- Use `React.forwardRef` for proper ref handling
- Export both component and variant functions
- Use `cn()` utility for class merging
- Support `asChild` prop pattern via Radix Slot when applicable

### Navigation Structure

- `/` - Homepage showcasing NothingCN
- `/components` - Creative component library with copy-paste functionality
- `/blocks` - Pre-built component combinations and layouts
- `/themes` - Theme customization and color schemes
- `/docs` - Documentation and contribution guidelines

### Project Goals

- **Open Source**: All components are freely available for developers to use
- **Creative Focus**: Emphasis on unique, visually appealing components
- **Copy-Paste Ready**: No installation required - just copy the code
- **Community Driven**: Built for and by the developer community

### Styling Guidelines

- Use CSS custom properties defined in `globals.css` for theming
- Dark mode is controlled by adding/removing `dark` class on document element
- All components support both light and dark themes automatically
- Responsive design follows mobile-first approach

## Component Showcase Philosophy

### Dogfooding Principle
As a component library, NothingCN must demonstrate its own value by extensively using its own components. This means:

- **Every showcase page MUST use established reusable components**:
  - `InstallationTabs` for installation sections
  - `ComponentPreview` for variant demonstrations
  - `ComponentCode` for source code display
  - `ComponentLayout` for consistent page structure
  - `ComponentNavigation` for page-to-page navigation

- **NEVER create manual implementations when a reusable component exists**
- **Always check for existing patterns before creating new ones**
- **If a pattern is used more than once, it should become a reusable component**

### Demonstration Best Practices

When creating component showcase pages:

1. **Reuse Over Recreate**: Always use existing components from the library
2. **Consistent Structure**: Follow the established section patterns (installation → usage → variants → source)
3. **Clear Examples**: Each code example should be focused and demonstrate one concept
4. **Real-World Usage**: Show how components work together in practical scenarios
5. **Accessibility First**: All examples must be keyboard navigable and screen reader friendly

### Component Development Checklist

Before creating a new component showcase page, ensure:
- [ ] Uses `ComponentLayout` for page structure
- [ ] Uses `InstallationTabs` for installation instructions
- [ ] Uses `ComponentPreview` for all demonstrations
- [ ] Uses `ComponentCode` for source code display
- [ ] Includes proper navigation with `getComponentNavigation`
- [ ] Follows CVA pattern for variants
- [ ] Includes all variant examples (default, nothing, pixel, minimal, etc.)
- [ ] Has clear, focused code examples for each variant
- [ ] Demonstrates the value of the component library through reuse

This approach ensures that NothingCN doesn't just provide components - it demonstrates their value by using them throughout its own documentation.

## Component Page Layout Standards

### Required Page Structure

All component documentation pages must follow this exact structure for consistency:

#### 1. **Imports and Setup**
```tsx
"use client";

import { ComponentPreview } from "@/components/component-preview";
import { ComponentCode } from "@/components/component-code";
import { ComponentLayout } from "@/components/component-layout";
import { InstallationTabs } from "@/components/installation-tabs";
import { getComponentNavigation } from "@/lib/component-navigation";
import { 
  // Import code examples from separate files
} from "./examples";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JassinAlSafe/NothingCN](https://github.com/JassinAlSafe/NothingCN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
