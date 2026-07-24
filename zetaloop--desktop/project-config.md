---
trigger: always_on
description: This repository contains GitHub Desktop, an open-source Electron-based GitHub application written in TypeScript and React.
---

# GitHub Desktop - Copilot Instructions

This repository contains GitHub Desktop, an open-source Electron-based GitHub application written in TypeScript and React.

## Technology Stack

- **Language**: TypeScript (strict mode enabled)
- **UI Framework**: React 16.x
- **Runtime**: Electron > 38.x (see `.npmrc` for specific version)
- **Build Tool**: Webpack with parallel builds
- **Package Manager**: Yarn (>= 1.21.1)
- **Node Version**: >= 22 (see `.nvmrc` for specific version)
- **Testing**: Node.js built-in test runner (run using `yarn test`, optionally providing one or more test files e.g `yarn test app/test/unit/repository-list-test.ts`)

## Code Style & Conventions

GitHub Desktop has been developed for many years through many iterations of technologies and coding styles, there may be conflicting styles in different parts of the codebase. When contributing new code or refactoring existing code, please follow the conventions outlined below.

### TypeScript Style

- Avoid creating new classes unless necessary; prefer functions and interfaces/types, sticking to more idiomatic TypeScript/JavaScript patterns.
- Avoid using enums; prefer union types of string literals instead.
- **Use strict TypeScript** with all strict mode checks enabled
- **Naming conventions**:
  - PascalCase for classes
  - camelCase for methods and properties
  - Interfaces MUST start with `I` prefix (e.g., `IRepository`, `ICommit`)
  - Avoid reserved keywords as variable names (`any`, `Number`, `String`, `Boolean`, `Undefined`, etc.)
- **Type safety**:
  - Avoid using `as` for type assertions, prefer proper type narrowing and guards.
  - Use the `assertNever` helper (from `app/src/lib/fatal-error.ts`) for exhaustiveness checks in switch statements or conditional logic
  - Avoid non-null assertions (`!`) unless absolutely necessary
  - Write custom type definitions when none exist
  - Avoid `any` unless absolutely necessary
- **Member ordering in classes**:
  1. Static fields
  2. Static methods
  3. Instance fields
  4. Abstract methods
  5. Constructor
  6. Instance methods
- **Visibility modifiers**: Always use explicit member accessibility (`public`, `private`, `protected`)
- **Avoid default exports**: Use named exports only

### React Conventions

- **Props and State**: Always use `readonly` for props and state types to prevent accidental mutation
- **JSX**: Always use explicit boolean values (e.g., `<Component visible={true} />` instead of `<Component visible />`)
- **No binding in JSX**: Use arrow functions or pre-bind methods instead of binding in render
- **No string refs**: Use React refs API instead
- **Accessibility**: Autofocus is allowed when used appropriately in dialogs and focused contexts

### Immutability & Pure Functions

- **Prefer `const` over `let`**: Use `const` whenever possible to enforce immutability
- **Prefer ternary over reassignment**: Use `const a = condition ? value : otherValue` instead of `let` with conditional reassignment
- **Pure functions**: Write functions that operate only on their parameters when possible
- **Lift computation logic**: Separate data gathering from data processing into different functions
- **Use readonly arrays**: Mark arrays and objects as `readonly` in interfaces and function parameters

### Import Restrictions

- **Never import `ipcRenderer` directly** from `electron` or `electron/renderer` - use `import * as ipcRenderer from 'ipc-renderer'` (app/src/lib/ipc-renderer.ts) for strongly typed IPC methods
- **Never import `ipcMain` directly** from `electron` or `electron/main` - use `import * as ipcMain from 'ipc-main'` (app/src/lib/ipc-main.ts) for strongly typed IPC methods

### Code Quality

- **Curly braces**: Always use curly braces for control structures
- **Strict equality**: Use `===` and `!==` (smart equality checking allowed)
- **No `eval`**: Never use `eval()`
- **No `var`**: Use `const` or `let`
- **Async operations**: Use async/await, avoid synchronous Node.js APIs in application code (use `Sync` suffix when necessary)

### Documentation

- **Use JSDoc format** for documentation with `/**` opener (exactly two stars)
- **Document public APIs**: All public classes, methods, and properties should have JSDoc comments
- **Format**: Use a short title line followed by blank line before detailed description
- **AppStore methods**: Internal methods called by Dispatcher should be prefixed with `_` and include comment: `/** This shouldn't be called directly. See 'Dispatcher'. */`

### ESLint Rules

The codebase uses comprehensive ESLint rules. Key custom rules:
- `insecure-random`: Prevents use of insecure random number generation
- `react-no-unbound-dispatcher-props`: Enforces proper dispatcher prop handling
- `react-readonly-props-and-state`: Prevents mutation of React props and state
- `react-proper-lifecycle-methods`: Enforces correct React lifecycle usage
- `no-loosely-typed-webcontents-ipc`: Ensures type-safe IPC communication

## Building & Testing

### Development Workflow

```bash
# Install dependencies
yarn

# Development build
yarn build:dev
```

### Testing

```bash
# Run all unit tests
yarn test

# Run specific test file
yarn test <file>

# Run tests in directory
yarn test <directory>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zetaloop/desktop](https://github.com/zetaloop/desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
