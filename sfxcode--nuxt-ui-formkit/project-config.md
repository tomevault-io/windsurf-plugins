---
trigger: always_on
description: This is a Nuxt module that integrates FormKit with Nuxt UI components. The module provides seamless integration between FormKit form handling and Nuxt UI's design system.
---

# GitHub Copilot Instructions for nuxt-ui-formkit

## Project Overview
This is a Nuxt module that integrates FormKit with Nuxt UI components. The module provides seamless integration between FormKit form handling and Nuxt UI's design system.

## Technology Stack
- **Framework**: Nuxt 4.x
- **Language**: TypeScript
- **Package Manager**: pnpm
- **Testing**: Vitest
- **Linting**: ESLint 9
- **Build Tool**: @nuxt/module-builder
- **Key Dependencies**:
  - @nuxt/ui 4.3.0
  - @nuxt/kit for module development
  - Vue 3 with Composition API

## Project Structure
- `/src/module.ts` - Main module definition and setup
- `/src/runtime/` - Runtime code that gets bundled with the module
  - `plugin.ts` - Nuxt plugin for client-side initialization
- `/playground/` - Development playground for testing the module
- `/test/` - Vitest test files and fixtures

## Coding Standards & Best Practices

### TypeScript
- Use strict TypeScript with proper type annotations
- Export interfaces for all module options
- Leverage Nuxt's auto-imports (no need to import composables like `useNuxtApp`, `useState`, etc.)
- Use `#app` and `#imports` for Nuxt-specific imports

### Nuxt Module Development
- Follow Nuxt Module Best Practices
- Use `@nuxt/kit` utilities for module operations:
  - `addPlugin()` for registering plugins
  - `addComponent()` for auto-importing components
  - `addImports()` for auto-importing composables
  - `createResolver()` for resolving paths
- Always use `defineNuxtModule<ModuleOptions>()` with typed options
- Module meta should include `name` and `configKey`

### Vue/Nuxt Component Guidelines
- Use `<script setup>` syntax for all Vue components
- Prefer Composition API over Options API
- Use TypeScript with `<script setup lang="ts">`
- Follow Nuxt 3+ auto-import conventions
- Use Nuxt UI components when building UI

### Nuxt UI Integration
- **Version**: This project uses @nuxt/ui 4.3.0
- **Component Usage**: Leverage Nuxt UI components for consistent design system integration
- **FormKit Integration**: Map FormKit input types to appropriate Nuxt UI components
- **Theming**: Respect Nuxt UI's color modes (light/dark) and theme configuration
- **Component Props**: Follow Nuxt UI's prop patterns and naming conventions
- **Icons**: Use Nuxt UI's icon integration (typically with `@iconify/vue` or similar)
- **Accessibility**: Maintain Nuxt UI's built-in accessibility features when creating custom integrations
- **Utilities**: Use Nuxt UI utilities and composables (e.g., `useToast`, `useModal`, `useColorMode`)
- **Styling**: Prefer Nuxt UI's utility classes and design tokens over custom CSS
- **Documentation**: Reference [Nuxt UI documentation](https://ui.nuxt.com/) for component APIs and patterns
- **Component Examples**: Common Nuxt UI components to integrate with FormKit:
  - `UInput` - Text inputs
  - `UTextarea` - Multi-line text
  - `USelect` - Dropdowns and selects
  - `UCheckbox` - Checkboxes
  - `URadio` - Radio buttons
  - `UButton` - Submit buttons and actions
  - `UFormGroup` - Form field wrappers with labels and help text
  - `UAlert` - Error and validation messages

### Code Style
- Use 2-space indentation
- Use single quotes for strings
- Add trailing commas in objects and arrays
- Follow ESLint configuration (@nuxt/eslint-config)
- No semicolons (as per typical Vue/Nuxt conventions)

### Conventional Commits
- **Format**: Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification for all commit messages
- **Structure**: `<type>[optional scope]: <description>`
- **Types**:
  - `feat:` - New features (triggers minor version bump)
  - `fix:` - Bug fixes (triggers patch version bump)
  - `docs:` - Documentation changes only
  - `style:` - Code style changes (formatting, missing semicolons, etc.)
  - `refactor:` - Code changes that neither fix bugs nor add features
  - `perf:` - Performance improvements
  - `test:` - Adding or updating tests
  - `chore:` - Maintenance tasks, dependency updates, build config
  - `ci:` - CI/CD configuration changes
  - `build:` - Build system or external dependency changes
- **Breaking Changes**: Use `BREAKING CHANGE:` in the footer or add `!` after type/scope (e.g., `feat!:` or `feat(api)!:`)
- **Scope Examples**: `feat(formkit):`, `fix(plugin):`, `docs(readme):`, `chore(deps):`
- **Examples**:
  - `feat: add UInput integration for FormKit text inputs`
  - `fix(plugin): resolve SSR hydration mismatch`
  - `docs: update installation instructions`
  - `chore(deps): upgrade @nuxt/ui to 4.3.0`
  - `feat!: change module configuration API`
- **Best Practices**:
  - Keep the subject line under 72 characters
  - Use imperative mood ("add" not "added" or "adds")
  - Don't capitalize the first letter of the description
  - No period at the end of the subject line
  - Provide detailed explanation in the body if needed
  - Reference issues/PRs in the footer (e.g., `Fixes #123`, `Closes #456`)

### Testing
- Write tests using Vitest
- Use `@nuxt/test-utils` for Nuxt-specific testing
- Place fixtures in `/test/fixtures/`
- Test files should end with `.test.ts`

## Module-Specific Guidelines

### When Adding Features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sfxcode/nuxt-ui-formkit](https://github.com/sfxcode/nuxt-ui-formkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
