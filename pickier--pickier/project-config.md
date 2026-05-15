---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Overview

Pickier is a fast linter and formatter built with Bun, designed to provide instant feedback with minimal configuration. It combines linting, formatting, import organization, and markdown linting in a single tool with an ESLint-style plugin system.

## Commands

### Development

```bash

# Install dependencies

bun i

# Run tests (with coverage)

bun test

# Run specific test suites

bun test:core # Core functionality tests
bun test:format # Formatting tests
bun test:lint # Linting tests
bun test:rules # All rule tests
bun test:plugin # Plugin system tests
bun test:watch # Watch mode

# Build the package

bun run -C packages/pickier build

# Build all packages

bun run build

# Compile native binaries

bun run -C packages/pickier compile # Current platform
bun run -C packages/pickier compile:all # All platforms

# Type checking

bun --bun tsc --noEmit
```

### Testing Locally

```bash
# Run the TypeScript CLI directly (fastest for development)

bun packages/pickier/bin/cli.ts run . --mode lint

# Run the built JavaScript CLI

bun packages/pickier/dist/bin/cli.js run . --mode lint

# Run the compiled native binary (after compiling)

./packages/pickier/bin/pickier-<platform> run . --mode lint
```

### Using Pickier

```bash

# Unified command (preferred)

pickier run . --mode lint --fix
pickier run . --mode format --write

# Shorthand commands

pickier lint . --fix
pickier format . --write
```

## Architecture

### Core Components

1.**Unified Entry Point (`src/run.ts`)**- Single entry point that routes to either lint or format mode

- `runUnified()` function handles mode detection and delegates to linter
- Formatting is now implemented as "linting with fixes applied"

2.**Linter (`src/linter.ts`)**- `runLint()`: Main CLI linting workflow with file globbing, scanning, fixing, and reporting

- `runLintProgrammatic()`: Programmatic API that returns structured results
- `lintText()`: Lint a single string with optional cancellation support
- `scanContent()`: Core scanning logic for built-in rules (quotes, indent, debugger, console, etc.)
- `applyPlugins()`: Executes plugin rules with timeout protection and error handling
- `applyPluginFixes()`: Iteratively applies rule fixers from plugins
- `parseDisableDirectives()`: Parses ESLint-style disable comments (disable-next-line, disable/enable blocks)
- `isSuppressed()`: Checks if a rule is suppressed for a given line

3.**Formatter (`src/formatter.ts`)**- Legacy module, mostly deprecated in favor of unified linting

- `applyFixes()`: Applies built-in fixes (debugger removal) + plugin fixes + global formatting
- `formatStylish()`: Formats lint issues in ESLint-style output

4.**Format Engine (`src/format.ts`)**- `formatCode()`: Core formatting logic for whitespace, quotes, indentation, semicolons, and imports

- Import organization: splits type/value imports, sorts modules/specifiers, removes unused imports
- Whitespace: trim trailing, limit consecutive blank lines, ensure final newline(s)
- Semicolon removal: safely removes stylistic semicolons while preserving for-loop headers
- Quote normalization: enforces single/double quotes in code (respects JSON double-quote requirement)
- Shell formatting: `processShellLinesFused()` normalizes indentation for shell control structures (`if`/`then`/`fi`, `case`/`esac`, `while`/`for`/`do`/`done`, function bodies), preserving heredoc content verbatim

5.**Plugin System (`src/plugins/`)**-**Core Plugins**: `pickier`, `style`, `regexp`, `ts`, `markdown`, `shell`, `publint`- Each plugin exports a`PickierPlugin`with`name`and`rules`Record

- Rules implement`RuleModule`interface:`check(content, context) => LintIssue[]`and optional`fix(content, context) => string`- Configuration via`pluginRules` in config, supporting both full IDs (`plugin/rule`) and bare rule names
- Rules can be marked as WIP (`meta.wip = true`) to surface implementation errors early
- Plugins are loaded via `getAllPlugins()` which returns all core plugins

6.**Configuration (`src/config.ts`)**- Uses `bunfig`to load`pickier.config.ts`from project root

- Exports`defaultConfig`and loaded`config`- Config includes`ignores`, `lint`, `format`, `rules`, `plugins`, and `pluginRules`

7.**AST Utilities (`src/ast.ts`)**- Lightweight TypeScript parsing utilities for plugin rules

- Used by import/sort rules to understand code structure

### Testing

Tests are organized by functionality:

- `test/core/`: Core functionality tests
- `test/format/`: Formatting behavior tests
- `test/lint/`: Linting workflow tests
- `test/rules/`: Rule-specific tests (sort, style, markdown, shell, imports, typescript, regexp)
- `test/plugin/`: Plugin system tests
- `test/fixtures/`: Sample files for testing

All tests use Bun's test runner. Set `PICKIER_NO_AUTO_CONFIG=1`to disable auto-loading config during tests.

### Environment Variables

-`PICKIER_NO_AUTO_CONFIG=1`: Disable automatic config loading (used in tests)

- `PICKIER_TRACE=1`: Enable verbose trace logging
- `PICKIER_TIMEOUT_MS`: Glob timeout in milliseconds (default: 8000)
- `PICKIER_RULE_TIMEOUT_MS`: Individual rule timeout in milliseconds (default: 5000)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pickier/pickier](https://github.com/pickier/pickier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
