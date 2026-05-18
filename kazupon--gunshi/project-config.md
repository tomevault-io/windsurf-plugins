---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gunshi is a modern JavaScript command-line library for creating CLI applications. It supports multiple JavaScript runtimes (Node.js, Deno, Bun) and provides features like declarative command configuration, type safety, composable sub-commands, lazy loading, internationalization, and a powerful plugin system.

This is a **pnpm workspace monorepo** containing the core library, plugins, utilities, and documentation. The project emphasizes type safety, runtime compatibility, minimal dependencies, and extensibility through a well-designed plugin architecture.

## Essential Commands

```sh
# Install dependencies
pnpm install

# Build all packages
pnpm build

# Run all tests with type checking
pnpm test

# Run E2E tests
pnpm e2e

# Run tests for specific packages
pnpm test:core                  # Core gunshi package
pnpm test:plugin-global         # Global options plugin
pnpm test:plugin-i18n           # i18n plugin
pnpm test:plugin-renderer       # Renderer plugin
pnpm test:plugin-completion     # Completion plugin

# Run linter (ESLint, Prettier, Knip, JSR checks)
pnpm lint

# Auto-fix linting issues
pnpm fix

# Run type checking
pnpm lint:typecheck             # Run all type checks (typecheck:deno, typecheck:tsc)
pnpm typecheck:deno             # Deno-specific type checking
pnpm typecheck:tsc              # TypeScript compiler diagnostics

# Run benchmarks
pnpm bench:vitest
pnpm bench:mitata

# Build documentation
pnpm build:docs

# Develop documentation site
pnpm dev:docs

# Release new version (bumps versions, creates tag, pushes)
pnpm release
```

## Monorepo Structure

The repository contains the following packages:

### Core Packages

- **`packages/gunshi/`** - Main library with CLI runtime, command system, and built-in plugins
  - Exports: `cli`, `define`, `defineWithTypes`, `lazy`, `lazyWithTypes`, `plugin`, `createCommandContext`
  - Published to: npm, JSR

- **`packages/bone/`** - Minimal gunshi distribution without plugins (`@gunshi/bone`)
  - Smaller bundle size for lightweight CLIs
  - Published to: npm, JSR

- **`packages/definition/`** - Command definition utilities (`@gunshi/definition`)
  - Provides `define`, `defineWithTypes`, `lazy`, `lazyWithTypes` helpers
  - Published to: npm, JSR

- **`packages/plugin/`** - Plugin development kit (`@gunshi/plugin`)
  - Type definitions and APIs for creating plugins
  - Same exports as `gunshi/plugin` but with smaller footprint
  - Published to: npm, JSR

### Support Packages

- **`packages/shared/`** - Shared utilities (`@gunshi/shared`)
  - Common utilities used across packages
  - Localization helpers, key resolution, type utilities

- **`packages/resources/`** - Built-in localization resources (`@gunshi/resources`)
  - Default translations for built-in messages
  - Supports multiple locales

### Official Plugins

- **`packages/plugin-global/`** - Global options plugin (`@gunshi/plugin-global`)
  - Provides `--version`, `--help` options
  - Header/usage rendering utilities

- **`packages/plugin-i18n/`** - Internationalization plugin (`@gunshi/plugin-i18n`)
  - Locale management and resource loading
  - Translation adapters for multiple i18n libraries
  - Helpers: `defineI18n`, `defineI18nWithTypes`, `withI18nResource`

- **`packages/plugin-renderer/`** - Usage/help renderer plugin (`@gunshi/plugin-renderer`)
  - Customizable help text and validation error rendering
  - Works with i18n plugin for localized output

- **`packages/plugin-completion/`** - Shell completion plugin (`@gunshi/plugin-completion`)
  - Tab completion for commands and options
  - Supports bash, zsh, fish

- **`packages/plugin-dryrun/`** - Dry-run mode plugin (`@gunshi/plugin-dryrun`)
  - Adds `--dry-run` option to commands

### Documentation

- **`packages/docs/`** - Documentation site (VitePress)
  - Available at: [gunshi.dev](https://gunshi.dev)
  - Guides, API references, examples

## Architecture Overview

### Core Library (`packages/gunshi/src/`)

```sh
packages/gunshi/src/
├── index.ts              # Main entry point, exports public APIs
├── bone.ts               # Minimal CLI entry (no plugins)
├── cli.ts                # CLI entry with built-in plugins
├── cli/
│   ├── core.ts           # Core CLI execution logic
│   ├── builtin.ts        # Built-in plugin integration
│   └── bone.ts           # Bone CLI logic
├── context.ts            # Command context creation and management
├── definition.ts         # Command definition helpers
├── decorators.ts         # Command runner decorators
├── generator.ts          # Command generator utilities
├── renderer.ts           # Usage/help rendering
├── utils.ts              # Runtime utilities (Node.js, Deno, Bun)
├── types.ts              # TypeScript type definitions
├── constants.ts          # Constants and defaults
└── plugin/
    ├── core.ts           # Plugin system core
    ├── context.ts        # Plugin context management
    └── dependency.ts     # Plugin dependency resolution

# Test files are co-located with source files
├── *.test.ts             # Unit tests (Vitest)
├── *.test-d.ts           # Type tests (Vitest + TypeScript)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kazupon/gunshi](https://github.com/kazupon/gunshi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
