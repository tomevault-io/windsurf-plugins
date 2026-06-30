---
trigger: always_on
description: Photofield is a high-performance photo viewer built to display massive photo collections (40k+ images) with seamless zooming and progressive loading. A monolithic Go HTTP server (`main.go` - 2400+ lines) renders photo tiles server-side using the Canvas library, serving them to a Vue.js + OpenLayers frontend. The architecture prioritizes read-only filesystem operations and SQLite caching.
---

# Photofield AI Coding Agent Instructions

## Project Overview
Photofield is a high-performance photo viewer built to display massive photo collections (40k+ images) with seamless zooming and progressive loading. A monolithic Go HTTP server (`main.go` - 2400+ lines) renders photo tiles server-side using the Canvas library, serving them to a Vue.js + OpenLayers frontend. The architecture prioritizes read-only filesystem operations and SQLite caching.

## Critical Architecture Patterns

### Monolithic HTTP Server Design
**`main.go` contains everything** - no separate server package. All HTTP handlers, tile rendering, and business logic live in a single file:
- OpenAPI-generated routes via `//go:generate` directive (line 76)
- Manual route registration using `chi` router
- Direct tile rendering in request handlers (not middleware)
- Global state: `imageSource`, `sceneSource`, `collections`, `globalGeo`

When adding endpoints: implement handler methods on `Api` struct, regenerate OpenAPI code with `task gen`.

### Configuration Hot-Reload System
`config.go` implements **dual filesystem watchers**:
1. Main config file watcher (`configuration.yaml`)
2. Expanded paths watcher (for `expand_subdirs: true` collections)

Changes trigger full reload via callback - don't cache config values in long-lived goroutines.

**Collection expansion**: Single collection definition can spawn multiple collections from subdirectories (see `defaults.yaml` line 4). The `ExpandedPaths` field tracks directories to watch.

### I/O Abstraction Layer (`internal/io/`)
**Composable image sources** - chain modules like middleware:
```go
// Real pattern from codebase - wrapping sources
cached := &Cached{Source: thumb, Cache: ristrettoCache}
```
Each module implements `Source` interface:
- `Get(ctx, id, path) Result` - fetch image
- `Size(original) Size` - calculate target dimensions
- `GetDurationEstimate(size)` - for source selection optimization
- `Exists()`, `Close()` - lifecycle management

**Module composition order matters**:
1. `cached/` - check memory/disk cache first
2. `thumb/` - extract embedded JPEG thumbnails
3. `djpeg/` - efficient JPEG decoding via libjpeg-turbo
4. `goimage/` - fallback Go image decoder

Cost-based source selection: sources compete to provide images, winner determined by `SizeCost()` calculation balancing speed vs quality.

### Database Migrations
**Two separate SQLite databases** with independent migration chains:
- `photofield.cache.db` - metadata, tags, AI embeddings (`db/migrations/`)
- `photofield.thumbs.db` - cached thumbnails (`db/migrations-thumbs/`)

Migration naming: `{sequence}_{description}.{up|down}.sql`
- Use `task db:add {description}` to create template
- Always include both up AND down migrations
- Key tables: `infos` (photos), `tag`, `infos_tag`, `clip_emb` (AI), `prefix` (path deduplication)

## Development Workflows

### Getting Started (Fresh Worktree or Clone)
**First time setup** - `task dev` automatically installs dependencies:
- Installs UI npm packages (cached based on package.json)
- Downloads geo assets for reverse geocoding (~50MB)
- Starts API + UI in watch mode

**Working in worktrees**: Set `PHOTOFIELD_DATA_DIR=~/code/photofield/data` to share the data directory (config, databases, caches) with the main repo. Without this, each worktree has isolated data.

### Build System (Taskfile.yml)
**Conditional compilation with build tags**:
- `embedui` - embeds `ui/dist/` into binary (requires `task build:ui` first)
- `embedgeo` - embeds `data/geo/*.gpkg` file (~50MB, requires `task assets` to download)
- `embeddocs` - embeds documentation site

Common workflows:
```bash
task dev           # Run API + UI in watch mode (auto-setup on first run)
task setup         # Manually install UI deps + geo assets
task watch         # API only with hot-reload via watchexec
task ui            # UI dev server (Vite)
task run:embed     # Build with embedded UI+docs, run locally
task test          # Run Go tests with race detector
```

**Important**: Default `task build` does NOT embed UI - you get a backend-only binary. Use `task run:embed` for full-stack development or `go build -tags embedui` for production.

### Profiling & Debugging
Built-in profiling infrastructure (always enabled):
- `http://localhost:8080/debug/pprof/` - standard Go profiling
- `task prof:cpu` - opens interactive pprof UI
- `task prof:heap` - memory profiling
- `task prof:trace` - execution trace
- Pyroscope integration when `PHOTOFIELD_PYROSCOPE_SERVER` env set

**Tile request debugging**: Set `tile_requests.log_stats: true` in config for per-request timing logs (format: `priority, start_ms, end_ms, duration_ms`).

## Project-Specific Patterns

### Layout System (`internal/layout/`)
Four layout types (album, timeline, wall/flex, map) implement different photo arrangement algorithms:
- **Album**: traditional photo grid with aspect ratio preservation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SmilyOrg/photofield](https://github.com/SmilyOrg/photofield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
