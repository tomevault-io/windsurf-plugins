---
trigger: always_on
description: This document provides context, conventions, and guidelines for AI agents working on the **lynx-ui** repository.
---

# lynx-ui Agent Guidelines

This document provides context, conventions, and guidelines for AI agents working on the **lynx-ui** repository.

> **CRITICAL NAMING CONVENTION**: The name of this repository and project is **lynx-ui** (lowercase). **NEVER** use "Lynx UI", "Lynx-UI", or any other variation in documentation, comments, or output.

## Project Overview

**lynx-ui** is a UI component library for ReactLynx, built as a Monorepo using TurboRepo.

- **Framework**: React (Lynx bindings)
- **Language**: TypeScript
- **Package Manager**: pnpm
- **Build System**: TurboRepo, Rslib

## Directory Structure

This repository follows a standard Monorepo structure. Understanding this is crucial for navigating and creating files correctly.

- **`packages/`**: The core of the library. Each UI component resides in its own package here.
  - Structure: `packages/lynx-ui-<component>/`
  - Key files per package:
    - `src/`: Source code (usually `index.tsx` and component files).
    - `package.json`: Dependency management.
    - `README.md`: Documentation.
  - Example: `packages/lynx-ui-button/` contains the Button component.

- **`apps/examples/`**: Contains runnable examples for development and testing.
  - Structure: `apps/examples/<Component>/<UseCase>/`
  - Key files per example:
    - `index.tsx`: The entry point for the example.
    - `index.css`: Styles for the example.
  - Example: `apps/examples/Button/Basic/` contains a basic usage example of the Button.

- **`luna/`**: The in-repo theming foundation (L.U.N.A) used by lynx-ui (tokens, styles, Tailwind preset, ReactLynx bindings).

## Development Workflow

### 1. Initial Setup

**ALWAYS** ensure dependencies are up to date.

```bash
# Install dependencies
pnpm install
```

### 2. Creating a New Component

**NEVER** manually create a component directory. Always use the provided script to scaffold a new component to ensure correct structure and configuration.

```bash
pnpm make-new-component --create <component-name>
# Example: pnpm make-new-component --create toast
```

### 3. Running Examples

To test changes, run the specific example for the component.

1. **Create example (if missing)**:

   ```bash
   pnpm make-new-component --example <component-name>
   ```

2. **Run the dev server**:
   Check `apps/examples/README.md` or `package.json` in the example folder for the exact filter name. Typically:

   ```bash
   npx turbo watch dev --filter '@lynx-example/lynx-ui-<component-name>'
   ```

Examples should prefer importing public APIs from `@lynx-js/lynx-ui` instead of `@lynx-js/lynx-ui-<component>` or `@lynx-js/lynx-ui-common`, unless a symbol is intentionally excluded from the aggregate entry.

### 4. Build & Verify

Before submitting changes, ensure the project builds and passes checks.
**CRITICAL**: You MUST run `pnpm turbo build` to perform a full workspace build and ensure no compilation errors are introduced.

```bash
# Build all packages
pnpm run build

# Verify the aggregate lynx-ui entry re-exports the expected package surface
pnpm check:exports

# Run all checks (format, lint, manypkg, sherif)
pnpm check:all
```

**Additional Verification Tools**:

- **`pnpm check:manypkg`**: Checks for dependency mismatches in the monorepo.
- **`pnpm check:sherif`**: Lints `package.json` files for potential issues using Sherif.
- **`pnpm check:exports`**: Verifies that `@lynx-js/lynx-ui` re-exports the expected public surface from included sub-packages.
- **`pnpm spell`**: Runs CSpell to check for spelling errors.

## Coding Standards

### Headless UI Principles

This library follows the **Headless** pattern, focusing on logic, state management, and accessibility while remaining unstyled by default.

- **Separation of Concerns**: Components provide the logic (state, event handling) but minimal styling.
- **Composition**: Use sub-components (e.g., `Checkbox` + `CheckboxIndicator`) to allow flexible layout and styling.
- **State via Context**: Share state between parent and child components using React Context (e.g., `CheckboxContext`).
- **Render Props / Children**: Support `children` as a function (render props) or standard children to pass state down for dynamic styling.
- **Styling API**: Provide `className` and `style` props on all components to allow users to apply their own design system (or L.U.N.A tokens).
- **State-based Class Names**: Components should not apply default class names, but users can use `className` props to apply state-based class names.

### General

- **TypeScript**: Use strict typing. Avoid `any`.
- **Comments**: All code comments **MUST** be written in English.
- **Functional Components**: Use React Functional Components with Hooks.
- **Controlled & Uncontrolled Modes**: We encourage all interactive components to support both **controlled** and **uncontrolled** modes. This provides flexibility for users who want to manage state externally (controlled) or let the component manage its own internal state (uncontrolled). Typically, you should provide a `value` (or `show`, etc.) prop for the controlled mode, and a `defaultValue` (or `defaultShow`, etc.) prop for the uncontrolled mode.
  - **Example**:
    ```tsx
    // Popover.tsx
    function PopoverRoot(props) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lynx-family/lynx-ui](https://github.com/lynx-family/lynx-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
