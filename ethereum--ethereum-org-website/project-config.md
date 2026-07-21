---
trigger: always_on
description: This is the official Ethereum.org website - a Next.js application that serves as the primary educational and community hub for Ethereum. The site is built with modern web technologies and focuses on accessibility, internationalization, and performance.
---

# CLAUDE.md - Ethereum.org Website

## Project Overview

This is the official Ethereum.org website - a Next.js application that serves as the primary educational and community hub for Ethereum. The site is built with modern web technologies and focuses on accessibility, internationalization, and performance.

## Technology Stack

### Core Framework

- **Next.js 16+** - React framework with App Router (Turbopack is the default bundler; webpack available via `pnpm dev:webpack` / `pnpm build:webpack`)
- **React 19** - UI library
- **TypeScript 5.5+** - Type safety and development experience
- **Tailwind CSS 4+** - Utility-first CSS framework (CSS-first config in `src/styles/global.css`)

### Key Dependencies

- **next-intl 4+** - Internationalization (i18n) with 25 languages
- **next-mdx-remote 5.0+** - MDX content processing
- **Motion 12+** (`motion` package, formerly Framer Motion) - Animations and transitions; import from `motion/react`
- **Radix UI** - Accessible component primitives
- **shadcn/ui** - Component library built on Radix UI
- **Recharts** - Data visualization
- **Viem/Wagmi** - Ethereum blockchain integration

### Development & Testing

- **Storybook 10+** - Component development and testing
- **Chromatic** - Visual regression testing
- **ESLint** - Code linting with custom rules
- **Prettier** - Code formatting
- **Husky** - Git hooks
- **PNPM** - Package manager

## Project Structure

- **app/** - Next.js App Router pages
  - **[locale]/** - Internationalized routes
- **src/**
  - **components/** - React components
    - **ui/** - Design system components
    - **icons/** - SVG icon components
  - **data/** - Static data and configurations
  - **hooks/** - Custom React hooks
  - **i18n/** - Internationalization config
  - **intl/** - Translation files (25 languages)
  - **layouts/** - Page layout components
  - **lib/** - Utility functions and types
    - **constants.ts** - App constants
    - **types.ts** - TypeScript type definitions
    - **utils/** - Utility functions
  - **styles/** - Global styles and design tokens
- **public/** - Static assets
  - **content/** - Markdown content files
  - **images/** - Image assets
- **docs/** - Development documentation
  - **solutions/** - Documented solutions to past problems, organized by category with YAML frontmatter (module, tags, problem_type)

## Code Conventions

### File Naming

- **Components**: kebab-case (e.g., `button-group.tsx`)
- **Utilities**: camelCase (e.g., `cn.ts`, `relativePath.ts`)
- **Pages**: kebab-case following Next.js conventions
- **Assets**: kebab-case (e.g., `eth-logo.png`)

### TypeScript Patterns

- Use `interface` for object shapes, `type` for unions/intersections
- Prefer explicit typing over `any` (ESLint enforces `fixToUnknown`)
- **NEVER leave unused variables or parameters** - ESLint `unused-imports/no-unused-vars` will fail the Netlify build. The only allowed unused arg pattern is a single underscore `_`. Do NOT use `_prefixedNames` (e.g., `_foo`) - either use the variable or remove it from the signature entirely.
- Use generic constraints for reusable components
- Export types from dedicated files in `@/lib/types`

### Styling Conventions

- **Primary approach**: Tailwind CSS utility classes
- **Component variants**: Use `tailwind-variants` (`tv`) for new and refactored work. Existing `class-variance-authority` (`cva`) components don't need bulk migration -- swap to `tv` opportunistically when you're already touching the component for another reason.
- **Dynamic classes**: Use `cn()` utility (clsx + tailwind-merge)
- **Custom properties**: CSS variables in `src/styles/` for theme values
- **Responsive design**: Mobile-first approach

For UI work, see the **`design-system` skill** at `.claude/skills/design-system/`. It's the canonical knowledge base for component choices, design tokens, RTL/i18n, server/client boundaries, and the "use a variant, not a new component" pattern.

## Development Workflows

### Available Scripts

```bash
# Development
pnpm dev                    # Start development server (Turbopack)
pnpm dev:webpack            # Start development server with webpack
pnpm build                  # Build for production (Turbopack)
pnpm build:webpack          # Build for production with webpack
pnpm start                  # Start production server

# Code Quality
pnpm lint                   # Run ESLint
pnpm lint:fix              # Fix ESLint issues
pnpm type-check            # TypeScript type checking
pnpm format                # Format with Prettier

# Testing
pnpm test:unit             # Playwright unit tests (unit project)
pnpm test:e2e              # Playwright end-to-end tests
pnpm test:visual           # Playwright + Chromatic full-page visual tests

# Storybook
pnpm storybook             # Start Storybook dev server
pnpm build-storybook       # Build Storybook
pnpm chromatic             # Run Chromatic visual tests

# Content Management
pnpm lint:md               # Lint English markdown content
pnpm lint:md:fix           # Auto-fix header IDs and duplicates
```

### Testing Strategy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ethereum/ethereum-org-website](https://github.com/ethereum/ethereum-org-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
