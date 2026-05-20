---
trigger: always_on
description: Pointer is a standalone editor distribution built with a layered TypeScript architecture, web APIs, and Electron. It combines web technologies with native desktop capabilities while preserving its own product identity from `product.json`.
---

# Pointer Copilot Instructions

## Project Overview

Pointer is a standalone editor distribution built with a layered TypeScript architecture, web APIs, and Electron. It combines web technologies with native desktop capabilities while preserving its own product identity from `product.json`.

Pointer is not an official Microsoft product, is not Microsoft-signed, and does not use the official Microsoft Marketplace. User-facing product text, app IDs, folders, release artifacts, and docs must say Pointer unless a file is explicitly documenting upstream compatibility or preserving an upstream copyright/license notice.

For the complete human/AI project guide, read `docs/PROJECT_GUIDE.md`. For a concise stack map, read `docs/TECH_STACK.md`.

### Common Pointer Commands

- App-only start: `run\start.bat`
- Live dev mode: `run\start-dev.bat`
- Stop dev processes: `run\dev-stop.bat`
- Release build: `run\build-pointer.bat`
- Release build with ZIP and installer: `run\build-pointer.bat --Zip --Installer`

The codebase is organized into key architectural layers:

### Root Folders
- `src/`: Main TypeScript source code with unit tests in `src/vs/*/test/` folders
- `build/`: Build scripts and CI/CD tools
- `extensions/`: Built-in extensions that ship with Pointer
- `test/`: Integration tests and test infrastructure
- `scripts/`: Development and build scripts
- `resources/`: Static resources (icons, themes, etc.)
- `out/`: Compiled JavaScript output (generated during build)

### Core Architecture (`src/` folder)
- `src/vs/base/` - Foundation utilities and cross-platform abstractions
- `src/vs/platform/` - Platform services and dependency injection infrastructure
- `src/vs/editor/` - Text editor implementation with language services, syntax highlighting, and editing features
- `src/vs/workbench/` - Main application workbench for web and desktop
  - `workbench/browser/` - Core workbench UI components (parts, layout, actions)
  - `workbench/services/` - Service implementations
  - `workbench/contrib/` - Feature contributions (git, debug, search, terminal, etc.)
  - `workbench/api/` - Extension host and VS Code API implementation
- `src/vs/code/` - Electron main process specific implementation
- `src/vs/server/` - Server specific implementation
- `src/vs/sessions/` - Agent sessions window, a dedicated workbench layer for agentic workflows (sits alongside `vs/workbench`, may import from it only where the local architecture allows it)

The core architecture follows these principles:
- **Layered architecture** - from `base`, `platform`, `editor`, to `workbench`
- **Dependency injection** - Services are injected through constructor parameters
    - If non-service parameters are needed, they need to come after the service parameters
- **Contribution model** - Features contribute to registries and extension points
- **Cross-platform compatibility** - Abstractions separate platform-specific code

### Built-in Extensions (`extensions/` folder)
The `extensions/` directory contains built-in extensions that ship with Pointer:
- **Language support** - `typescript-language-features/`, `html-language-features/`, `css-language-features/`, etc.
- **Core features** - `git/`, `debug-auto-launch/`, `emmet/`, `markdown-language-features/`
- **Themes** - `theme-*` folders for default color themes
- **Development tools** - `extension-editing/`, `vscode-api-tests/`

Each extension follows the standard extension structure with `package.json`, TypeScript sources, and contribution points to extend the workbench through the Extension API.

### Finding Related Code
1. **Semantic search first**: Use file search for general concepts
2. **Grep for exact strings**: Use grep for error messages or specific function names
3. **Follow imports**: Check what files import the problematic module
4. **Check test files**: Often reveal usage patterns and expected behavior

## Validating TypeScript changes

MANDATORY: Always check for compilation errors before running any tests or validation scripts, or declaring work complete, then fix all compilation errors before moving forward.

- NEVER run tests if there are compilation errors
- NEVER use `npm run compile` to compile TypeScript files

### TypeScript compilation steps
- If the `#runTasks/getTaskOutput` tool is available, check the Pointer/VS Code build watch task output for compilation errors. This task runs `Core - Build` and `Ext - Build` to incrementally compile Pointer TypeScript sources and built-in extensions. Start the task if it's not already running in the background.
- If the tool is not available (e.g. in CLI environments) and you only changed code under `src/`, run `npm run compile-check-ts-native` after making changes to type-check the main Pointer sources (it validates `./src/tsconfig.json`).
- If you changed built-in extensions under `extensions/` and the tool is not available, run the corresponding gulp task `npm run gulp compile-extensions` instead so that TypeScript errors in extensions are also reported.
- For TypeScript changes in the `build` folder, you can simply run `npm run typecheck` in the `build` folder.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PointerIDE/Pointer](https://github.com/PointerIDE/Pointer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
