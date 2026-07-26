---
trigger: always_on
description: This is a **multi-repository development workspace** for the Microsoft MakeCode Arcade ecosystem. The workspace contains four linked repositories that work together to provide a complete block-based and TypeScript programming environment for creating retro-style arcade games.
---

# MakeCode Arcade Multi-Repository Workspace Instructions for GitHub Copilot

## Workspace Overview
This is a **multi-repository development workspace** for the Microsoft MakeCode Arcade ecosystem. The workspace contains four linked repositories that work together to provide a complete block-based and TypeScript programming environment for creating retro-style arcade games.

## General Development Guidelines

### Code Quality Standards
- Follow TypeScript strict mode conventions
- Use semantic versioning for all packages
- Maintain backward compatibility when possible
- Write clear, self-documenting code with JSDoc comments
- Follow the existing code style and patterns in each component

### Code Generation Standards
- Always include proper type annotations
- Use `export` statements for public APIs
- Prefer `const` over `let` where possible
- Use arrow functions for callbacks and short functions
- Include error handling for async operations

## Workspace Structure & Repository Responsibilities

### Workspace Layout
```
workspace-root/
├── pxt/                    # Core MakeCode framework (pxt-core)
├── pxt-common-packages/    # Shared cross-target APIs and game engine
└── pxt-arcade/             # Arcade target configuration and extensions
```

### Repository-Specific Responsibilities

#### pxt/ (pxt-core)
**Repository**: https://github.com/microsoft/pxt
**Local Path**: `../pxt` relative to pxt-arcade
**Package Name**: `pxt-core`
**Changes should go here for:**
- Core editor functionality (Monaco editor integration, block editor)
- Build system and compilation pipeline (`gulpfile.js`, `Makefile`)
- Package management and extension loading
- Core APIs and runtime
- Static TypeScript definitions in `libs/` (e.g., `libs/base/`, `libs/core/`)
- Web app infrastructure (`webapp/`, `pxteditor/`)
- Language services and IntelliSense (`pxtcompiler/`)
- Import/export functionality
- Cloud services integration (`pxtservices/`)

#### pxt-common-packages/
**Repository**: https://github.com/microsoft/pxt-common-packages  
**Local Path**: `../pxt-common-packages` relative to pxt-arcade
**Package Name**: `pxt-common-packages`
**Changes should go here for:**
- Cross-target game engine APIs (sprites, physics, input)
- Hardware abstraction layers
- Common blocks and TypeScript APIs in `libs/`
- Shared simulator functionality in `sim/`
- Device-specific implementations (`libs/core---*/`)

#### pxt-arcade-sim (internal dependency)
**Note**: This is an internal Microsoft repository not accessible to external contributors.
**Changes that might require coordination:**
- Complex simulator functionality changes
- Game rendering and display issues
- Input handling problems in simulator
- Audio/sound systems in simulator

*For external contributors: If you encounter simulator-specific issues, please file them in this repository and maintainers will coordinate with the appropriate internal teams.*

#### pxt-arcade/ (This Repository)
**Repository**: https://github.com/microsoft/pxt-arcade
**Local Path**: `.` (workspace root)
**Package Name**: `pxt-arcade`
**Changes should go here for:**
- Target configuration (`pxtarget.json`, `targetconfig.json`)
- Arcade-specific library extensions in `libs/`
- Arcade-specific documentation (`docs/`, `docfiles/`)
- Editor customizations (`editor/`)
- Arcade themes and styling (`theme/`)
- Build and deployment scripts (`scripts/`)

## Code Change Guidelines

### Changes that belong in THIS repository (pxt-arcade):
1. **Arcade-specific features**
   - Arcade-specific blocks or APIs in `libs/`. These tend to be specifically extensions, or overrides from the common game library features found in pxt-common-packages
   - Target configuration changes in `pxtarget.json` or `targetconfig.json`
       - `pxtarget.json` is the main target configuration file that defines the capabilities and features of the Arcade target. This is part of a released version of the application.
       - `targetconfig.json` is served live and may include changes that occur between releases
   - Arcade hardware support
   - Documentation specific to Arcade in `docs/` and `docfiles/`

2. **Arcade target configuration**
   - Build configurations specific to arcade
   - Theme and styling changes in `theme/`
   - Arcade-specific editor extensions in `editor/`

### Changes that should go to pxt-core:
1. **Core functionality**
   - Block editor improvements
   - Monaco/TypeScript editor changes
   - Build system modifications
   - Package management features
   - Cloud/sharing functionality
   - Language service improvements
   
2. **Static TypeScript definitions**
   - Core language APIs in `libs/base/`, `libs/core/`
   - Fundamental data types and primitives
   - Basic runtime functionality
   - Cross-target foundational APIs

### Changes that should go to pxt-common-packages:
1. **Cross-target APIs**
   - Sprite system improvements
   - Physics engine changes
   - Game loop modifications
   - Input handling that affects multiple targets
   - Hardware abstraction layers

## Development Workflow & Dependencies


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/pxt-arcade](https://github.com/microsoft/pxt-arcade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
