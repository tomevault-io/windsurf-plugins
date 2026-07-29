---
trigger: always_on
description: **Glimdown** is an interactive browser-based REPL and tutorial platform for writing Ember/Glimmer applications using markdown syntax. This monorepo hosts:
---

# Copilot Instructions for Limber/Glimdown

## Project Overview

**Glimdown** is an interactive browser-based REPL and tutorial platform for writing Ember/Glimmer applications using markdown syntax. This monorepo hosts:
- **Limber REPL** (https://limber.glimdown.com) - Interactive code editor for Glimmer components
- **Glimmer Tutorial** (https://tutorial.glimdown.com) - Interactive learning platform

The project enables users to write, test, and share Ember/Glimmer code directly in their browser without any setup.

## Tech Stack

- **Framework**: Ember.js v6+ (Octane edition), Glimmer components
- **Build System**: Vite (beta), Embroider, Turbo (monorepo orchestration)
- **Package Manager**: pnpm v10.25.0+ (required)
- **Node Version**: v24.12.0+ (enforced via Volta)
- **Languages**: TypeScript 5.9.3, JavaScript (ES2022+)
- **Testing**: Vitest (node), Testem (browser), QUnit
- **Linting**: ESLint (flat config), Prettier, ember-template-lint
- **Styling**: TailwindCSS
- **Syntax Parsing**: CodeMirror v6, Lezer parsers

## Repository Structure

```
limber/
├── apps/                   # Production applications
│   ├── repl/              # Main REPL application (Vite-based Ember)
│   ├── tutorial/          # Interactive tutorial application
│   └── home/              # Landing page
├── packages/              # Shared libraries and addons
│   ├── ember-repl/        # Core REPL addon
│   ├── repl-sdk/          # SDK for REPL functionality
│   ├── limber-ui/         # UI component library
│   ├── app-support/       # Styles and shared utilities
│   └── syntax/            # CodeMirror & Lezer syntax definitions
│       ├── glimmer/       # Base Glimmer syntax
│       ├── glimmer-ts/    # TypeScript variant
│       ├── glimdown/      # Markdown/Glimdown syntax
│       └── glimmer-js/    # JavaScript variant
├── dev/                   # Development tools (linting CLI)
├── patches/               # pnpm patch files for dependencies
└── .github/workflows/     # CI/CD pipelines
```

## Build & Development Commands

### Essential Commands

**Always run `pnpm i` after pulling changes or switching branches** to ensure dependencies are up to date.

```bash
pnpm i                     # Install all dependencies (required first step)
pnpm build                 # Build all apps and packages (uses Turbo caching)
pnpm start                 # Start all dev servers (REPL + Tutorial + styles)
pnpm lint                  # Run all linters (JS, HBS, Prettier, types)
pnpm lint:fix              # Auto-fix lint issues where possible
pnpm test                  # Run all tests (node + browser)
```

### App-Specific Commands

```bash
pnpm --filter=limber start        # Start REPL dev server (port 4201)
pnpm --filter=tutorial start      # Start tutorial dev server (port 4200)
pnpm --filter=limber-styles start # Watch and compile TailwindCSS
```

### Testing Commands

```bash
pnpm --filter=<package> test:node       # Run Vitest tests for package
pnpm --filter=<package> test:chrome     # Run browser tests in Chrome
pnpm --filter=<package> test:firefox    # Run browser tests in Firefox
```

### Build Order & Dependencies

1. Always build packages before apps (Turbo handles this automatically)
2. Syntax packages must be built before ember-repl
3. ember-repl and limber-ui must be built before the apps

## Linting & Code Quality

### Lint Checks (all must pass in CI)

```bash
pnpm lint         # Runs all lint tasks below
pnpm lint:fix     # Auto-fixes issues where possible
```

Individual lint tasks (via Turbo):
- `lint:js` - ESLint with flat config
- `lint:hbs` - Ember template linting
- `lint:prettier` - Code formatting (100 char line width, single quotes)
- `lint:types` - TypeScript type checking with Glint
- `lint:package` - Package.json validation
- `lint:published-types` - Validates published TypeScript declarations

### Code Style Guidelines

- **Line length**: 100 characters (Prettier enforced)
- **Quotes**: Single quotes preferred
- **Trailing commas**: Always use (es5 mode)
- **Templates**: Use `<template>` tag format for Glimmer components (`.gjs`/`.gts`)
- **TypeScript**: Prefer explicit types for public APIs
- **Imports**: Use workspace dependencies (`workspace:*`) for internal packages

## CI/CD Pipeline (.github/workflows/ci.yml)

The CI pipeline runs on every PR and push to main:

1. **install_dependencies** - Installs with `pnpm i` and builds with `turbo build --force`
2. **build_tests** - Builds test artifacts
3. **build_prod** - Builds production apps, uploads artifacts
4. **lints** - Runs full lint suite (must pass)
5. **tests** - Matrix tests across Node, Chrome, Firefox
6. **DeployProduction** (main only) - Deploys to Cloudflare Pages

### Important CI Notes

- Turbo uses `--force` flag in CI to ensure fresh builds
- All lint checks must pass before merge
- Test failures block deployment
- Production deploys only happen on main branch
- Build artifacts are cached between jobs

## Common Development Tasks

### Making Changes to Packages

1. Make code changes in `packages/<package-name>/`
2. Run `pnpm --filter=<package-name> build` to rebuild
3. Test changes in dependent apps
4. Run linting: `pnpm lint` before committing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NullVoxPopuli/limber](https://github.com/NullVoxPopuli/limber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
