---
trigger: always_on
description: This repository contains a Mantine extension component (`@gfazioli/mantine-window`) that renders draggable, resizable floating windows with persistent state, customizable boundaries, collapsible content, z-index management, and flexible control over position, size, and interaction modes. Includes a WindowGroup compound component for coordinated multi-window management with layout presets.
---

# Copilot Instructions for Mantine Window Component

## Project Overview

This repository contains a Mantine extension component (`@gfazioli/mantine-window`) that renders draggable, resizable floating windows with persistent state, customizable boundaries, collapsible content, z-index management, and flexible control over position, size, and interaction modes. Includes a WindowGroup compound component for coordinated multi-window management with layout presets.

## Project Purpose

The component is designed for:
- Floating window UIs (tool panels, inspectors, chat widgets)
- Multi-window layouts with drag and resize
- Persistent window state across sessions (localStorage)
- Customizable window boundaries and constraints

## Technology Stack

- **Framework**: React 19.2.0
- **UI Library**: Mantine 8.3.8
- **Language**: TypeScript 5.9.3
- **Build Tool**: Rollup
- **Package Manager**: Yarn 4.11.0
- **Testing**: Jest with React Testing Library
- **Linting**: ESLint with Mantine config
- **Code Formatting**: Prettier with sort-imports plugin
- **CSS**: PostCSS with Mantine preset
- **Documentation**: Storybook and custom docs site

## Repository Structure

- `package/`: Main package source code
  - `src/`: Component source files
    - `Window.tsx`: Main component
    - `Window.module.css`: Component styles
    - `Window.test.tsx`: Component tests
    - `lib/`: Utility functions and helpers
- `docs/`: Documentation site
- `scripts/`: Build and release scripts
- `.storybook/`: Storybook configuration
- `@types/`: TypeScript type definitions

## Build and Development Process

### Installation
```bash
yarn install
```

### Development
```bash
npm run dev          # Start dev server (docs site)
npm run storybook    # Start Storybook on port 8271
```

### Building
```bash
npm run build        # Build the package
npm run docs:build   # Build documentation site
```

### Testing
```bash
npm test             # Run all tests (syncpack, prettier, typecheck, lint, jest)
npm run jest         # Run Jest tests only
npm run typecheck    # TypeScript type checking
```

### Linting and Formatting
```bash
npm run lint                # Run both ESLint and Stylelint
npm run eslint              # Run ESLint
npm run stylelint           # Run Stylelint for CSS
npm run prettier:check      # Check code formatting
npm run prettier:write      # Format code
npm run syncpack            # Check dependency version mismatches
```

## Coding Standards and Technical Principles

### General Guidelines
1. **Minimal Changes**: Make the smallest possible changes to achieve the goal
2. **Type Safety**: All code must be fully typed with TypeScript
3. **Code Quality**: Follow the existing ESLint and Prettier configurations
4. **Testing**: Write tests for new features and bug fixes
5. **Documentation**: Update Storybook stories for component changes

### TypeScript
- Target: ES2015
- Module: ESNext
- JSX: React
- Use strict type checking
- Avoid `any` types

### React
- Use functional components with hooks
- Follow React 19 best practices
- Use Mantine components and hooks where applicable
- Component props should be well-typed interfaces

### CSS
- Use CSS Modules (`.module.css` files)
- Follow PostCSS preset for Mantine
- Use Mantine's styling tokens and utilities
- Ensure styles pass Stylelint checks

### Testing
- Use Jest with React Testing Library
- Write unit tests for components
- Test accessibility with jest-axe
- Follow existing test patterns in `*.test.tsx` files

### Git Workflow
- Branch naming: Use descriptive names (feature/*, fix/*, etc.)
- Commits: Write clear, concise commit messages
- PRs: Ensure all CI checks pass before merging

## Key Files and Their Purpose

- `package/src/Window.tsx`: Main component implementation
- `package/src/lib/`: Helper functions and utilities
- `package/package.json`: Package metadata and dependencies
- `rollup.config.mjs`: Build configuration
- `tsconfig.json`: TypeScript configuration
- `eslint.config.mjs`: ESLint configuration
- `.prettierrc.mjs`: Prettier configuration
- `.stylelintrc.json`: Stylelint configuration

## Dependencies

### Core Dependencies
- `@mantine/core`: UI component library
- `@mantine/hooks`: React hooks library
- `react` and `react-dom`: React framework

### Build and Dev Dependencies
- `rollup`: Module bundler
- `typescript`: Type checking
- `eslint`: Code linting
- `prettier`: Code formatting
- `jest`: Testing framework
- `storybook`: Component documentation

## Important Constraints

1. **Mantine Version**: Keep in sync with Mantine 8.3.8 (or compatible versions)
2. **React Version**: Using React 19 - be aware of any API changes
3. **Node Version**: Use Node.js 20.9.0 (specified in workflows)
4. **Package Manager**: Must use Yarn 4.11.0
5. **Browser Support**: Target modern browsers supporting ES2015
6. **Breaking Changes**: Avoid breaking the public API without major version bump

## Release Process

- `npm run release:patch`: Patch version bump

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gfazioli/mantine-window](https://github.com/gfazioli/mantine-window) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
