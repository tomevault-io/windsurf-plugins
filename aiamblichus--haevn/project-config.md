---
trigger: always_on
description: **HAEVN** (a psychopomp for digital consciousness) is a Chrome Manifest V3 extension that syncs AI conversations from multiple LLM platforms into a unified local archive. It preserves the full context of your interactions—including branching conversations, multi-modal content, and metadata—using a canonical `HAEVN.Chat` format.
---

# What This Project Is About

**HAEVN** (a psychopomp for digital consciousness) is a Chrome Manifest V3 extension that syncs AI conversations from multiple LLM platforms into a unified local archive. It preserves the full context of your interactions—including branching conversations, multi-modal content, and metadata—using a canonical `HAEVN.Chat` format.

**Core Value:** Own your AI conversation history. Search across platforms. Export for backup. Never lose context.

**Supported Platforms:** Claude, ChatGPT, Gemini, Poe, Open WebUI, Qwen, DeepSeek, AI Studio, Claude Code (import-only), Grok

---

## Core Technologies

- **Language:** TypeScript 5.9.3 (strict mode, ~33k lines)
- **Build:** esbuild (multi-entry bundling, 30s full rebuild)
- **Runtime:** Chrome Extension MV3 (Service Worker environment)
- **Frontend:** React 18.3, Tailwind CSS, shadcn/ui, Radix UI primitives
- **Persistence:** Dexie (IndexedDB) + OPFS (Origin Private File System for media)
- **State:** Zustand (UI state), event-driven architecture (chrome.runtime messaging)
- **Search:** Lunr.js (full-text indexing in Web Worker)
- **Testing:** Vitest, fake-indexeddb
- **Linting:** Biome 2.3.7

---

## Architectural Principles

### 1. Three-Tier Worker Architecture

Chrome MV3 service workers cannot create Web Workers. Solution: Service Worker → Offscreen Document → Web Workers. This pattern enables CPU-intensive operations (search, stats, import/export) without blocking the service worker.

### 2. Data Transformation Pipeline

Raw platform data → Extractor (fetch) → Transformer (normalize) → `HAEVN.Chat` format → Persistence → Indexing → Export. Single canonical format eliminates N×M format conversions.

### 3. Provider Abstraction

Plugin architecture: Each platform implements `Extractor<TRaw>` and `Transformer<TRaw>` interfaces. Core sync logic is platform-agnostic. Adding new platforms requires no changes to orchestration or persistence layers.

### 4. Separation of Concerns

- **Handlers**: Route messages, orchestrate
- **Services**: CRUD, caching, indexing
- **Providers**: Platform-specific extraction/transformation
- **Workers**: Heavy computation
- **UI**: Display, user interaction

### 5. Soft Delete with Deferred Cleanup

Instant UX via soft-delete flag, expensive cleanup happens in background (Janitor service runs every 30 minutes). Compound indexes filter deleted items at query time.

### 6. Event-Driven UI Updates

Background service worker is source of truth. UI components subscribe to events (`chatSynced`, `bulkSync*`, `importProgress`) for real-time updates. No centralized state management needed.

---

## Guild Conventions

### Code Style (enforced by Biome)

- 2-space indentation, 100 character line width
- Double quotes, trailing commas
- camelCase for functions/variables, PascalCase for components/types
- `fireAndForget()` for non-critical async operations (not bare `.catch(() => {})`)

### Development Workflow

```bash
pnpm install              # Install dependencies
pnpm run build            # Build extension → dist/
curl -X POST http://localhost:5556/command -d '{"action": "reload"}' # Reload extension via proxy
pnpm test                 # Run tests
pnpm run lint             # Check code (currently 51 errors, 55 warnings)
pnpm run lint:fix         # Auto-fix linting issues
```

### Architecture Rules

- **All persistence through `SyncService`** - Never use Dexie directly from handlers
- **All indexing via search worker** - SyncService proxies to worker
- **Heavy operations in workers** - ZIP generation, indexing, stats calculation
- **Browser APIs via service worker** - Workers use bridge pattern (CRD-003)
- **Downloads via `downloadFile` handler** - No blob URLs in UI components
- **Emit events** for real-time UI updates (chatSynced, bulkSync*, import*)

### Testing Standards

- Unit tests for pure functions (extractors, transformers, utilities)
- Integration tests for service workflows (sync, search, import/export)
- Use `fake-indexeddb` for Dexie operations (see `tests/setup.ts`)

---

## System Structure

### Components (7 major subsystems)

1. **Message Router** (`background/handlers/`) - Type-safe message dispatch (82 handlers, 767 lines of message types)
2. **Provider Abstraction** (`providers/`) - 10 platform plugins (API fetch or DOM extraction)
3. **Persistence Layer** (`services/`) - ChatRepository + SearchIndexManager + SearchService
4. **Worker Pool** (`offscreen/`) - 6 specialized workers (search, stats, thumbnail, bulk ops)
5. **Bulk Operations** (`background/bulkSync/`, `bulkExport/`, `import/`) - Stateful orchestration
6. **UI Layer** (`popup/`, `options/`, `viewer/`) - 3 browser contexts (popup, archive, viewer)
7. **Content Script** (`content/content.ts`) - Injected bridge to LLM platforms

### Data Flows

**Single Chat Sync:**
User → Popup → Handler → Content Script → Extractor → Transformer → Repository → Index → Event

**Bulk Sync:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aiamblichus/haevn](https://github.com/aiamblichus/haevn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
