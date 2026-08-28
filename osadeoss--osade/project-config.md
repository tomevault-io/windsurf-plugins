---
trigger: always_on
description: Visual Studio Code is built with a layered architecture using TypeScript, web APIs and Electron, combining web technologies with native app capabilities. The codebase is organized into key architectural layers:
---

# VS Code Copilot Instructions

## Project Overview

Visual Studio Code is built with a layered architecture using TypeScript, web APIs and Electron, combining web technologies with native app capabilities. The codebase is organized into key architectural layers:

### Root Folders
- `src/`: Main TypeScript source code with unit tests in `src/vs/*/test/` folders
- `build/`: Build scripts and CI/CD tools
- `extensions/`: Built-in extensions that ship with VS Code
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
- `src/vs/sessions/` - Agent sessions window, a dedicated workbench layer for agentic workflows (sits alongside `vs/workbench`, may import from it but not vice versa)

The core architecture follows these principles:
- **Layered architecture** - from `base`, `platform`, `editor`, to `workbench`
- **Dependency injection** - Services are injected through constructor parameters
    - If non-service parameters are needed, they need to come before the service parameters
- **Contribution model** - Features contribute to registries and extension points
- **Cross-platform compatibility** - Abstractions separate platform-specific code

### Built-in Extensions (`extensions/` folder)
The `extensions/` directory contains first-party extensions that ship with VS Code:
- **Language support** - `typescript-language-features/`, `html-language-features/`, `css-language-features/`, etc.
- **Core features** - `git/`, `debug-auto-launch/`, `emmet/`, `markdown-language-features/`
- **Themes** - `theme-*` folders for default color themes
- **Development tools** - `extension-editing/`, `vscode-api-tests/`

Each extension follows the standard VS Code extension structure with `package.json`, TypeScript sources, and contribution points to extend the workbench through the Extension API.

### Finding Related Code
1. **Semantic search first**: Use file search for general concepts
2. **Grep for exact strings**: Use grep for error messages or specific function names
3. **Follow imports**: Check what files import the problematic module
4. **Check test files**: Often reveal usage patterns and expected behavior

## Validating TypeScript changes

Choose validation based on the scope and risk of the change. Large-scale builds and typechecking can be slow, and consume significant resources, so minimize their use. Prefer existing editor or watch-task diagnostics and the smallest targeted tests that cover the changed behavior. Do not start build or watch tasks, run broad type checks, or make type checking a prerequisite for targeted tests solely as a completion ritual.

When running in a VS Code editor window with a workspace folder, use the VS Code task tools for build and watch workflows: inspect the existing task output first, and run an existing task instead of invoking its equivalent shell command. Do not start a duplicate build or watch process when the workspace task already provides current diagnostics. Agents window chats and isolated worktree sessions may not have access to the editor's workspace tasks; use the repository commands directly in those contexts.

Run a targeted type check or build when you are not fully confident in the change, and the change is broad or cross-cutting, it affects build or type configuration, or another validation step reports a compilation problem. When task tools are unavailable or no suitable task exists, useful commands include:

- `npm run typecheck-client` for the main sources under `src/`
- `npm run gulp compile-extensions` for built-in extensions
- `npm run typecheck` from the `build` folder for build tooling

Development compile tasks already type-check their inputs. Do not run `npm run typecheck-client` immediately before `npm run compile` or `npm run compile-client`; choose the command that covers the required validation. When tests only need fresh output files, use the fast one-shot `npm run transpile-client` instead of compiling.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OsadeOSS/Osade](https://github.com/OsadeOSS/Osade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
