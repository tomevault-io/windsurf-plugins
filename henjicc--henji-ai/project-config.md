---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Henji-AI (痕迹AI) is a Tauri-based desktop application that aggregates multiple AI providers (PPIO, Fal, ModelScope, KIE) for generating images, videos, and audio through a unified interface. The application uses an adapter pattern to abstract provider-specific APIs.

## Development Commands

### Frontend Development
```bash
npm install              # Install dependencies
npm run dev              # Vite dev server only
npm run build            # TypeScript compilation + Vite build
npm run preview          # Preview production build
npm run lint             # Run ESLint
npm run lint:fix         # Auto-fix ESLint issues
```

### Tauri Development
```bash
# Windows (requires Visual Studio Build Tools)
npm run tauri:dev        # Development mode with hot reload
npm run tauri:build      # Production build (generates MSI)
npm run tauri:build:ci   # CI build without VS environment setup

# macOS (requires Xcode Command Line Tools)
npm run tauri:dev:mac    # Development mode
npm run tauri:build:mac  # Production build (generates DMG)
```

Build artifacts are located in `src-tauri/target/release/bundle/`

## Architecture

### Adapter Pattern Implementation

The core architecture uses a **Factory + Strategy pattern** to integrate multiple AI providers:

```
MediaGenerator (UI)
    ↓
ApiService (Singleton)
    ↓
AdapterFactory.createAdapter(config)
    ↓
BaseAdapter (Abstract)
    ↓
├── PPIOAdapter
├── FalAdapter
├── KIEAdapter
└── ModelscopeAdapter
```

**Key Files:**
- `src/adapters/base/BaseAdapter.ts` - Abstract base class defining the adapter interface
- `src/adapters/index.ts` - `AdapterFactory` with `createAdapter()` method
- `src/services/api.ts` - `ApiService` singleton managing adapter lifecycle

### Model Routing System

Each adapter implements a **route-based model system** that maps model IDs to request builders:

```typescript
interface ModelRoute {
  matches: (modelId: string) => boolean
  buildImageRequest?: (params) => { endpoint, requestData }
  buildVideoRequest?: (params) => { endpoint, requestData }
  buildAudioRequest?: (params) => { endpoint, requestData }
}
```

**Route Registration:**
- `src/adapters/ppio/models/index.ts` - PPIO model routes
- `src/adapters/fal/models/index.ts` - Fal model routes
- `src/adapters/kie/models/index.ts` - KIE model routes
- `src/adapters/modelscope/models/index.ts` - ModelScope model routes

**Flow:** `adapter.generateImage()` → `findRoute(modelId)` → `route.buildImageRequest()` → API call

### Provider-Specific Implementations

**PPIO Adapter** (`src/adapters/ppio/PPIOAdapter.ts`)
- Uses Axios for HTTP requests
- Polling via `PPIOStatusHandler` class
- Config: `src/adapters/ppio/config.ts` (base URL, poll interval: 3000ms, max attempts: 120)

**Fal Adapter** (`src/adapters/fal/FalAdapter.ts`)
- Uses official `@fal-ai/client` SDK
- Automatic polling via `fal.subscribe()`
- Handles image/video uploads to Fal CDN
- Config: `src/adapters/fal/config.ts` (model-specific poll counts)

**KIE Adapter** (`src/adapters/kie/KIEAdapter.ts`)
- Uses Axios with separate upload client
- Uploads images to KIE CDN before processing
- Status mapping: waiting → QUEUED, generating → PROCESSING, success → COMPLETED
- Config: `src/adapters/kie/config.ts` (poll interval: 3000ms, max attempts: 200)

**ModelScope Adapter** (`src/adapters/modelscope/ModelscopeAdapter.ts`)
- Uses Tauri backend via `invoke()` for API calls
- Optional Fal CDN integration for image uploads
- Currently supports image generation only

### Configuration System

**Provider Registry** (`src/config/providers.ts`)
- Loads from `providers.json`
- Defines provider metadata and available models
- Structure: `Provider { id, name, type, models[] }`

**Model Parameter System** (`src/models/index.ts`)
- Centralized `modelSchemaMap` mapping model IDs to parameter schemas
- Key functions:
  - `getModelSchema(modelId)` - Get parameter schema
  - `getModelDefaultValues(modelId)` - Extract default values
  - `getAutoSwitchValues(modelId, currentValues)` - Conditional parameter switching
  - `getSmartMatchValues(modelId, imageDataUrl, currentValues)` - Intelligent aspect ratio matching

**Model Parameter Files:**
- `src/models/ppio/` - PPIO model parameters
- `src/models/fal/` - Fal model parameters
- `src/models/modelscope/` - ModelScope model parameters
- `src/models/kie/` - KIE model parameters

### Response Parsing

Each adapter has provider-specific parsers:
- `src/adapters/ppio/parsers/index.ts` - PPIO response parsing
- `src/adapters/fal/parsers/imageParser.ts` - Fal response parsing
- `src/adapters/kie/parsers/` - KIE response parsing

### Data Storage

- **API Keys:** localStorage
- **History:** AppLocalData (`Henji-AI/history.json`)
- **Media Files:** AppLocalData (`Henji-AI/Media/`)
- **Cache:** AppLocalData (`Henji-AI/Uploads/`, `Henji-AI/Waveforms/`)

## Adding New AI Models or Providers

See **[docs/model-adaptation-guide.md](docs/model-adaptation-guide.md)** for detailed instructions on:
1. Defining model parameter schemas
2. Implementing adapter routes
3. Registering models in the system

### Quick Steps:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henjicc/Henji-AI](https://github.com/henjicc/Henji-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
