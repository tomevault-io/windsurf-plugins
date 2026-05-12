---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is **openMindMap** - a modular TypeScript-based community plugin for the Obsidian note-taking application that renders markdown files as interactive mind maps using D3.js. The plugin automatically detects files starting with `#mindmap` and replaces the standard markdown editor with a hierarchical D3-based visualization.

**Current Version**: 1.0.0
**Main Language**: TypeScript 5.9.3
**Target**: Obsidian API (minimum app version 0.15.0)
**Key Features**: AI-powered node suggestions, mobile support, AES-256 API key encryption

## Development Commands

### Core Development Workflow
```bash
npm run dev              # Development mode with watch for live reloading
npm run build            # Production build with TypeScript validation + esbuild bundling
npm run rebuild          # Clean rebuild using rebuild.sh script
npm run version          # Automated version bump (updates manifest.json and versions.json)
```

### Testing Commands
```bash
npm run build:core       # Build core modules only (for testing)
npm run test             # Run overlap verification tests (requires build:core)
npm run test:unit        # Run unit tests (requires build:core)
```

### Build Process
The build system uses a two-step process:
1. **TypeScript Validation**: `tsc -noEmit -skipLibCheck` for type checking
2. **Esbuild Bundling**: Bundles `src/main.ts` into `main.js` with proper externals

Key configuration files:
- `esbuild.config.mjs` - Main build configuration
- `esbuild.test.config.mjs` - Test build configuration
- `tsconfig.json` - TypeScript compiler configuration

### Development Notes
- Modular architecture with separated services, handlers, and renderers
- Development mode watches for changes and rebuilds automatically
- Built `main.js` is loaded directly by Obsidian during development
- Source maps are generated in development mode (disabled in production)
- Configuration system supports desktop and mobile devices through early branching pattern

## Architecture Overview

### Device Detection Strategy
The plugin implements an **early branching pattern** for device support:
1. Device detection happens once during plugin initialization (`main.ts:onload()`)
2. `ConfigManager` selects either `DesktopConfig` or `MobileConfig` based on device type
3. Configuration is immutable after creation - no runtime device checking
4. Ensures desktop and mobile code paths never mix
5. Respects user setting: can override auto-detection with explicit 'desktop' or 'mobile' choice

### Modular Plugin Structure
The plugin follows a modular service-oriented architecture with clear separation of concerns:

```
MindMapPlugin (extends Plugin)
├── Plugin lifecycle management
├── ConfigManager (device detection & config selection)
├── MindMapService (central coordination layer)
├── AIClient (AI-powered node suggestions)
├── View registration (MIND_MAP_VIEW_TYPE)
├── Command palette integration
└── Settings management with encryption

MindMapView (extends ItemView)
├── View state management (getState/setState)
├── RendererManager (desktop/mobile renderer selection)
├── InteractionManager (desktop/mobile interaction selection)
└── Theme integration

Configuration Layer
├── ConfigManager (central config access)
├── DesktopConfig (desktop-specific settings)
└── MobileConfig (mobile-specific settings)

Service Layer
├── MindMapService (central coordination)
├── D3FileHandler (file operations)
├── LayoutCalculator (position calculations)
└── StateHandler (view state management)

Handler Layer
├── D3FileHandler (file I/O and markdown parsing)
├── InteractionHandler (base interaction handling)
├── DesktopInteraction (desktop-specific interactions)
├── MobileInteraction (mobile-specific interactions)
└── StateHandler (state persistence)

Renderer Layer
├── RendererManager (renderer selection based on device)
├── DesktopTreeRenderer (desktop D3.js visualization)
├── MobileTreeRenderer (mobile D3.js visualization)
├── NodeRenderer (individual node rendering)
└── LayoutCalculator (position and layout calculations)

Utility Layer
├── AIClient (OpenAI-compatible API client)
├── EncryptionUtil (AES-256 API key encryption)
├── AIPrompts (AI prompt template management)
├── FontSizeManager (dynamic font sizing)
└── CoordinateSystem (spatial calculations)
```

### Core Components

#### 1. Plugin Class (`MindMapPlugin`)
Located in `src/main.ts`
- **Device Detection**: Single point of device type determination (respects user setting or auto-detects)
- **Configuration Management**: Creates `ConfigManager` with detected device type
- **Service Coordination**: Initializes `MindMapService` with config and AI client
- **AI Integration**: Manages `AIClient` with encrypted API key storage
- **File Detection**: Listens for `file-open` events to detect `#mindmap` identifier
- **View Replacement**: Uses `setViewState()` to replace markdown editor with mind map
- **Command Integration**: Ribbon icon and command palette access
- **Settings Management**: Handles plugin settings with API key encryption/decryption
- **Lifecycle Management**: Proper load/unload handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hanzzh/openMindMap](https://github.com/Hanzzh/openMindMap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
