---
trigger: always_on
description: - **Multiple Directory Support**: Add and manage multiple image directories simultaneously.
---

# AI **Recent Changes (v1.9.0):**
- **Multiple Directory Support**: Add and manage multiple image directories simultaneously.
- **Settings Management**: New modal for cache and update preferences.
- **Resizable Grid**: Dynamic thumbnail sizing for any display.
- **Command-Line Support**: Launch with specified directory for automation.
- **Dev Server Access**: Network-accessible development server.
- **Path Fixes**: Improved cross-platform path handling and file operations.Assistant Instructions for Image MetaHub

## Project Overview
This is a React + TypeScript + Electron application that provides local browsing and filtering of AI-generated images. The app runs in both web browsers and as a desktop application, with dual file system APIs for cross-platform compatibility.

**Recent Changes (v1.9.0):**
- **📂 Multiple Directory Support**: Add and manage multiple image directories simultaneously.
- **⚙️ Settings Management**: New modal for cache and update preferences.
- **�️ Resizable Grid**: Dynamic thumbnail sizing for any display.
- **� Command-Line Support**: Launch with specified directory for automation.
- **🌐 Dev Server Access**: Network-accessible development server.
- **� Path Fixes**: Improved cross-platform path handling and file operations.

## Architecture & Data Flow

### Core Architecture
- **Frontend**: React 18 + TypeScript + Vite build system
- **State Management**: Zustand store (useImageStore.ts) for centralized state management
- **Desktop**: Electron wrapper with auto-updater
- **Storage**: IndexedDB for client-side caching, localStorage for UI preferences
- **File Access**: File System Access API (browser) + Node.js fs APIs (Electron)

### Key Components
- `App.tsx` - Main application orchestrator (minimal, focused on composition)
- `store/useImageStore.ts` - Centralized Zustand store for all application state
- `hooks/useImageLoader.ts` - Directory loading and automatic filter extraction
- `hooks/useImageFilters.ts` - Search and filtering logic
- `hooks/useImageSelection.ts` - Image selection management
- `components/` - Modular UI components (Header, StatusBar, ActionToolbar, ImageGrid, SearchBar, ImageModal, Sidebar, BrowserCompatibilityWarning, etc.)
- `services/` - Business logic (fileIndexer, cacheManager, fileOperations)
- `services/parsers/` - Modular metadata parsers for InvokeAI, A1111, ComfyUI, SwarmUI, Easy Diffusion, and Midjourney formats (PNG + JSON sidecar)
- `types.ts` - TypeScript interfaces for InvokeAI metadata and file handles

### Data Flow Patterns
1. **Directory Selection** → Environment detection (browser vs Electron)
2. **File Indexing** → PNG metadata extraction → IndexedDB caching
3. **Automatic Filter Extraction** → Models, LoRAs, and schedulers extracted from loaded images
4. **Search/Filter** → In-memory filtering with word-boundary matching
5. **File Operations** → IPC communication for Electron file management

### Architecture Refactoring (2025-09-29)
The application underwent a major refactoring to improve modularity and LLM-friendliness:

**State Management Migration**: All component state (useState) was migrated to a centralized Zustand store (`useImageStore.ts`). This consolidates state logic and makes it easier to manage and understand.

**Custom Hooks Extraction**: Logic was extracted from the monolithic `App.tsx` into focused custom hooks:
- `useImageLoader.ts` - Directory loading and automatic filter extraction
- `useImageFilters.ts` - Search and filtering logic  
- `useImageSelection.ts` - Image selection management

**Component Modularization**: UI elements were broken out into dedicated components:
- `Header.tsx` - Application header
- `StatusBar.tsx` - Status information display
- `ActionToolbar.tsx` - Action buttons and controls

**Parser Modularization**: The monolithic `fileIndexer.ts` was broken down into smaller, focused parser modules:
- `services/parsers/` - Modular metadata parsers for InvokeAI, A1111, ComfyUI, and SwarmUI formats
- Factory pattern for automatic parser selection based on metadata format

This architecture transformation significantly improves code maintainability, testability, and LLM comprehension.

## ComfyUI Parser Architecture

### Rule-Based Parsing System
The ComfyUI parser uses a declarative, rule-based architecture for extracting metadata from complex workflow graphs:

**Core Components:**
- **`nodeRegistry.ts`** - Declarative registry of ComfyUI node behaviors and parameter mappings
- **`traversalEngine.ts`** - Graph traversal logic with multi-path exploration and parameter extraction
- **`comfyUIParser.ts`** - High-level parsing orchestration and terminal node detection

**Node Registry Pattern:**
```typescript
export const NodeRegistry: Record<string, NodeDefinition> = {
  'KSampler (Efficient)': {
    category: 'SAMPLING',
    roles: ['SINK'],
    inputs: { model: { type: 'MODEL' }, positive: { type: 'CONDITIONING' } },
    outputs: { LATENT: { type: 'LATENT' } },
    param_mapping: {
      seed: { source: 'trace', input: 'seed' },
      steps: { source: 'widget', key: 'steps' },
      cfg: { source: 'widget', key: 'cfg' },
      sampler_name: { source: 'widget', key: 'sampler_name' },
      prompt: { source: 'trace', input: 'positive' }
    },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuqP2/Image-MetaHub](https://github.com/LuqP2/Image-MetaHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
