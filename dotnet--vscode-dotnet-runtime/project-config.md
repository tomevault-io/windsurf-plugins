---
trigger: always_on
description: This document provides guidance for GitHub Copilot when working with the vscode-dotnet-runtime repository.
---

# GitHub Copilot Instructions for vscode-dotnet-runtime

This document provides guidance for GitHub Copilot when working with the vscode-dotnet-runtime repository.

## Repository Overview

This repository contains VS Code extensions for acquiring and managing .NET runtimes and SDKs:

- **vscode-dotnet-runtime-library**: Core library for .NET acquisition logic (shared by extensions)
- **vscode-dotnet-runtime-extension**: Extension for installing .NET runtimes (designed to be used by other extensions)
- **vscode-dotnet-sdk-extension**: Extension for installing .NET SDKs (for internal features, not for general use)
- **sample**: Sample extension demonstrating usage of the runtime acquisition APIs

### Architecture

The repository follows a layered architecture:
1. **Library layer** (`vscode-dotnet-runtime-library`): Contains all acquisition logic, installers, version resolvers, and utilities
2. **Extension layer** (`vscode-dotnet-runtime-extension`, `vscode-dotnet-sdk-extension`): VS Code extension wrappers that expose commands and APIs
3. **Sample layer** (`sample`): Demonstrates proper usage of the extensions

Core logic goes in the library, UI/commands in extensions.

## Build Process

### Full Build

```bash
./build.sh   # Linux/macOS
build.cmd    # Windows
```

The build script downloads .NET install scripts, compiles all components (library, extensions, sample), and runs mock webpack.

### Individual Component Build

```bash
cd <component-directory>
npm ci              # Install dependencies
npm run compile     # Compile TypeScript
npm run clean       # Remove dist/ artifacts
```

## Testing

### Two Test Types

1. **Library Unit Tests** (`vscode-dotnet-runtime-library/src/test/unit/`)
   - Mocha with TDD interface, fast, no VS Code runtime required
   - Run: `npm run unit-test` (in library directory)

2. **Extension Functional Tests** (`*-extension/src/test/functional/`)
   - End-to-end tests using `@vscode/test-electron`, slower
   - Run: `npm run test` (in extension directory)

### Running Tests

```bash
# Full test suite
./test.sh --eslint  # All tests + linting
./test.sh lib       # Library only
./test.sh rnt       # Runtime extension only
./test.sh sdk       # SDK extension only

# Specific unit test (compile first!)
npx mocha --bail -u tdd -- dist/test/unit/LocalInstallUpdateService.test.js
```

**Important**: Tests run against compiled JavaScript in `dist/`, so run `npm run compile` first.

### Debugging Tests

- Library tests: Open `vscode-dotnet-runtime-library` workspace, use VS Code test runner
- Extension tests: Open extension workspace, use debug launch configurations
- Add logging statements to understand test failures; compiled JS is in `dist/` directory

## Linting and Code Style

```bash
npm run lint  # From repository root (ESLint + TypeScript + auto-fix)
```

### Code Conventions

- **TypeScript**: All code
- **File Headers**: Use .NET Foundation license header (see `contributing.md`)
- **Naming**: PascalCase (classes/interfaces/types), camelCase (variables/functions/methods), UPPER_CASE (constants)
- **Testing**: Follow TDD (write tests first when fixing bugs)
- **Security**: Be extra careful - this code downloads and executes .NET installers

## Common Development Workflows

### Updating Dependencies

When asked to update dependencies, follow this process:

```bash
# Update all components in order
cd <repo-root>
npm update && yarn install && yarn upgrade

cd vscode-dotnet-runtime-library
npm update && yarn install && yarn upgrade

cd ../vscode-dotnet-runtime-extension
npm update && yarn install && yarn upgrade

cd ../vscode-dotnet-runtime-library
npm update && yarn upgrade

cd ../sample
npm update && yarn upgrade

cd ../vscode-dotnet-sdk-extension
npm update && yarn upgrade

cd ..
./build.sh  # or build.cmd on Windows
```

### Version Bumping

**Important**: Only bump extension versions when explicitly requested. If not requested, do NOT run `npm version patch`.

When version bumping IS requested:
- Run `npm version patch` in `vscode-dotnet-runtime-extension` after its updates
- Run `npm version patch` in `sample` after its updates
- Update the corresponding CHANGELOG.md file with the new version and changes:
  - `vscode-dotnet-runtime-extension/CHANGELOG.md` for runtime extension
  - `vscode-dotnet-sdk-extension/CHANGELOG.md` for SDK extension

### Fixing a Bug

1. Write a failing test that reproduces the bug
2. Fix the bug with minimal changes
3. Verify: `npx mocha --bail -u tdd -- dist/test/unit/AffectedFile.test.js`
4. Run full test suite to ensure no regressions

### Adding a New Command

When registering a new `vscode.commands.registerCommand` in an extension:
1. Add the command key to the `commandKeys` namespace in the extension's `extension.ts`
2. Document the command in [`Documentation/commands.md`](../Documentation/commands.md) — include the accepted context type, return type, offline behavior, and version format restrictions
3. Add a sample invocation in [`sample/src/extension.ts`](../sample/src/extension.ts) if the command is intended for use by other extensions
4. Register the command in the extension's `package.json` under `contributes.commands` if it should be user-facing

### Making Library Changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/vscode-dotnet-runtime](https://github.com/dotnet/vscode-dotnet-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
