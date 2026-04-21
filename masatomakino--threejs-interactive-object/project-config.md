---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## DevContainer Environment

This project uses DevContainer for isolated npm execution to prevent supply chain attacks.

### Required Setup
- Docker Desktop installed and running
- DevContainer CLI installed (`npm install -g @devcontainers/cli`)

### Container Management

#### Starting Container
- `devcontainer up --workspace-folder .` - Start the development container

#### Executing Commands
- `devcontainer exec --workspace-folder . <command>` - Execute command in container

#### Restarting Container
**Important**: Always use this procedure to restart the container. Using `docker start` alone will not apply DevContainer configuration (e.g., `--shm-size=2g`).

```bash
# 1. Stop the container
docker stop threejs-interactive-object-npm-runner

# 2. Start with DevContainer configuration
devcontainer up --workspace-folder .
```

**When to restart**:
- After KillShell forcefully terminates a hung test process (orphaned Chrome processes remain)
- After extended development sessions with watch mode
- When "Port 63315 is in use" messages appear

### Important Policy
- **Host OS npm/npx execution is prohibited** for security isolation
- All npm/npx commands must be executed through `devcontainer exec`
- Git hooks automatically use devcontainer for code quality checks

### Container Details
- **Image**: Node 22 (bookworm-slim) with Chrome and xvfb
- **Container Name**: `threejs-interactive-object-npm-runner`
- **Ports**: 3000 (Dev Server), 3001 (Dev Server UI)

## Development Commands

### Build and Development
- `devcontainer exec --workspace-folder . npm run build` - Full build: TypeScript compilation, demo generation, and API docs
- `devcontainer exec --workspace-folder . npm run buildTS` - TypeScript compilation only
- `devcontainer exec --workspace-folder . npm run start:dev` - Start comprehensive development environment (runs server, TypeScript watch, and demo watch in parallel)
- `devcontainer exec --workspace-folder . npm run server` - Start browser-sync server for demo pages with live reload
- `devcontainer exec --workspace-folder . npm run watch:tsc` - TypeScript compilation in watch mode with incremental builds
- `devcontainer exec --workspace-folder . npm run watch:demo` - Demo page generation in watch mode

### Testing
- `devcontainer exec --workspace-folder . npm run test:ci` - Run all tests with virtual display (for DevContainer/CI)
- `devcontainer exec --workspace-folder . npm run test:watch:ci` - Run tests in watch mode with virtual display
- `devcontainer exec --workspace-folder . npm run coverage:ci` - Generate comprehensive test coverage report with virtual display

> **Important**: Always use commands with `:ci` suffix in DevContainer (`test:ci`, `test:watch:ci`, `coverage:ci`). Commands without `:ci` suffix are kept only for backward compatibility and should not be used.

### Code Quality
- `devcontainer exec --workspace-folder . npx biome check` - Run linter and formatter
- `devcontainer exec --workspace-folder . npx biome check --write` - Run linter and formatter with auto-fix

### Documentation
- `devcontainer exec --workspace-folder . npm run typedocs` - Generate TypeDoc API documentation
- `devcontainer exec --workspace-folder . npm run demo` - Generate demo pages with custom canvas element and ES2020 compilation target

## Architecture Overview

This is a TypeScript library for creating mouse-interactive objects in Three.js applications. The architecture follows a clear separation of concerns:

### Core Components

**MouseEventManager** (`src/MouseEventManager.ts`)
- Central event dispatcher that handles mouse interactions on canvas
- Uses raycasting to detect object intersections
- Manages throttling for performance (33ms default)
- Identifies interactive objects via `IClickableObject3D` interface

**Interactive Views** (`src/view/`)
- `InteractiveMesh.ts` - Mesh-based interactive objects:
  - `InteractiveMesh<Value, InteractionHandler>` - **@internal** Generic base class (not exported; use concrete subclasses)
    - `ClickableMesh<Value>` - Basic clickable mesh with button-like behavior
    - `CheckBoxMesh<Value>` - Checkbox mesh with toggle selection behavior  
    - `RadioButtonMesh<Value>` - Radio button mesh with exclusive selection behavior
- `InteractiveSprite.ts` - Sprite-based interactive objects:
  - `InteractiveSprite<Value, Handler>` - **@internal** Generic base class (not exported; use concrete subclasses)
    - `ClickableSprite<Value>` - Basic clickable sprite with button-like behavior
    - `CheckBoxSprite<Value>` - Checkbox sprite with toggle selection behavior
    - `RadioButtonSprite<Value>` - Radio button sprite with exclusive selection behavior
- `ClickableGroup.ts` - Group-based interactive container:
  - `ClickableGroup<Value>` - Interactive Three.js Group with UUID-based collision deduplication

**Interaction Handlers** (`src/interactionHandler/`)
- `ButtonInteractionHandler` - Base interaction logic with EventEmitter
- `CheckBoxInteractionHandler` - Checkbox-specific behavior (toggle state)
- `RadioButtonInteractionHandler` - Radio button behavior (exclusive selection)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MasatoMakino) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
