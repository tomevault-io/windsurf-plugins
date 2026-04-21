---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`jsont` is a Terminal User Interface (TUI) JSON viewer built with React and Ink. The application reads JSON data from stdin or files and displays it in various interactive formats including tree view, collapsible view, and schema view with advanced query capabilities using jq and JSONata.

## Key Commands

### Development
- `npm run dev` - Run the application in development mode using tsx
- `npm run build` - Bundle the application using tsup (supports extensionless imports)
- `npm run start` - Run the compiled application from `dist/index.js`

### Testing
- `npm run test` - Run tests in watch mode
- `npm run test:run` - Run tests once and exit
- `npm run test:watch` - Run tests in explicit watch mode
- `npm run test:ui` - Open Vitest UI for interactive testing
- `npm run test:ci` - Run CI-optimized tests (memory-limited, single-threaded)
- `npm run test:coverage` - Run tests with coverage reporting (HTML, JSON, text)
- `npm run test:coverage:ui` - Open Vitest UI with coverage for interactive testing and viewing
- `npm run test -- tree` - Run specific test files matching "tree"
- `npm run test -- --coverage` - Run tests with coverage reporting (alternative syntax)

**Memory-Intensive Test Exclusions**: Several tests are excluded in CI for memory optimization:
- Performance tests, JSON processing tests, SQL converter tests, integration tests
- See `vitest.config.ts` exclude list for complete details

**Coverage Reports**: Coverage reports are generated in `./coverage/` directory:
- **Text Report**: Displayed in terminal during test execution
- **HTML Report**: Interactive web interface at `./coverage/index.html`
  - Browse file-by-file coverage with line-by-line highlighting
  - View uncovered lines, branch coverage, and function coverage
  - Color-coded visualization: red (uncovered), green (covered), orange (partial)
- **JSON Report**: Machine-readable format at `./coverage/coverage-final.json`
- Coverage metrics include: Statements, Branches, Functions, Lines

### Code Quality
- `npm run check` - Run Biome linter and formatter checks
- `npm run check:write` - Run Biome checks and apply safe fixes
- `npm run type-check` - Run TypeScript type checking only
- `npm run lint` - Lint source code only
- `npm run lint:fix` - Lint and automatically fix issues
- `npm run format` - Format source code (read-only)
- `npm run format:write` - Format and write source code

### Usage
```bash
echo '{"key": "value"}' | npm run dev
cat file.json | npm run dev
npm run dev path/to/file.json
```

## Architecture Overview

### Clean Architecture Pattern

The codebase follows a feature-driven clean architecture with clear separation of concerns:

#### Entry Point (`src/index.tsx`)
- Minimal entry point that delegates to AppService
- Uses proper error handling with `handleFatalError`

#### Core Layer (`src/core/`)
- **Services**: `AppService` orchestrates application lifecycle and initialization
- **Utils**: Comprehensive utility libraries
  - Terminal management (`terminal.ts`, `processManager.ts`)
  - Advanced stdin processing (`stdinHandler.ts`) 
  - Error handling with recovery suggestions (`errorHandler.ts`)
  - LRU caching system (`lruCache.ts`)
  - CLI argument parsing (`cliParser.ts`)
  - Data conversion system (`dataConverters/`) with SQL, XML, YAML, CSV, JSON Schema support
- **Types**: Application-wide type definitions and interfaces
- **Config**: YAML configuration system with schema validation, hot reloading, and defaults
- **Context**: React context providers for configuration management

#### Features Layer (`src/features/`)
Organized by feature domains, each containing:
- **Components**: React/Ink UI components
- **Types**: Feature-specific type definitions  
- **Utils**: Business logic and utilities
- **Tests**: Co-located test files

**Key Features:**
- `tree/` - Interactive tree view with keyboard navigation
- `collapsible/` - Collapsible JSON viewer with syntax highlighting
- `search/` - Search functionality across JSON data
- `jq/` - jq query transformation support
- `schema/` - JSON schema inference and multi-format export (JSON, YAML, CSV, XML, SQL)
- `json-rendering/` - Core JSON parsing and syntax highlighting
- `common/` - Shared components (TextInput, BaseViewer, hooks)
- `debug/` - Debug logging and viewer components
- `navigation/` - Goto navigation (gg/G) and keyboard shortcuts
- `help/` - Context-sensitive help system
- `status/` - Status bar and line number display
- `settings/` - Interactive settings TUI with live preview and validation

#### Component Architecture (`src/components/`)
- **Providers**: Application-wide context and state providers (`AppStateProvider`)
- **Content**: View routing and content management (`ContentRouter`)
- **Modals**: Modal management system (`ModalManager`)
- **Status**: Status bar and line number management (`StatusBarManager`) 
- **Keyboard**: Input handling coordination (`KeyboardManager`)

#### Hooks Architecture (`src/hooks/`)
- **State Hooks**: Application state management (`useAppState`, `useLayoutCalculations`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SuzumiyaAoba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
