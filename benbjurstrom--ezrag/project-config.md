---
trigger: always_on
description: **New to this codebase?** Start here:
---

# EzRAG - Obsidian Plugin for Semantic Search via Google Gemini

## Quick Start for Developers

**New to this codebase?** Start here:

1. **Read this file** for high-level overview and module guide
2. **Read ARCHITECTURE.md** for detailed design, data models, and implementation notes
3. **Key entry points**:
   - `main.ts` - Plugin lifecycle (start here to understand initialization)
   - `src/lifecycle/indexingLifecycleCoordinator.ts` - Centralized runner/connection gating & store provisioning
   - `src/indexing/indexingController.ts` - Indexing lifecycle management
   - `src/indexing/indexManager.ts` - Core indexing logic
   - `src/indexing/filePreparationService.ts` / `documentMetadata.ts` / `documentReplacer.ts` - Shared file ingestion helpers
   - `src/indexing/persistentQueue.ts` - Queue orchestration, retries, and connection-aware scheduling
   - `src/gemini/geminiService.ts` - Gemini API integration

4. **Build and test**:
   ```bash
   npm install
   npm run dev  # Watch mode for development
   ```

5. **Critical concepts** to understand:
   - **Runner pattern**: Only one machine indexes per vault
   - **Hot path optimization**: No remote checks during file changes
   - **Queue persistence**: Uploads survive restarts
   - **Smart reconciliation**: Rebuild doesn't create duplicates

## What is EzRAG?

EzRAG is an Obsidian plugin that indexes your notes into Google Gemini's File Search API, enabling semantic search and AI-powered chat over your vault. Key features:

1. **Automatic Indexing**: Continuously syncs selected notes to Gemini as you edit
2. **Smart Change Detection**: Uses content hashing to avoid redundant uploads
3. **Multi-Device Support**: "Runner" pattern designates one machine to handle indexing
4. **Chat Interface**: Query your notes using natural language
5. **MCP Server** (planned): External tools can query your vault via Model Context Protocol

## How It Works

### The Runner Pattern (Critical Concept)

In multi-device setups (laptop + desktop), **only one machine** (the "runner") handles indexing:

- **Runner machine**: Monitors vault changes, uploads to Gemini, keeps index in sync
- **Non-runner machines**: Can query/chat but don't perform indexing
- **Runner state**: Stored in browser `localStorage` (per-machine, non-synced)
- **Plugin settings**: API key syncs via vault data, but runner status stays local

This prevents race conditions and duplicate documents when the same vault is open on multiple devices.

### Key Architectural Concepts

1. **Vault-centric identity**: Obsidian paths are primary identifiers
2. **Content-based change detection**: SHA-256 hashes detect real changes
3. **Delete-then-recreate**: Work with Gemini's immutable document model
4. **Metadata-driven mapping**: Store identity in Gemini customMetadata
5. **Queue persistence**: Uploads survive restarts/crashes via persisted queue
6. **Throttling**: Configurable debounce prevents rapid-fire uploads

## Environment & tooling

- Node.js: use current LTS (Node 18+ recommended).
- **Package manager: npm** (required - `package.json` defines npm scripts and dependencies).
- **Bundler: esbuild** (required - `esbuild.config.mjs` and build scripts depend on it).
- Types: `obsidian` type definitions + `@google/genai` SDK.
- **Desktop-only indexing**: Uses Node.js `crypto` module (mobile can query but not index)

## Module Guide: Where to Find Things

### Core Entry Points

- **`main.ts`**: Plugin lifecycle, event registration, command setup
  - Creates `IndexingController`, `IndexingLifecycleCoordinator`, and `StoreManager`
  - Registers vault events (after `onLayoutReady()` to prevent startup flooding)
  - Adds runner-only commands (rebuild-index, cleanup-orphans, run-janitor)
  - Delegates status bar updates to lifecycle coordinator/controller signals

- **`src/lifecycle/indexingLifecycleCoordinator.ts`**
  - Owns runner + platform gating, API-key validation, and Gemini store provisioning
  - Listens to `ConnectionManager` and pauses/resumes `IndexingController`
  - Provides `requireConnection` helper for commands/settings flows

### Runner Management

- **`src/runner/runnerState.ts`**: Per-machine runner state (localStorage-based)
  - `RunnerStateManager`: Check/set runner status for this device
  - Storage key: `ezrag.runner.<pluginId>.<vaultKey>` (vault-isolated)
  - Used to gate all indexing operations

### State & Persistence

- **`src/state/state.ts`**: Core state management (Obsidian-agnostic)
  - `StateManager`: Persisted data, indexed document tracking, queue entries
  - Can be reused by MCP server
- **`src/storage/indexStateStorageManager.ts`**: Device-local persistence layer
  - Loads/saves the `StateManager` index snapshot to `window.localStorage`
  - Ensures only the runner caches docs/queue data while other devices can rebuild as needed
- **`src/types.ts`**: All TypeScript interfaces
  - `PersistedData`, `PluginSettings`, `IndexState`, `IndexedDocState`, `IndexQueueEntry`

### Indexing Engine

- **`src/indexing/indexingController.ts`**: Lifecycle management
  - Start/stop/pause/resume indexing
  - Phase tracking (idle/scanning/indexing/paused)
  - State persistence coordination (debounced saves)
  - Event delegation to IndexManager

- **`src/indexing/indexManager.ts`**: Core indexing orchestrator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benbjurstrom/ezrag](https://github.com/benbjurstrom/ezrag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
