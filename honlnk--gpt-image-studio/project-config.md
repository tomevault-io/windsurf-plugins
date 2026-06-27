---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev                  # Start Vite dev server (127.0.0.1:8888)
pnpm dev:companion        # Start Companion CLI server (127.0.0.1:19750)
pnpm dev:desktop          # Start Tauri dev shell (webview + hot reload; needs Rust)
pnpm build:desktop        # Build desktop app to .app/.dmg (needs Rust)
pnpm build                # Production build to dist/
pnpm preview              # Preview production build
pnpm typecheck            # Type-check web app with vue-tsc --noEmit
pnpm typecheck:companion  # Type-check companion with tsc --noEmit
pnpm test                 # Run Vitest tests (all)
pnpm test:watch           # Run Vitest in watch mode
```

Run a single test file: `pnpm vitest run src/services/backups.test.ts`

No linter or formatter is configured. Vitest is configured for service-level tests.

## Architecture

Local-first AI image creation workbench. Vue 3 + Composition API (`<script setup>`), no router. Only runtime dependency beyond Vue is Pinia for state management. ZIP creation/reading, base64 conversion, image dimension reading, and storage usage estimation are all hand-written (no external libs).

See `docs/README.md` for the maintained documentation map and `docs/architecture.md` for the current architecture direction.

### State Management

Pinia stores in `src/stores/` manage cross-component shared state by domain (`settingsStore`, `composerStore`, `imagesStore`, `conversationsStore`, `generationStore`, `feedbackStore`, `analyticsStore`).

`src/app/studio/useStudioViewModel.ts` is the page-level orchestration layer: it coordinates across stores for workflows like draft switching, backup/restore, and preview. `App.vue` calls it once and distributes state/methods to children via props and events.

Feature composables in `src/features/*/` serve as compatibility wrappers bridging the old composable API to the new Pinia stores.

**Hydration**: On mount, stores load all data from IndexedDB into memory refs. All subsequent mutations happen in memory first, then async-persist to IndexedDB via the `services/` layer.

**Dual storage**: Lightweight drafts (`composerText`, `attachedImages`, API config) go to localStorage. Everything else (conversations, messages, image assets, image blobs, settings) goes to IndexedDB.

### Data Flow

```
User action → Component emit → App.vue handler → useStudioViewModel method
  → Pinia store action (reactive UI update) → async persist to services/ → IndexedDB
```

### Component Organization

```
src/components/
  studio/       # Page-level layout shells (ChatWorkspace, ConversationSidebar, ImageLibrary, modals)
  chat/         # Chat composer, message list, parameter bar, mask editor
  image-library/# Image grid, card, details panel, storage usage
  settings/     # API config, backup, batch operations panels
  ui/           # Generic reusable: Tooltip, ConfirmDialog, NoticeToast, RenameDialog
```

### Service Layer (`src/services/`)

All IndexedDB access goes through `db.ts` (generic CRUD: `getAllFromStore`, `getFromStore`, `putInStore`, `deleteFromStore`). Domain services build on top:

| File | Purpose |
|------|---------|
| `conversations.ts` | List/save/delete conversations |
| `messages.ts` | List/save/delete messages |
| `imageAssets.ts` | Image metadata + blob CRUD (separate stores) |
| `settings.ts` | Single-record app settings |
| `imagesApi.ts` | OpenAI-compatible image generation (`/generations`) and editing (`/edits`) API calls |
| `conversationDrafts.ts` | Per-conversation draft persistence (localStorage) |
| `generationParams.ts` | Generation parameter validation and defaults |
| `imageMetadata.ts` | Read image dimensions via `createImageBitmap` / `HTMLImageElement` |
| `storageUsage.ts` | Estimate IndexedDB usage via `navigator.storage.estimate()` |
| `backups.ts` | Full project export/import as ZIP |
| `analyticsEvents.ts` | Analytics event persistence + JSONL export (excluded from backups) |
| `analyticsExport.ts` | Analytics ZIP export (manifest + JSONL + Markdown timeline shards) |
| `zipArchive.ts` | Hand-written ZIP file creator (CRC32 + binary format) |

### Generation / Image Client

`src/features/generation/imageClients/imageClient.ts` defines the `ImageClient` interface (`generate` + `edit`). Two implementations:
- `directImagesClient` — browser calls user-configured OpenAI-compatible Images API directly.
- `localCompanionImagesClient` — browser calls a paired local companion service on `127.0.0.1`.

### Types

All business types in `src/types/studio.ts`: `Conversation`, `Message`, `ImageAsset`, `GenerationParams`, `AppSettings`, plus union type aliases.

### Key Patterns

- **Image storage**: Metadata (`imageAssets` store) and binary data (`imageBlobs` store) are separated. `ImageAsset.blobKey` links them. `previewUrl` (`URL.createObjectURL`) is memory-only — created during hydration, stripped before persist via `toPlainImageAsset`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [honlnk/gpt-image-studio](https://github.com/honlnk/gpt-image-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
