---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude, Codex, and other LLM tools) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude, Codex, and other LLM tools) when working with code in this repository.

## Project Overview

This is **OHIF** v3  (Open Health Imaging Foundation) - a medical imaging viewer. It's an extensible web imaging platform.
## Development Commands

### Main Development
```bash
# Start development server for all packages
yarn dev
```

### Building
```bash
# Build all packages for production
yarn build

# Build specific packages
cd platform/app && yarn build    # Main viewer app
```

## Architecture Overview

### Monorepo Structure
- **`platform/`** - Core OHIF infrastructure
  - `app/` - Main viewer application (`@ohif/viewer`)
  - `core/` - Core services and utilities
  - `ui-next/` - Modern UI component library
- **`extensions/`** - Modular functionality plugins
- **`modes/`** - Application workflow configurations

### Key Extension Architecture

**Extension System**: Each extension exports modules (viewports, tools, panels, commands) that the app dynamically loads. Extensions are self-contained with their own webpack builds.


**Core Extensions:**
- `cornerstone/` - Medical image rendering engine
- `cornerstone-dicom-pmp/` - DICOM PMP support
- `cornerstone-dicom-seg/` - DICOM Segmentation support
- `cornerstone-dicom-sr/` - DICOM SR support
- `dicom-pdf/` - DICOM PDF support
- `dicom-video/` - DICOM Video support
- `measurement-tracking/` - Measurement tracking support
- `default/` - Standard OHIF functionality

### Service-Oriented Design (PUB-SUB)

The app uses a Services Manager pattern with these core services:
- **Display Set Service**: Manages image series organization
- **Measurement Service**: Handles annotations and measurements
- **Hanging Protocol Service**: Controls image layout and display rules
- **UI Service**: Manages panels, modals, and notifications
- **Segmentation Service**: AI/ML powered image segmentation, loading segmentations, etc.
- **Viewport Grid Service**: Manages viewport layout and display rules
- **Viewport Display Set History Service**: Manages viewport display set history
- **Viewport Dialog Service**: Manages viewport dialogs
- **Notification Service**: Manages notifications
- **Modal Service**: Manages modals
- **Dialog Service**: Manages dialogs, more general not just viewport dialogs
- **Customization Service**: Manages customization of the app
- **Toolbar Service**: Manages the toolbar, viewport action corners, tool states
- **User Authentication Service**: Manages user authentication, but used only for injecting tokens in dicomweb requests in our context
- **Panel Service**: Manages side panels
- **Cornerstone Viewport Service**: Manages the cornerstone viewport, rendering engines, presentation states, more tightly coupled to cornerstone than the other services
- **Tool Group Service**: Manages tool groups, creating and managing tool groups, etc.
- **Sync Group Service**: Manages sync groups, syncing zooming, panning, scrolling, etc.
- **Cornerstone Cache Service**: Manages the cornerstone cache, caching images, etc.

Most of the services utilize a pub sub architecture and extend the pub sub service interace at `pubSubServiceInterface.ts`

### Commands Manager

The Commands Manager tracks named commands (or functions) that are scoped to
a context. When we attempt to run a command with a given name, we look for it
in our active contexts, in the order specified.
If found, we run the command, passing in any application
or call specific data specified in the command's definition.

You can call `commandsManager.runCommand` to run a command.

### Extension Manager

Aggregates and exposes extension modules throughout the OHIF application, manages data sources, and provides a centralized registry for accessing extension functionality.

### Build System

**Yarn Workspaces**: Optimized monorepo builds with dependency caching
**Webpack 5**: Module federation for dynamic extension loading
**Plugin Import System**: Extensions auto-register via `writePluginImportsFile.js`

### Key Technologies

- **React 18 + TypeScript**: UI framework
- **Cornerstone.js**: Medical image rendering
- **DICOM**: Medical imaging standard support
- **ONNX Runtime**: AI model inference (SAM segmentation models)
- **Zustand**: State management
- **TailwindCSS**: Styling system

## Development Patterns

### Adding New Tools
1. Create tool class in `extensions/cornerstone/src/tools/`
2. Register in tool module's `toolNames.ts`
3. Add to toolbar via `getToolbarModule.tsx`
4. Add measurement mapping if needed in `measurementServiceMappings/`

### Creating Extensions
Extensions must export:
- `id.js` - Unique extension identifier
- `index.tsx` - Extension registration
- Module functions (`getToolbarModule`, `getViewportModule`, etc.)

### Viewport Customization
Custom viewports extend base Cornerstone viewport:
- Override render methods for custom overlays
- Implement measurement tracking
- Add viewport-specific tools and interactions

### Service Integration
Register services in extension's `servicesManager.registerService` and access via:
```javascript
const { MeasurementService } = servicesManager.services;
```

### Creating stores

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OHIF/Viewers](https://github.com/OHIF/Viewers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
