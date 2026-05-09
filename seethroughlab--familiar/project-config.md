---
trigger: always_on
description: An LLM-powered local music player that combines library management with AI-powered discovery. Users describe what they want to listen to in natural language, and Codex creates playlists from a deeply-analyzed local music collection.
---

# Familiar

An LLM-powered local music player that combines library management with AI-powered discovery. Users describe what they want to listen to in natural language, and Codex creates playlists from a deeply-analyzed local music collection.

## Architecture

- **Backend**: Python FastAPI + PostgreSQL (pgvector) + Redis
- **Frontend**: React + TypeScript + Vite + Tailwind + Zustand (pnpm workspace monorepo)
- **Analysis**: Audio embeddings and features extracted via librosa/torch
- **LLM**: Codex API with tool-use
- **Offline**: IndexedDB (Dexie) for track caching, download queue, playlist cache

## Key Directories

```
packages/
├── frontend/              # Shared React code (components, hooks, stores, types)
│   └── src/
│       ├── components/    # React components
│       ├── hooks/         # Custom hooks (useFavorites, useAutoDownload, etc.)
│       ├── stores/        # Zustand state stores (playerStore, downloadStore)
│       ├── player/        # Audio engine abstraction, playback hooks
│       ├── services/      # offlineService, playlistCache, syncService, profileService
│       └── db/            # IndexedDB/Dexie storage
├── web/                   # Web entry point + Web Audio engine + PWA
│   ├── src/
│   │   ├── main.tsx       # Registers WebAudioEngine, sets up SW
│   │   └── WebAudioEngine.ts
│   ├── e2e/               # Playwright E2E tests
│   └── vite.config.ts     # PWA plugin, dev proxy, manual chunks
└── ios/                   # Capacitor + native Swift + iOS deploy
    ├── src/
    │   ├── main.tsx       # Registers CapacitorEngine
    │   ├── CapacitorEngine.ts
    │   └── plugins/familiarAudio.ts
    ├── native/            # Xcode project + Swift code
    ├── capacitor.config.ts
    └── scripts/           # deploy-device.sh, release-testflight.sh
backend/
├── app/
│   ├── api/routes/        # FastAPI endpoints (~29 route files)
│   ├── db/models.py       # SQLAlchemy models
│   └── services/          # Business logic
│       └── llm/           # LLM module (service.py, executor.py, tools.py, providers.py)
├── migrations/versions/   # Alembic database migrations
└── tests/                 # pytest tests
```

## Key Files

| Task | Files |
|------|-------|
| Database models | `backend/app/db/models.py` |
| Database migrations | `backend/migrations/versions/*.py` |
| API routes | `backend/app/api/routes/*.py` |
| Audio analysis | `backend/app/services/analysis.py` |
| LLM tools | `backend/app/services/llm/tools.py`, `llm/executor.py` |
| Library scanning | `backend/app/services/scanner.py` |
| Smart playlists | `backend/app/services/smart_playlists.py` |
| Background tasks | `backend/app/services/background.py`, `services/tasks.py` |
| Audio engine abstraction | `packages/frontend/src/player/audio/types.ts`, `createEngine.ts` |
| Audio playback | `packages/frontend/src/player/useAudioEngine.ts` |
| Web Audio engine | `packages/web/src/WebAudioEngine.ts` |
| iOS Audio engine | `packages/ios/src/CapacitorEngine.ts` |
| Player state | `packages/frontend/src/stores/playerStore.ts` |
| Download queue | `packages/frontend/src/stores/downloadStore.ts` |
| Offline storage | `packages/frontend/src/services/offlineService.ts` |
| Playlist caching | `packages/frontend/src/services/playlistCache.ts` |
| Favorites | `packages/frontend/src/hooks/useFavorites.ts` |
| IndexedDB schema | `packages/frontend/src/db/index.ts` |
| Full player | `packages/frontend/src/components/FullPlayer/` |
| Discovery | `packages/frontend/src/components/Discovery/` |
| Smart playlists UI | `packages/frontend/src/components/SmartPlaylists/` |
| Settings | `packages/frontend/src/components/Settings/` |

## Frontend Architecture

The frontend uses a **registration pattern** for platform-specific code. The shared `@familiar/frontend` package has zero `@capacitor` dependencies:

- **`createEngine.ts`** — `registerEngineFactory(fn)` sets the audio engine constructor
- **`api/base.ts`** — `registerPreferencesProvider(p)` for Capacitor Preferences

Each platform entry point (`packages/web/src/main.tsx`, `packages/ios/src/main.tsx`) registers its implementations before calling `renderApp()`.

## Common Tasks

### Add a new audio feature
1. Add extraction logic to `analysis.py` in `extract_features()`
2. No schema change needed (features stored as JSONB)
3. Bump `ANALYSIS_VERSION` in `config.py` to re-analyze existing tracks

### Add a new LLM tool
1. Define tool schema in `MUSIC_TOOLS` list in `services/llm/tools.py`
2. Implement handler in `ToolExecutor` class in `services/llm/executor.py`
3. Tools can query JSONB with PostgreSQL `->` operator

### Add a new API endpoint
1. Create route in `backend/app/api/routes/`
2. Register router in `main.py`
3. Use dependency injection from `deps.py` for DB/auth

### Add a database migration
1. Create file in `backend/migrations/versions/` named `YYYYMMDD_slug.py`
2. **Revision ID must be ≤32 characters** (alembic_version column limit)
3. Always use the `_column_exists()` guard pattern for idempotent migrations:
```python
def _column_exists(table_name: str, column_name: str) -> bool:
    conn = op.get_bind()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seethroughlab/familiar](https://github.com/seethroughlab/familiar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
