---
trigger: always_on
description: This document provides a comprehensive guide for AI assistants working with the `vue-translations-cleanup` codebase. It covers the project structure, development workflows, coding conventions, and key patterns.
---

# CLAUDE.md

This document provides a comprehensive guide for AI assistants working with the `vue-translations-cleanup` codebase. It covers the project structure, development workflows, coding conventions, and key patterns.

## Project Overview

**vue-translations-cleanup** is a CLI tool designed to find and remove unused translation keys in Vue.js i18n projects. It primarily targets the official vue-i18n (Intlify) library but may work with compatible i18n libraries.

### Key Features
- Auto-detection of source and translation paths (Vite + @intlify/unplugin-vue-i18n)
- Advanced translation detection (t(), $t(), rt(), $rt(), tc(), $tc(), Composition API)
- Support for Vue template directives (v-t) and components (<i18n-t>)
- Safe updates with automatic backups
- Dry-run mode for previewing changes
- Automatic pruning of empty objects after deletions
- Both single-file and directory-wide cleanup support

### Package Information
- **Name**: vue-translations-cleanup
- **Current Version**: 1.4.0
- **License**: MIT
- **Author**: Mindaugas Kristutis
- **Main Entry**: dist/index.js
- **CLI Binary**: dist/cli.js

## Repository Structure

```
vue-translations-cleanup/
├── src/                              # Source code
│   ├── translations-cleanup/         # Core translation cleanup logic
│   │   ├── index.ts                  # Main cleanup function
│   │   ├── fileScanner.ts            # File scanning and pattern matching
│   │   ├── translationUtils.ts       # Translation key utilities
│   │   ├── patterns.ts               # Regex patterns for detecting translation usage
│   │   └── types.ts                  # TypeScript type definitions
│   ├── cli.ts                        # CLI entry point and command handling
│   ├── cli-detection.ts              # Auto-detection logic for paths
│   └── cli-style.ts                  # CLI styling utilities (colors, symbols)
├── tests/                            # Test files
│   ├── translations-cleanup/         # Core functionality tests
│   │   ├── edgeCases.test.ts         # Edge case testing
│   │   ├── pruning.test.ts           # Empty object pruning tests
│   │   ├── nestedTranslations.test.ts # Nested key handling
│   │   ├── nestedParams.test.ts      # Nested parameter tests
│   │   ├── vueTemplateDirectives.test.ts # Vue template usage tests
│   │   ├── validation.test.ts        # Input validation tests
│   │   └── translationPatterns.test.ts # Pattern detection tests
│   ├── cli.test.ts                   # CLI interface tests
│   ├── cli-directory-mode.test.ts    # Directory mode tests
│   ├── cli-autodetect-usage.test.ts  # Auto-detection tests
│   └── setup.ts                      # Test setup configuration
├── .github/workflows/                # GitHub Actions workflows
│   ├── test.yml                      # PR testing workflow
│   ├── release.yml                   # Release automation workflow
│   └── auto-merge-release-pr.yml     # Auto-merge for release PRs
├── dist/                             # Compiled output (git-ignored)
├── package.json                      # Package configuration
├── tsconfig.json                     # TypeScript configuration
├── eslint.config.js                  # ESLint configuration
├── vitest.config.mts                 # Vitest configuration
├── README.md                         # User documentation
├── CHANGELOG.md                      # Version history
└── LICENSE                           # MIT License
```

## Technology Stack

### Core Dependencies
- **commander** (^14.0.0) - CLI argument parsing
- **glob** (^11.0.3) - File pattern matching
- **kleur** (^4.1.5) - Terminal color styling

### Development Dependencies
- **TypeScript** (^5.9.2) - Type-safe development
- **Vitest** (^3.2.4) - Unit testing framework
- **ESLint** (^9.35.0) - Code linting
- **@antfu/eslint-config** (^5.2.2) - Opinionated ESLint config
- **conventional-changelog-cli** (^4.1.0) - Changelog generation
- **commit-and-tag-version** (^12.4.0) - Version management

### Package Manager
The project uses **yarn** for dependency management (as evidenced by yarn.lock and CI workflows).

## Development Workflows

### Build Process
```bash
# Build TypeScript to JavaScript
yarn build   # or pnpm run build

# Output directory: dist/
# Entry points: dist/index.js (main), dist/cli.js (binary)
```

**TypeScript Configuration**:
- Target: ES2020
- Module: CommonJS
- Strict mode enabled
- Declaration files generated
- Path alias: `@/*` -> `src/*`

### Testing

**Test Framework**: Vitest

```bash
# Run all tests
yarn test

# Tests are located in tests/ directory
# Setup file: tests/setup.ts
# Pattern: tests/**/*.test.{ts,js}
```

**Test Coverage Areas**:
- Core translation cleanup logic
- Pattern matching and detection
- Nested key handling
- Empty object pruning
- CLI interface and options
- Auto-detection functionality
- Directory mode processing
- Vue template directives and components

### Linting

**Linter**: ESLint with @antfu/eslint-config

```bash
# Lint source files
yarn lint

# Lints: src/**/*.ts
```

**ESLint Configuration**:
- Based on @antfu/eslint-config
- Vue support disabled (vue: false)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [butaminas/vue-translations-cleanup](https://github.com/butaminas/vue-translations-cleanup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
