---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

HeroUI v3 is a modern React UI library built with Tailwind CSS v4, using a pnpm monorepo structure managed by Turborepo.

### Key Technical Stack

- **Node.js**: v22+ required
- **pnpm**: v10.9.0 (package manager)
- **React**: v19+
- **Tailwind CSS**: v4.1.4
- **TypeScript**: v5.8.3
- **Turborepo**: Build orchestration
- **Storybook**: Component development
- **Vitest**: Testing framework

## Development Commands

### Core Development Commands

```bash
# Install dependencies (use --hoist flag)
pnpm i --hoist

# Start Storybook for component development
pnpm dev

# Start documentation site
pnpm dev:docs

# Build all packages
pnpm build

# Build specific package
pnpm build --filter=@heroui/react

# Run linting
pnpm lint

# Run tests
pnpm test

# Test specific component
pnpm test button

# Run formatting
pnpm run format

# Run type checking
pnpm typecheck
```

### Package-Specific Commands

- Use `--filter` flag with package name: `pnpm build --filter=@heroui/react`
- Main packages: `@heroui/react`, `@heroui/docs`, `@heroui/storybook`

## Git Commit Convention

**IMPORTANT**: This repository uses conventional commits with strict validation. All commits must follow this format:

```
<type>(<scope>): <message>
```

### Allowed Types:

- `feat` / `feature`: New features
- `fix`: Bug fixes
- `refactor`: Code refactoring
- `docs`: Documentation changes
- `build`: Build system changes
- `test`: Test changes
- `ci`: CI configuration changes
- `chore`: Other changes

### Examples:

```bash
git commit -m "feat(components): add new prop to avatar component"
git commit -m "fix(button): resolve click handler issue"
git commit -m "docs: update installation guide"
git commit -m "ci: add Claude Code GitHub Action workflow"
```

**Note**: Commits without proper format will be rejected by git hooks.

## Repository Architecture

### Monorepo Structure

```
/
├── apps/
│   └── docs/          # Documentation site (Next.js + Fumadocs)
├── packages/
│   ├── react/         # Main UI component library
│   ├── standard/      # Shared ESLint, Prettier, TypeScript configs
│   ├── storybook/     # Storybook configuration
│   └── vitest/        # Shared Vitest configurations
├── turbo.json         # Turborepo configuration
└── pnpm-workspace.yaml # Workspace definition
```

### Component Architecture Pattern

Each component in `packages/react/src/components/` follows this structure:

```
component-name/
├── component-name.tsx      # Main component (uses React Aria)
├── component-name.styles.ts # Tailwind Variants styling
├── component-name.stories.tsx # Storybook stories (title: "Components/ComponentName")
└── index.ts               # Barrel exports
```

**IMPORTANT**: All Storybook stories must use the "Components" group in their title. For example: `title: "Components/Card"`, `title: "Components/Button"`, etc.

### CSS Class Naming Convention

**IMPORTANT**: HeroUI v3 uses BEM (Block Element Modifier) style for CSS classes to ensure predictable and maintainable styling:

- **Block**: The main component class (e.g., `button`, `card`, `alert`)
- **Modifier**: Variations of the component using double dashes (e.g., `button--primary`, `button--lg`, `button--icon-only`)
- **Element**: Child elements within a component (e.g., `card__header`, `alert__icon`)

**Migration to CSS-based Styling**:

- The `button` component has been migrated to use CSS styles from `@heroui/styles/src/components/button.css`
- This approach allows for better customization through CSS utilities and `@utility` directives
- Other components will gradually be migrated to follow this CSS-based pattern
- Components use `tv()` from `tailwind-variants` to map variant props to BEM class names

**Default Size Pattern**:

**CRITICAL**: All components MUST include default sizes in their base classes to prevent broken appearances when no size modifier is specified. Following the following pattern:

- **Base classes** include default dimensions (equivalent to the `--md` variant)
- **Medium variants** (`--md`) are empty with explanatory comments
- **Size modifiers** override the defaults when specified

Example implementation:

```css
/* Base component with default size */
.avatar {
  @apply relative flex size-10 shrink-0 overflow-hidden rounded-full;
  /* size-10 is the default, equivalent to --md */
}

/* Size variants */
.avatar--sm {
  @apply size-8; /* Override default */
}

.avatar--md {
  /* No styles as this is the default size */
}

.avatar--lg {
  @apply size-12; /* Override default */
}
```

This ensures components work properly without explicit size classes:

- `<div className="avatar">` → Works perfectly (size-10)
- `<div className="avatar avatar--lg">` → Override to large (size-12)

### Core Component Design Principles

**IMPORTANT**: HeroUI v3 follows a compound component pattern similar to Radix UI, built on top of React Aria Components primitives. This enables maximum flexibility and customization for users.

### React Aria Components Integration

**CRITICAL**: Before implementing any component, you MUST:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heroui-inc/heroui](https://github.com/heroui-inc/heroui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
