---
trigger: always_on
description: - **Type**: Monorepo-like structure (manual orchestration, no workspaces)
---

# AGENTS.md - Codebase Guide for AI Agents

## Project Overview
- **Type**: Monorepo-like structure (manual orchestration, no workspaces)
- **Language**: JavaScript (ES Modules with .js extensions)
- **Node.js**: v22.20.0+ required (see .nvmrc)
- **Type Safety**: Zod schemas for runtime validation (no TypeScript)

## Build, Lint & Test Commands

### Development Commands
```bash
npm run dev                # Start all services (backend + admin + desktop) - RECOMMENDED
npm run dev:backend        # Backend server with --watch (port 5621)
npm run dev:admin          # Admin UI webpack dev server (port 9000)
npm run dev:desktop        # Electron desktop app
```

### Build Commands
```bash
npm run build              # Build all (admin-ui + core)
npm run build:core         # Build server package (esbuild)
npm run build:admin-ui      # Build admin UI (webpack → packages/web/)
npm run build:desktop       # Build desktop app (current platform)
npm run build:desktop:all   # Build desktop for all platforms (mac/win/linux)
npm run build:icons        # Build app icons
```

### Linting & Formatting
```bash
npm run lint              # Run ESLint with auto-fix
npm run lint:fix          # Run ESLint with auto-fix
npm run lint:check        # Check ESLint without fixing
npm run format            # Run Prettier to format code
npm run format:fix        # Run Prettier with auto-fix
npm run format:check      # Check formatting without changing files
```

### Test Commands
```bash
npm test                  # Run ALL tests (verification + server + integration)
npm run test:server       # Server unit tests only
npm run test:server:watch       # Watch mode for unit tests
npm run test:server:coverage    # Generate coverage report (70% threshold)
npm run test:server:integration  # Server integration tests
npm run test:e2e          # E2E tests for packaged app
npm run test:module-loading # Module loading tests
```

### Verification Commands
```bash
npm run verify             # Run comprehensive publish verification (recommended before commits)
npm run verify:publish     # Verify npm package publish readiness
npm run check:deps        # Check and install all dependencies
npm run check:env         # Check development environment
```

### System Commands
```bash
npm run fix:pty           # Rebuild node-pty native module
npm run help              # Show CLI help information
npm run setup:env         # Run pre-install environment check
```

### Single Test Execution
```bash
# Run specific test file
cd packages/server && vitest tests/unit/core.test.js

# Run tests matching pattern
cd packages/server && vitest tests/unit/terminal*

# Run single test suite by name
cd packages/server && vitest -t "WebSocketService"
```

### NPM Publish Verification
```bash
# Verify npm package publish readiness (recommended before creating tag)
npm run verify:publish
```
This command checks:
- ESLint and Prettier compliance
- All tests pass
- All files listed in package.json exist
- Version consistency across all files
- Publish readiness

### Verification (Recommended before commits/PRs)
```bash
# Run comprehensive verification
npm run verify
```
This command performs comprehensive verification:
- **Code Quality**: ESLint, Prettier, all tests
- **NPM Package**: File existence, version consistency, publish readiness

**Use this before**:
- Creating pull requests
- Pushing to main branch
- Creating version tags for release

### Dependency Management
```bash
# Check and install all missing dependencies
npm run check:deps

# Check development environment
npm run check:env

# Clean environment cache and dependencies (recommended when encountering build issues)
npm run clean           # Full environment cleanup
npm run clean:cache     # Clean only caches
npm run clean:deps      # Clean and reinstall dependencies
npm run clean:build     # Clean build artifacts only
```

## Code Style Guidelines

### Imports
- **Named imports**: `import { logger } from '../utils/logger.js'`
- **Default imports**: `import fs from 'fs-extra'`
- **Explicit .js extensions**: Required for ES modules
- **Order**: Node built-ins → third-party → local modules
- **Relative paths**: Use `../` and `./` for local imports

### Formatting (Prettier + ESLint Integration)

**Integrated Configuration:**
- ESLint and Prettier are integrated via `eslint-config-prettier` and `eslint-plugin-prettier`
- `npm run lint` checks both code quality (ESLint) and formatting (Prettier)
- `npm run lint:fix` automatically fixes both issues in one command
- Formatting rules are managed by Prettier and reported as ESLint errors

**Prettier Rules:**
- **Indentation**: 2 spaces (no tabs)
- **Quotes**: Single quotes for strings
- **Semicolons**: Required
- **Max line length**: 120 characters
- **Trailing commas**: None
- **Arrow parens**: Avoid for single parameter: `arg => expr`
- **Line endings**: LF only

**Important:** Do NOT configure formatting rules (quotes, semi, indent, etc.) in ESLint - they are managed by Prettier via `plugin:prettier/recommended`.

### Type Definitions (Zod)
```javascript
import { z } from 'zod';

const SchemaName = z.object({
  field: z.string().min(1, 'Error message'),
  optionalField: z.string().optional(),
  enumField: z.enum(['value1', 'value2']).default('value1'),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BeCrafter/prompt-manager](https://github.com/BeCrafter/prompt-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
