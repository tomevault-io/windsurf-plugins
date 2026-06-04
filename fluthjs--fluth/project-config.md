---
trigger: always_on
description: This document describes the folder structure and component functionality of the Fluth project.
---

# Fluth Project Structure Rules

This document describes the folder structure and component functionality of the Fluth project.

## Project Overview

Fluth is a Promise-like asynchronous flow control library that provides powerful stream programming capabilities.

## Directory Structure

### 📁 Root Directory Files

- **`index.ts`** - Main entry file that exports all public APIs
- **`package.json`** - Project configuration file containing dependencies, scripts, and metadata
- **`README.md`** - Project documentation
- **`LICENSE`** - MIT open source license file
- **`CHANGELOG.md`** - Version update changelog
- **`pnpm-lock.yaml`** - pnpm package manager lock file

### 📁 Source Code Directory (`src/`)

Core business logic code containing the following modules:

- **`observable.ts`** - Observable class implementation providing core observable object functionality
- **`stream.ts`** - Stream class implementation extending Observable with streaming operations
- **`factory.ts`** - Factory functions for creating Stream instances
- **`operator.ts`** - Operator implementations providing various stream operations (combine, merge, race, etc.)
- **`plugins.ts`** - Plugin system providing delay, throttle, debounce and other functional plugins
- **`utils.ts`** - Utility function collection

### 📁 Test Directory (`test/`)

Contains all test files corresponding to the `src/` directory structure:

- **`factory.test.ts`** - Factory function tests
- **`observer.test.ts`** - Observable related tests
- **`operator.test.ts`** - Operator functionality tests
- **`plugins.test.ts`** - Plugin system tests
- **`stream.test.ts`** - Stream class tests
- **`utils.ts`** - Test utility functions

### 📁 Configuration Directories

#### `.github/workflows/`

GitHub Actions workflow configurations:

- **`check.yml`** - Code checking workflow
- **`release-publish.yml`** - Release workflow

#### `.husky/`

Git hooks configuration:

- **`commit-msg`** - Commit message format checking
- **`pre-commit`** - Pre-commit code checking

### 📁 Configuration Files

#### TypeScript Configuration

- **`tsconfig.json`** - Base TypeScript configuration
- **`tsconfig.cjs.json`** - CommonJS module build configuration
- **`tsconfig.mjs.json`** - ES module build configuration

#### Code Quality Tools

- **`eslint.config.mjs`** - ESLint code checking configuration
- **`.lintstagedrc.cjs`** - lint-staged configuration for pre-commit checking
- **`.prettierrc.json`** - Prettier code formatting configuration
- **`.prettierignore`** - Prettier ignore file configuration
- **`.commitlintrc.mjs`** - Commit message convention configuration

#### Test Configuration

- **`vitest.config.js`** - Vitest testing framework configuration

#### Other Configuration

- **`.gitignore`** - Git ignore file configuration

## Development Standards

### Language Requirements

- **All code must be written in English** - Variable names, function names, class names, and all identifiers
- **All comments must be in English** - Including JSDoc comments, inline comments, and documentation
- **All commit messages must be in English** - Following conventional commit format
- **All documentation must be in English** - README, API docs, and inline documentation

### File Naming Conventions

- Source files use lowercase letters and hyphens
- Test files end with `.test.ts`
- Configuration files named according to tool requirements

### Code Organization Standards

- Core functionality code in `src/` directory
- Each module has corresponding test files
- Public APIs exported uniformly through `index.ts`
- Plugins and operators organized in separate files

### Operator Design Rules

#### Stream Lifecycle Management

- When designing operators with stream inputs and outputs, consider the impact of input streams that are already finished on the output stream
- When designing operators with stream inputs and outputs, consider the impact of input stream unsubscription on the output stream
- When designing operators with stream inputs and outputs, consider the impact of output stream unsubscription on the input stream
- When designing operators with stream inputs and outputs, consider the impact of input empty
- Properly handle stream completion and unsubscription propagation in both directions

#### Memory Management

- When designing operators with stream inputs and outputs, consider memory leak issues
- Implement proper cleanup mechanisms for all subscriptions and references
- Use `useUnsubscribeCallback` utility for managing multiple stream unsubscriptions
- Clear arrays and references in cleanup callbacks to prevent memory leaks
- Handle circular references and parent-child relationships properly

#### Input Validation and Edge Cases

- Thoroughly consider input boundary conditions when designing operators
- Validate input types and throw appropriate errors for invalid inputs
- Handle empty input scenarios gracefully
- Consider single stream vs multiple stream scenarios
- Handle mixed valid/invalid input combinations

#### Status and State Management

- Properly track and propagate PromiseStatus (PENDING, RESOLVED, REJECTED)
- Handle already finished streams correctly during operator initialization
- Ensure consistent status updates across all stream operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluthjs/fluth](https://github.com/fluthjs/fluth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
