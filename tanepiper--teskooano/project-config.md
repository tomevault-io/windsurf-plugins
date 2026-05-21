---
trigger: always_on
description: A guide for AI coding agents working on the Teskooano N-Body simulation engine.
---

# AGENTS.md

A guide for AI coding agents working on the Teskooano N-Body simulation engine.

## Project Overview

**Teskooano** is a 3D N-Body simulation engine that accurately simulates real physics and provides a multi-view experience in real time. It features collision detection, realistic orbital mechanics, and procedural generation to create unique star systems.

### Key Architecture

- **Modular Monorepo**: Managed with `moon` and `proto` for streamlined development
- **Plugin-Based UI**: DockView for modular, dockable panels with custom plugin system
- **Reactive State**: Centralized RxJS-based state management
- **3D Rendering**: Three.js with custom renderer packages and GLSL shaders
- **Physics Engine**: N-Body simulation with Verlet integrator and orbital mechanics

## Setup Commands

### Prerequisites

- Install [moon](https://moonrepo.dev/) and [proto](https://moonrepo.dev/proto) for task running and dependency management
- Node.js 24.2.0 (specified in package.json engines)

### Installation & Development

```bash
# Clone and setup
git clone https://github.com/tanepiper/teskooano.git
cd teskooano

# Install dependencies and start the main app
proto use
moon run teskooano:dev
```

The application will be available at `http://localhost:3000`.

### Key Commands

- **Start dev server**: `moon run teskooano:dev`
- **Build**: `moon run teskooano:build`
- **Run tests**: `moon run :test` (runs all package tests)
- **Run specific package tests**: `moon run <package-name>:test`
- **Format code**: `npm run format` (prettier)

## Code Style & Conventions

### TypeScript Standards

- **Strict Mode**: Always use strict TypeScript configuration
- **Type Safety**: Prefer explicit types over inference when clarity is needed
- **Interfaces**: Define dedicated TypeScript interfaces for constructor options instead of inline object types
- **JSDoc**: Include documentation but omit explicit type annotations (types are in TypeScript code)

### Code Style

- **Indentation**: Use 2-space indentation
- **Naming**:
  - `PascalCase` for classes, interfaces, and types
  - `camelCase` for variables, properties, and functions
  - `UPPER_CASE` for constants
- **File Size**: Target max 300-400 lines per file
- **Modularity**: Prefer small, composable files with single responsibility

### Import Patterns

- **Static Imports**: Use ES import statements at the top of files exclusively
- **Never use**: `require()` or dynamic `import()`
- **Path Aliases**: Use `@teskooano/*` aliases defined in tsconfig.json

## Package Architecture

### Core Packages (`@teskooano/core-*`)

- **Purpose**: Application-agnostic business logic and data structures
- **Dependencies**: No UI-specific dependencies (no DockView, ThreeJS, etc.)
- **Examples**: `core-math`, `core-physics`, `core-state`, `core-debug`

### Renderer Packages (`@teskooano/renderer-*`)

- **Purpose**: Three.js rendering modules with specific responsibilities
- **Pattern**: Compositional architecture with LOD management
- **Examples**: `renderer-threejs-core`, `renderer-threejs-celestial`, `renderer-threejs-orbits`

### System Packages (`@teskooano/systems-*`)

- **Purpose**: Domain-specific logic for celestial systems
- **Examples**: `systems-procedural-generation`, `systems-solar-system`

### App Packages (`@teskooano/app-*`)

- **Purpose**: Application-specific functionality
- **Examples**: `app-simulation`, `app-ui-plugin`, `design-system`

## Plugin Development Patterns

### MVC Architecture

All UI plugins follow a strict **Model-View-Controller (MVC)** pattern:

```
plugin-name/
├── controller/
│   └── PluginName.controller.ts    # Business logic
├── view/
│   ├── PluginName.view.ts          # Custom element (dumb view)
│   └── PluginName.template.ts      # HTML/CSS template
├── services/                       # Reusable business logic
├── index.ts                        # Plugin registration
└── README.md                       # Architecture documentation
```

### Plugin Registration

- **Custom Elements**: Define as `ComponentConfig` objects in plugin definition
- **No Manual Registration**: Don't call `customElements.define()` manually
- **Plugin Manager**: Handles registration automatically during plugin loading

### Component Organization

Plugins should follow a consistent directory structure based on component complexity:

**Flat Structure (Most Common)**

```
plugin-name/
├── controller/
│   └── PluginName.controller.ts
├── view/
│   ├── PluginName.view.ts
│   └── PluginName.template.ts
├── services/                      # Optional
├── index.ts
└── README.md
```

**Nested Structure (Complex Plugins)**

```
plugin-name/
├── controller/
├── view/
├── components/                    # For reusable sub-components
│   ├── component-name/
│   │   ├── ComponentName.ts
│   │   └── ComponentName.template.ts
│   └── another-component/
├── services/
├── index.ts
└── README.md
```

**When to Use `components/` Subdirectory:**

- Component is **reusable** across multiple plugins or views
- Component has **complex internal structure** (own controller/view/template pattern)
- Component is **not a standalone plugin** (doesn't register with plugin manager)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanepiper/teskooano](https://github.com/tanepiper/teskooano) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
