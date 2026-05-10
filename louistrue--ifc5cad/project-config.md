---
trigger: always_on
description: This document provides coding guidelines and build instructions for agentic coding agents working in the Chili3D codebase.
---

# Chili3D Agent Guidelines

This document provides coding guidelines and build instructions for agentic coding agents working in the Chili3D codebase.

## Project Overview

Chili3D is a 3D CAD/WebGL application framework with WebAssembly integration. It's a TypeScript monorepo with 12 packages and a C++ WebAssembly module built with OpenCascade.

- **Tech Stack**: TypeScript, Rspack, Biome, Rstest, C++/Emscripten, Three.js
- **Architecture**: Modular monorepo with package-based organization
- **Target**: Modern web browsers with WebGL support

## Build Commands

```bash
# Development
npm run dev          # Start development server with Rspack
npm run build        # Production build
npm run preview      # Preview production build

# WebAssembly (C++)
npm run build:wasm   # Build C++ WASM module with CMake
npm run setup:wasm   # Setup WASM dependencies

# Code Quality
npm run check        # Biome linting and auto-fix (run this before commits!)
npm run format       # Format all code (Biome + clang-format)

# Testing
npm run test         # Run all tests with Rstest
npm run testc        # Run tests with coverage
npx rstest xyz.test.ts   # Run single test file (pattern matching supported)
npx rstest -t "should do"      # Run tests matching pattern
npx rstest packages/chili-core/test/  # Run all tests in directory
```

## Code Style Guidelines

### TypeScript/JavaScript

**Formatting (Biome)**:
- Indentation: 4 spaces
- Line width: 110 characters
- Quotes: Double quotes
- Semicolons: Required
- CSS Modules: Enabled

**Import Conventions**:
```typescript
// Package imports (use package names, not relative paths for internal packages)
import { AppBuilder } from "chili-builder";
import { type IApplication, Logger } from "chili-core";

// Type imports prefixed with 'type'
import type { ICommand } from "./command";
import type { IDataExchange } from "./dataExchange";

// Standard library and external packages
import { EventEmitter } from "events";
import * as THREE from "three";
```

**File Headers**: Every source file must include:
```typescript
// Part of the Chili3d Project, under the AGPL-3.0 License.
// See LICENSE file in the project root for full license information.
```

**Naming Conventions**:
- Interfaces: `IName` (e.g., `IApplication`, `IDocument`)
- Classes: `PascalCase` (e.g., `Application`, `Document`)
- Functions/Variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Files: `kebab-case.ts` or `kebab-case.test.ts`

**Error Handling**:
- Use `Result<T>` pattern for operations that can fail
- Prefer async/await over promises
- Always handle errors in async functions
- Use proper TypeScript types, avoid `any` (Biome will warn)

### C++ (WebAssembly)

**Style**: WebKit style with clang-format
- Formatting: `clang-format --style=Webkit --sort-includes`
- Standard: C++17
- License header: LGPL-3.0 (different from TypeScript code)

**File Headers**:
```cpp
// Part of the Chili3d Project, under the LGPL-3.0 License.
// See LICENSE-chili-wasm.text file in the project root for full license information.
```

## Testing Guidelines

**Framework**: Rstest with Happy-DOM
**Test Environment**: DOM simulation for web components

**Test Structure**:
```typescript
// Part of the Chili3d Project, under the AGPL-3.0 License.
// See LICENSE file in the project root for full license information.

import { describe, test, expect } from "@rstest/core";
import { TestDocument } from "../src/test-utils";

describe("FeatureName", () => {
    test("should do something", () => {
        // Arrange
        const doc = new TestDocument();
        
        // Act
        const result = someFunction();
        
        // Assert
        expect(result).toEqual(expectedValue);
    });
});
```

**Test Patterns**:
- Use `describe` blocks for logical grouping
- Follow Arrange-Act-Assert pattern
- Mock DOM APIs with Happy-DOM
- Use `TestDocument` for document testing utilities
- Test both positive and negative cases

## Package Architecture

**Core Packages**:
- `chili-core`: Core interfaces and utilities
- `chili-web`: Web application entry point
- `chili-builder`: Application builder pattern
- `chili-three`: Three.js 3D rendering integration
- `chili-ui`: UI components
- `chili-vis`: Visualization layer
- `chili-geo`: Geometry operations
- `chili-controls`: User controls and interactions
- `chili-storage`: Data persistence
- `chili-i18n`: Internationalization
- `chili-wasm`: WebAssembly bindings

**Module Resolution**:
- Use package names for internal imports, not relative paths
- Export everything from package index files
- TypeScript path resolution handles the mapping

## Git Commit Guidelines

### Commit Message Format

Follow conventional commits with strict formatting:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix  
- `docs`: Documentation changes
- `style`: Code style changes (formatting, missing semicolons, etc.)
- `refactor`: Code refactoring without functional changes
- `test`: Adding or updating tests
- `chore`: Build process, dependencies, or other maintenance

**Scope**: Use package name (e.g., `core`, `three`, `ui`, `wasm`)

**Examples**:
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [louistrue/ifc5cad](https://github.com/louistrue/ifc5cad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
