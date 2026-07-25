---
trigger: always_on
description: This document contains rules, conventions, and best practices for AI agents and developers working on the PlayCanvas Engine codebase.
---

# Agent Guidelines for PlayCanvas Engine

This document contains rules, conventions, and best practices for AI agents and developers working on the PlayCanvas Engine codebase.

## Project Overview

PlayCanvas is an open-source WebGL/WebGPU game engine written in JavaScript. It's a performance-critical library used by thousands of developers worldwide.

- **Language**: JavaScript (ES2022) with JSDoc for TypeScript type definitions
- **Module System**: ES Modules
- **Node Version**: >=18.0.0
- **Build System**: Rollup
- **Testing**: Mocha + Chai + Sinon
- **Linting**: ESLint with @playcanvas/eslint-config
- **License**: MIT

## General Code Rules

### 1. Code Style and Formatting

- **Follow ESLint rules**: Always run `npm run lint` before committing
  - **Important**: Only fix lint issues in code you are actively modifying or creating
  - Do not fix pre-existing lint issues in unrelated code unless specifically asked
  - Focus on ensuring new and refactored code is lint-free
- **Use JSDoc comments**: All public APIs must have comprehensive JSDoc documentation
- **Module imports**: Use ES6 import/export syntax
- **Naming conventions**:
  - Classes: PascalCase (e.g., `GraphicsDevice`, `Entity`)
  - Functions/methods: camelCase (e.g., `createShader`, `setPosition`)
  - Constants: UPPER_SNAKE_CASE (e.g., `PIXELFORMAT_RGBA8`)

### 2. File Organization

- **Source files**: All engine source code goes in `src/`
- **Directory structure**:
  - `src/core/` - Core utilities and data structures
  - `src/platform/` - Platform-specific code (graphics, audio, input)
  - `src/scene/` - Scene graph, rendering, materials, shaders
  - `src/framework/` - High-level components and application framework
  - `src/extras/` - Optional extras and utilities
- **Build output**: Generated files go in `build/` (never edit these directly)
- **Examples**: Live in `examples/src/examples/`
- **Tests**: Unit tests go in `test/` with `.mjs` extension
- **File naming**: Module file names should match the main class they contain
  - Use kebab-case for file names (e.g., `graphics-device.js` for `GraphicsDevice` class)
  - If a class is renamed, the file should be renamed to match
  - Multiple related classes can share a file if they're tightly coupled

#### Module Dependency Hierarchy

The codebase follows a strict hierarchical structure to maintain clean architecture:

```
core → platform → scene → framework
```

**Rules**:
- Lower-level modules **cannot import** from higher-level modules
- Lower-level modules **cannot use instances** from higher-level modules
- Example: `core/` cannot import from `platform/`, `scene/`, or `framework/`
- Example: `scene/` cannot import from `framework/`

**Known Exception**:
- `CameraComponent` (from `framework/`) is currently used in multiple places at the `scene/` level
- **Do not introduce new exceptions** unless explicitly requested and confirmed
- When in doubt, ask before breaking the hierarchy

This hierarchy ensures:
- Clean separation of concerns
- Prevents circular dependencies
- Makes the codebase more maintainable and testable

### 3. Documentation Standards

- **JSDoc is mandatory** for all public APIs:
  ```javascript
  /**
   * Brief description of the function.
   *
   * @param {string} name - Parameter description.
   * @param {number} [optional=0] - Optional parameter with default.
   * @returns {boolean} Return value description.
   * @example
   * const result = myFunction('test', 5);
   */
  ```
- **Include examples** for complex APIs
- **Document side effects**: Mention if a function modifies state
- **Link related APIs**: Use `@see` tags to cross-reference
- **Mark deprecations**: Use `@deprecated` with migration instructions

### 4. TypeScript Definitions

- JSDoc comments are used to generate TypeScript definitions
- Run `npm run build:types` to generate `.d.ts` files
- Test types with `npm run test:types`
- Use proper JSDoc type annotations:
  - `@type {TypeName}` for variables
  - `@param {TypeName} paramName` for parameters
  - `@returns {TypeName}` for return values
  - Support for generics, unions, and complex types
- **Type-only imports**: Use `@import` for types referenced in JSDoc comments
  - These imports are only for type information, not runtime code
  - Place at the top of the file in a JSDoc comment block
  - Example:
    ```javascript
    /**
     * @import { Texture } from './texture.js'
     * @import { Shader } from './shader.js'
     */
    ```
  - These help TypeScript understand types without adding runtime dependencies

### 5. Testing

- **Write tests** for all new features and bug fixes if instructed
- **Test location**: `test/` directory, organized by module
- **Test naming**: Use descriptive names that explain what is being tested
- **Run tests**: `npm test` (or `npm run test:coverage` for coverage)
- **Test structure**:
  ```javascript
  describe('ClassName', function () {
      describe('#methodName', function () {
          it('should do something specific', function () {
              // Test implementation
          });
      });
  });
  ```

### 6. Performance Considerations

This is a **performance-critical** engine. Always consider:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [playcanvas/engine](https://github.com/playcanvas/engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
