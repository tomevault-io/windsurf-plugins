---
trigger: always_on
description: Clustta is a distributed version control and collaboration system for creative workflows (like GitHub for creative work). This is the **desktop client** built with **Wails v3** (Go backend + Vue 3 frontend).
---

# Clustta Client - AI Coding Instructions

## Project Overview
Clustta is a distributed version control and collaboration system for creative workflows (like GitHub for creative work). This is the **desktop client** built with **Wails v3** (Go backend + Vue 3 frontend).

## Architecture

### Technology Stack
- **Backend**: Go 1.25+ with Wails v3 framework
- **Frontend**: Vue 3 + Vite + Pinia stores
- **Database**: SQLite (per-project `.clst` files)
- **Serialization**: Protocol Buffers for efficient data transmission

### Key Directories
```
services/          # Go services exposed to frontend via Wails bindings
internal/          # Core Go business logic (not directly exposed)
  repository/      # Database models and operations (SQLite)
  auth_service/    # Authentication handling
  sync_service/    # Sync/collaboration logic
frontend/src/
  services/        # Service abstraction layer (Wails bindings or HTTP)
  stores/          # Pinia state management
  instances/       # Platform-specific UI (desktop/, web/, common/)
  lib/             # Utilities including mitt event bus
```

### Service Layer Pattern
Go services in `services/` are registered in `main.go` and auto-generate TypeScript bindings:
```go
// services/asset_services.go - Backend service
func (t *AssetService) GetAssetByID(projectPath, assetId string) (models.Task, error)

// Frontend usage via auto-generated bindings
import { AssetService } from '@/services';
await AssetService.GetAssetByID(projectUri, id);
```

Frontend services are abstracted in `frontend/src/services/index.js` to support both:
- **Desktop (Wails)**: Direct Go bindings via `adapters/wails.js`
- **Web mode**: HTTP REST calls via `adapters/http.js` (set `VITE_PLATFORM=web`)

### Event Communication
- **Go → Frontend**: `app.Event.Emit("event-name", data)` (Wails events)
- **Frontend listens**: `Events.On("event-name", callback)` from `@wailsio/runtime`
- **Frontend internal**: `emitter.emit()` via mitt (`@/lib/mitt`)

Common events: `progress-update`, `fs-change`, `sync-project`, `refresh-browser`

### External Server Integration
Clustta connects to two external servers for collaboration:
- **[clustta-server](https://github.com/eaxum/clustta-server)** - Global authentication server
- **[clustta-studio](https://github.com/eaxum/clustta-studio)** - Studio/team management server

For local development with these servers, set build flags in `build/platform/Taskfile.yml`:
```bash
-ldflags="-X clustta/internal/constants.host=http://127.0.0.1:5000"
```

## Development Commands

```bash
# Run development client (primary command)
make client           # or: wails3 dev

# Build for production
make build            # Creates MSIX (Windows) or Mac App Store build

# Install dev dependencies
make install-dev      # Installs wails3 CLI

# Run tests
go test ./...

# Protocol buffer regeneration (after editing schema.proto)
protoc --go_out=. .\internal\repository\schema.proto
pbjs -t static-module -w es6 --keep-case -o .\frontend\src\lib\repositorypb.js .\internal\repository\schema.proto
```

## Core Domain Concepts

| Term | Meaning |
|------|---------|
| **Entity/Collection** | Folder-like container for organizing assets |
| **Task/Asset** | Individual file being version-controlled |
| **Checkpoint** | A saved version/snapshot of assets |
| **Project** | A `.clst` SQLite database containing all metadata |
| **Studio** | Team/organization server for collaboration |

## Code Style Conventions

### Go Services
- One or two line comments preceding functions (no inline comments except at major blocks)
- See `services/collection_service.go` for reference pattern:
```go
// GetCollectionCount returns the total number of collections in the project.
// Returns the count or an error if the operation fails.
func (t *CollectionService) GetCollectionCount(projectPath string) (int, error) {
```

### Vue Components (`<script setup>`)
Organize sections in this order with comment headers (see `frontend/src/assets/boilerplate.js`):
```javascript
// imports
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';
import emitter from '@/lib/mitt';

// components
import ActionButton from '@/instances/desktop/components/ActionButton.vue';
import GridView from '@/instances/desktop/components/GridView.vue';
import PageState from '@/instances/common/components/PageState.vue';

// services
import { AssetService, CollectionService, SyncService } from '@/services';

// stores (alphabetically)
const assetStore = useAssetStore();
const collectionStore = useCollectionStore();
const iconStore = useIconStore();
const notificationStore = useNotificationStore();
const projectStore = useProjectStore();

// refs (alphabetically)
const browserRoot = ref(null);
const isLoading = ref(false);
const searchQuery = ref('');

// computed properties (alphabetically, but dependencies first)
const canModifyEntity = computed(() => { ... });
const filteredAssets = computed(() => { ... });
const isTasksModified = computed(() => { ... });

// Note: If a computed depends on another computed, the dependency must come first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eaxum/clustta-client](https://github.com/eaxum/clustta-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
