---
trigger: always_on
description: **Teskooano (N-Body Sim):**
---

---
# Teskooano Coding Style Guide

## Project Overview

**Teskooano (N-Body Sim):**
*   **Core:** N-Body simulation with real physics & orbital mechanics using ThreeJS
*   **Architecture:** Modular monorepo with clear separation between core packages, renderers, and applications
*   **UI:** DockView for modular UI with plugin system. Web Components for custom elements
*   **3D Engine:** ThreeJS (Vanilla TypeScript) with custom renderer packages
*   **Workflow:** Follow established patterns, commit frequently with conventional commits

## Key Development Rules & Tools

**Monorepo Management:**
*   **`proto` (Dependency Management):** Use for local copies of system dependencies
*   **`moon` (Repository Management):** Use for overall monorepo structure and tasks
*   **Package Management:** Use `npm` exclusively with `file:` dependencies for inter-package references
*   **Running TypeScript:** Use `tsx` (e.g., `npx tsx src/index.ts`)

**TypeScript Standards:**
*   **Strict Mode:** Always use strict TypeScript configuration
*   **Type Safety:** Prefer explicit types over inference when clarity is needed
*   **Interfaces:** Define dedicated TypeScript interfaces for constructor options instead of inline object types
*   **JSDoc:** Include documentation but omit explicit type annotations (types are in TypeScript code)

**Code Style & Structure:**
*   **Indentation:** Use 2-space indentation
*   **Cleanliness:** Remove dead code and large comment blocks
*   **Modularity:** Prefer small, composable files (target max 300-400 lines)
*   **File Organization:** Follow established patterns for each package type

## Package Architecture Patterns

### Core Packages (`@teskooano/core-*`)
*   **Purpose:** Application-agnostic business logic and data structures
*   **Dependencies:** No UI-specific dependencies (no DockView, ThreeJS, etc.)
*   **Examples:** `core-math`, `core-physics`, `core-state`, `core-debug`
*   **Pattern:** Pure functions, state management, mathematical operations

### Data Packages (`@teskooano/data-*`)
*   **Purpose:** Shared data types and interfaces used across packages
*   **Dependencies:** Minimal, only core types
*   **Examples:** `data-types` (RenderableCelestialObject, CelestialType, etc.)
*   **Pattern:** Type definitions, enums, interfaces

### Renderer Packages (`@teskooano/renderer-*`)
*   **Purpose:** ThreeJS-specific rendering implementations
*   **Dependencies:** ThreeJS, core packages, data packages
*   **Examples:** `renderer-threejs-celestial`, `renderer-threejs-labels`, `renderer-threejs-orbits`
*   **Pattern:** Renderer classes, shader materials, GPU optimizations

### Celestial Packages (`@teskooano/celestials-*`)
*   **Purpose:** Celestial object definitions and factories
*   **Dependencies:** Data packages, core packages
*   **Examples:** `celestials-stars-main-sequence`, `celestials-terrestrial`
*   **Pattern:** Factory functions, celestial object classes

### System Packages (`@teskooano/systems-*`)
*   **Purpose:** Complex system implementations (procedural generation, solar system)
*   **Dependencies:** Core packages, data packages
*   **Examples:** `systems-procedural-generation`, `systems-solar-system`
*   **Pattern:** System managers, generators, complex business logic

### App Packages (`@teskooano/app-*`)
*   **Purpose:** Application-specific functionality
*   **Dependencies:** All other package types
*   **Examples:** `app-simulation`, `app-ui-plugin`, `app-design-system`
*   **Pattern:** Application logic, UI components, design system

## Frontend Development Patterns

### MVC Architecture
*   **View:** Custom elements (Web Components) responsible only for rendering
*   **Controller:** Dedicated classes that handle business logic and state management
*   **Model:** Data structures and state management (often RxJS Observables)

### Component Structure
```
component-name/
├── view/
│   ├── component-name.component.ts    # Custom element class
│   └── component-name.template.ts     # HTML template and CSS
├── controller/
│   ├── component-name.controller.ts    # Main controller
│   ├── component-name.streams.ts       # RxJS streams
│   ├── component-name.effects.ts       # Side effects
│   └── component-name.utils.ts         # Utility functions
└── services/                          # (if applicable)
    └── service-name.ts                # Reusable services
```

### Plugin System
*   **Plugin Registration:** Use `TeskooanoPlugin` interface with components, functions, panels
*   **Dynamic Loading:** Plugins loaded via `PluginManager` with HMR support
*   **Dependency Injection:** Pass `PluginExecutionContext` to plugin constructors
*   **Custom Elements:** Register via `ComponentConfig` in plugin definition, not manually

### State Management
*   **RxJS:** Use for reactive state management and data pipelines
*   **Observables:** Prefer `Observable<T>` over direct state access
*   **Context Pattern:** Pass dependencies down through constructor injection
*   **Global State:** Use `@teskooano/core-state` for shared application state

## 3D Rendering Patterns

### Renderer Architecture
*   **Compositional Pattern:** Use layers, composite renderers, and component systems
*   **LOD System:** Implement Level of Detail with distance-based switching

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanepiper/teskooano](https://github.com/tanepiper/teskooano) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
