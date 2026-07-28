---
trigger: always_on
description: **CRITICAL**: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Button Card - GitHub Copilot Development Instructions

**CRITICAL**: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

Button Card is a TypeScript-based custom Lovelace card for Home Assistant. It provides customizable button elements with advanced styling, state management, and interaction capabilities.

## Working Effectively

### Bootstrap and Build
- Install dependencies: `yarn install --frozen-lockfile` -- takes 10-60 seconds depending on cache. NEVER CANCEL. Set timeout to 120+ seconds.
- Run linting: `npm run lint` -- takes ~2 seconds
- Build the project: `npm run build` -- takes ~7 seconds (includes lint + rollup)
- Build only (skip lint): `npm run rollup` -- takes ~5 seconds
- NEVER CANCEL: All build commands must complete fully. Use timeout values of 120+ seconds for yarn install, 60+ seconds for builds.

### Development Workflow
- Start development server: `npm run watch` -- runs on `http://localhost:5000`
- The dev server serves the built card at `http://localhost:5000/button-card.js`
- Changes auto-rebuild and are served immediately
- Access the built card via the development server for testing

### Validation and Quality
- ALWAYS run `npm run lint` before committing changes or the CI (.github/workflows/ci.yml) will fail
- Format code: `npx prettier --check src/**/*.ts` (check) and `npx prettier --write src/**/*.ts` (fix)
- ALWAYS build and test your changes: `npm run build` followed by manual validation
- Security updates: `npm run audit-fix` - fixes known security vulnerabilities
- CRITICAL VALIDATION: After making changes, ALWAYS test the card functionality by:
  1. Building the project: `npm run build`
  2. Starting the dev server: `npm run watch`
  3. Verifying the card loads at `http://localhost:5000/button-card.js`
  4. Testing basic card functionality if possible

## Repository Structure

### Key Directories
- `src/` - TypeScript source code
  - `button-card.ts` - Main card component (LitElement)
  - `types/` - TypeScript type definitions
  - `common/` - Shared utilities and helpers
  - `helpers.ts` - Utility functions for state, color, and entity management
  - `styles.ts` - CSS styling definitions
  - `action-handler.ts` - Event handling for user interactions
- `dist/` - Built output (generated, do not edit)
- `test/` - Home Assistant test environment configuration
- `.github/` - CI/CD workflows and GitHub configuration
- `examples/` - Example configurations and images

### Important Files
- `package.json` - Dependencies and npm scripts
- `rollup.config.js` - Build configuration using Rollup bundler
- `tsconfig.json` - TypeScript compiler configuration
- `.eslintrc.js` - ESLint linting rules
- `.prettierrc.yaml` - Code formatting rules
- `test/configuration.yaml` - Home Assistant test configuration
- `test/ui-lovelace.yaml` - Lovelace dashboard test configuration

## Technology Stack

### Core Technologies
- **TypeScript** - Primary language
- **Lit** (LitElement) - Web components framework for the card UI
- **Rollup** - Module bundler and build tool
- **Home Assistant** - Target platform (custom Lovelace card)
- **Web Components** - Card is implemented as a custom element

### Development Tools
- **ESLint** - Code linting with TypeScript rules
- **Prettier** - Code formatting
- **Yarn** - Package management (lockfile: yarn.lock)
- **Node.js 22.x** - Development environment (as per CI configuration)

## Common Development Tasks

### Making Code Changes
1. Edit TypeScript files in `src/`
2. Run `npm run lint` to check code style
3. Run `npm run build` to build and test compilation
4. Start dev server with `npm run watch` to test changes
5. Validate card functionality manually

### Adding New Features
- Main card logic: Edit `src/button-card.ts`
- Type definitions: Add to `src/types/types.ts` or related files
- Styling: Update `src/styles.ts`
- Utilities: Add to `src/helpers.ts` or `src/common/`

### Debugging and Testing
- NO AUTOMATED TESTS: This project relies on manual testing and linting only
- Test environment: Use files in `test/` directory with Home Assistant
- The test environment includes mock entities and a sample Lovelace configuration
- Manual testing workflow: Build → Start dev server → Test in Home Assistant or browser

### Working with Home Assistant Integration
- The card integrates with Home Assistant's Lovelace dashboard system
- Entity management: Use functions in `src/helpers.ts`
- State handling: Core logic in `src/button-card.ts`
- Events and actions: Handled by `src/action-handler.ts`

## Build System Details

### Scripts Available
- `npm run build` - Full build (lint + rollup)
- `npm run rollup` - Bundle only (no linting)
- `npm run lint` - ESLint validation
- `npm run watch` - Development server with auto-rebuild
- `npm run audit-fix` - Fix security vulnerabilities

### Build Timing Expectations
- yarn install: 10-60 seconds depending on cache (NEVER CANCEL - use 120+ second timeout)
- npm run lint: ~2 seconds
- npm run build: ~7 seconds (NEVER CANCEL - use 60+ second timeout)
- npm run rollup: ~5 seconds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [custom-cards/button-card](https://github.com/custom-cards/button-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
