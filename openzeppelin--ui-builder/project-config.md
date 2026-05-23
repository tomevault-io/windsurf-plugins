---
trigger: always_on
description: Use this rule when you are building any UI.
---


# Tech Stack Overview

This project uses a modern React + TypeScript frontend stack with Vite as the build tool, organized as a multi‑package monorepo. Key packages include:

- **@openzeppelin/ui-builder-app (packages/builder)**: Main builder application with the UI and export system
- **@openzeppelin/ui-builder-renderer (packages/renderer)**: Shared rendering library
- **@openzeppelin/ui-builder-ui (packages/ui)**: Shared UI component library
- **@openzeppelin/ui-builder-types (packages/types)**: Shared type system (single source of truth, includes ContractAdapter)
- **@openzeppelin/ui-builder-styles (packages/styles)**: Centralized styling system
- **@openzeppelin/ui-builder-utils (packages/utils)**: Shared utilities (logger, AppConfigService, cn, etc.)
- **@openzeppelin/ui-builder-react-core (packages/react-core)**: Shared React providers/hooks (adapter and wallet state)
- **@openzeppelin/ui-builder-storage (packages/storage)**: Dexie/IndexedDB persistence for builder history
- **@openzeppelin/ui-builder-adapter-_ (packages/adapter-_)**: Chain‑specific adapters (EVM, Solana, Stellar, Midnight, ...)

Key technologies include:

- React 19 - UI library with modern hooks API and concurrent features
- TypeScript 5.8+ - Static typing with enhanced type inference
- Vite 6/7 - Fast build tool and dev server with HMR
- pnpm workspaces - Monorepo package management
- Tailwind CSS v4 - Utility-first CSS framework with new HSL theme syntax
- shadcn/ui - Unstyled, accessible component system built on Radix UI
- Vitest - Testing framework integrated with Vite
- pnpm - Fast, disk-efficient package manager
- ESLint 9 + Prettier - Code quality and formatting tools
- Husky + lint-staged - Git hooks for quality assurance
- Conventional Commits - Structured commit message format

# Framework-Specific Rules

When working with React components:

- Use functional components with hooks
- Prefer destructuring props
- Keep components focused on a single responsibility
- Extract reusable logic into custom hooks
- Use memoization (useMemo, useCallback) for expensive operations
- Use React.ComponentRef instead of React.ElementRef

# Monorepo Architecture

Workspace structure (selected):

- **packages/builder** - Main application with builder UI and export system
- **packages/renderer** - Shared rendering library
- **packages/ui** - Shared UI components
- **packages/types** - Shared types
- **packages/styles** - Centralized styles
- **packages/utils** - Utilities
- **packages/react-core** - React providers/hooks
- **packages/storage** - IndexedDB storage
- **packages/adapter-\*** - Chain‑specific adapters

Guidelines for monorepo development:

- Keep core rendering logic in the renderer package
- The builder app imports shared libraries by published package names (no cross‑package source imports outside configured aliases/virtual modules)
- Each package has its own tsconfig.json extending from the root tsconfig.base.json
- Run commands with package filtering: `pnpm --filter=@openzeppelin/ui-builder-app <command>`
- Use workspace-level commands when applicable: `pnpm -r <command>` (runs in all packages)
- Keep package.json scripts synchronized between packages for consistency

# Frontend Architecture

Architecture patterns:

- Component-driven development with composition patterns
- Separation of UI components from business logic
- Headless UI pattern (shadcn/ui + Radix) for accessibility and customization
- Atomic design principles for component organization
- Custom hooks for shared stateful logic
- Utility functions for shared stateless logic
- Shared form rendering logic in the form-renderer package

# TypeScript

[tsconfig.json](mdc:packages/builder/tsconfig.json)
[tsconfig.json](mdc:packages/renderer/tsconfig.json)
[tsconfig.base.json](mdc:tsconfig.base.json)
[tsconfig.json](mdc:tsconfig.json)

When writing TypeScript:

- Use explicit return types for functions and React components
- Prefer interfaces for public APIs and types for internal usage
- Use proper type narrowing instead of type assertions (avoid `as` casts)
- Leverage TypeScript's utility types (Partial, Pick, Omit, etc.)
- Prefer readonly arrays and properties when data shouldn't change
- Use discriminated unions for state management
- Ensure strict null checks by avoiding `!` assertions
- Use path aliases for imports between packages:
  - Within builder: `@/` for src directory
  - Use published package imports (e.g., `@openzeppelin/ui-builder-renderer`) and configured aliases

# Tailwind CSS v4

[tailwind.config.cjs](mdc:packages/builder/tailwind.config.cjs)
[postcss.config.cjs](mdc:postcss.config.cjs)
[index.css](mdc:packages/builder/src/index.css)

When using Tailwind CSS v4:

- Use centralized `@styles/global.css` and semantic classes (`bg-primary`, etc.)
- Use simplified class names like `bg-primary` instead of verbose `bg-[hsl(var(--color))]` syntax
- Define colors using OKLCH format for better color reproduction (e.g., `--color-primary: oklch(0.5 0.2 120)`)
- Use Tailwind's built-in dark mode with the 'class' strategy
- Use component composition rather than extending with apply when possible
- For complex UI patterns, create reusable components instead of utility classes
- Follow the container pattern for responsive layouts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenZeppelin/ui-builder](https://github.com/OpenZeppelin/ui-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
