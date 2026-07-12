---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Frontend
pnpm dev              # Vite dev server (port 1420, HMR on 1421)
pnpm build            # tsc --noEmit + vite build

# Full Tauri app
pnpm tauri dev        # Run desktop app in dev mode
pnpm tauri build      # Production build

# Rust (run from src-tauri/)
cargo check                         # Fast compile check
cargo test                          # Run tests
cargo build --manifest-path src-tauri/Cargo.toml
```

Full check (both sides): `cargo check --manifest-path src-tauri/Cargo.toml 2>&1 | grep -E "^error|Finished" && echo "---" && npx tsc --noEmit 2>&1 | head -15`

## Architecture

BonNext is a **Tauri v2 desktop app** — a cross-platform Minecraft Java Edition launcher with a ZZZ-inspired Neo-Tokyo cyberpunk aesthetic. Rust backend, React 18 + TypeScript frontend.

### Backend (Rust — `src-tauri/src/`)

The binary entry is `main.rs` → calls `lib.rs::run()` which registers all ~50 Tauri commands and starts Tauri.

**`lib.rs`** — All `#[tauri::command]` handlers live here. Managed state includes `AppState { launch_state: Arc<Mutex<LaunchState>> }` and `ApiCache` (in-memory TTL cache for content API responses).

**Core launcher modules** (original, don't touch lightly):

- `auth/` — Microsoft OAuth 2.0 device flow + offline mode. Token store with auto-refresh.
- `config.rs` — JSON config at `config_dir/config.json`. Defaults: 2GB max / 512MB min memory, 8 concurrent downloads.
- `download/` — Parallel download queue with retry, SHA1 verification, mirror failover (Official → BMCLAPI → MCBBS).
- `version/` — Mojang manifest fetching, JSON parsing with parent inheritance, OS/feature rule evaluation.
- `launch/` — State machine (idle→checking→downloading→validating→launching→running/crashed/exited). JVM arg builder, process spawner.
- `loader/` — Fabric + Forge installation via meta.fabricmc.net / maven.minecraftforge.net.
- `instance/` — Per-instance config with isolated `.minecraft` dirs. Shared libraries/assets via hard links.
- `platform/` — Java detection, structured logging (tracing-subscriber), cross-platform path management.
- `error.rs` — Unified `LauncherError` enum (thiserror). HTTP, IO, JSON, ZIP, SHA1 mismatch, and domain errors.
- `http_client.rs` — reqwest factory. Sets `User-Agent: BonNext/1.0 (MinecraftLauncher)` header.

**Content platform modules** (marketplace, collections, updates):

- `modrinth.rs` — Modrinth API v2: search, popular, project details, versions, file download. Types: `ModResult`, `ModProjectFull`, `ModVersion`, `ModFile`.
- `curseforge.rs` — CurseForge API v1: search, featured, mod files, download. Maps CF responses to shared `ModResult`/`ModFile` types. Uses a default community API key.
- `cache.rs` — In-memory TTL cache (5 min default) for Modrinth/CF API responses. Separate maps for searches, projects, popular items.
- `content.rs` — Per-instance install metadata (`installed_content.json`). Tracks slug, version_id, content_type for update checking.
- `collections.rs` — User wishlist/saved items (`collections.json`). CRUD operations for saving content across sessions.

### Frontend (React — `src/`)

**`api.ts`** — Typed wrappers around `invoke()` for every Tauri command (~50 methods). Also provides `onDownloadProgress()` for real-time game download events. Key method groups: versions, config, auth, instances, Modrinth, CurseForge, collections, library, marketplace, updates.

**State management** — React Context + `useReducer` pattern in `src/stores/`:

- `authStore.tsx` — logged-in user, account list, login/logout/switchAccount
- `configStore.tsx` — app settings with save/reload
- `instanceStore.tsx` — instance CRUD
- `toastStore.tsx` — toast notification queue (auto-dismiss, max 5)
- `themeStore.tsx` — dark/light/OLED theme, persisted to localStorage
- `downloadStore.tsx` — download task queue for the Steam-style download panel

**Routing** — Hash-based manual routing in `App.tsx` (no React Router). `getPageFromHash()` parses `window.location.hash`. Current routes:

| Hash                  | Page               | Description                                                         |
| --------------------- | ------------------ | ------------------------------------------------------------------- |
| `#/home`              | HomePage           | Dashboard with launch panel + news                                  |
| `#/store`             | StorePage          | Marketplace hub: banner, categories, trending                       |
| `#/store/:type/:slug` | ContentDetailPage  | Content detail: desc, versions, gallery, deps                       |
| `#/instances`         | InstancesPage      | Instance list with search/filters                                   |
| `#/instances/new`     | NewInstancePage    | Instance creation wizard                                            |
| `#/instances/:id`     | InstanceDetailPage | Instance overview & management                                      |
| `#/mods`              | ModsPage           | Content browser: tabs, gallery/list, pagination, Modrinth/CF toggle |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NoWint/BonNextMinecraftLauncher-Rust](https://github.com/NoWint/BonNextMinecraftLauncher-Rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
