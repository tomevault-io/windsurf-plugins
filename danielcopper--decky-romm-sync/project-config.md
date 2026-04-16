---
trigger: always_on
description: A Decky Loader plugin that syncs a self-hosted RomM library into Steam as Non-Steam shortcuts. Games launch via RetroDECK. The QAM panel handles settings, sync, downloads, and BIOS management.
---

# decky-romm-sync — Decky Loader Plugin

## What This Is

A Decky Loader plugin that syncs a self-hosted RomM library into Steam as Non-Steam shortcuts. Games launch via RetroDECK. The QAM panel handles settings, sync, downloads, and BIOS management.

## Architecture

```
RomM Server <-HTTP-> Python Backend (main.py + services/ + adapters/)
                          | callable() / emit()
                   Frontend (TypeScript) <-> SteamClient.Apps API
                          |
                     Steam Library (shortcuts appear instantly)
                          |
                     bin/romm-launcher (bash) -> RetroDECK (flatpak)
```

**Backend layers** (dependency direction: services → protocols ← adapters):
- **`main.py`** — Plugin entry point, callable routing, composition root
- **`py_modules/services/`** — Business logic (9 services, depend on Protocols only)
- **`py_modules/adapters/`** — I/O boundaries (HTTP, persistence, Steam VDF, save API)
- **`py_modules/domain/`** — Domain logic (ES-DE config, RetroDECK path resolution)
- **`py_modules/models/`** — Domain dataclasses
- **`py_modules/lib/`** — Utilities (errors, certifi bundle)

**Frontend** (`src/`): SteamClient shortcut CRUD, QAM panel UI, game detail page injection

**Communication**: `callable()` for request/response, `decky.emit()` for backend-to-frontend events

**Layer rules** (enforced by import-linter in CI):
- Services must not import concrete adapter implementations (Protocols OK)
- Adapters must not import services
- Domain must not import services, adapters, or lib
- Utilities must not import services, adapters, or domain
- Services must be independent of each other

## Key Technical Constraints

- **Shortcuts**: Use `SteamClient.Apps.AddShortcut()` from frontend JS, NOT VDF writes. VDF edits require Steam restart; SteamClient API is instant.
- **Frontend API**: `@decky/ui` + `@decky/api` (NOT deprecated `decky-frontend-lib`). Use `callable()` (NOT `ServerAPI.callPluginMethod()`).
- **RomM API quirks**: Filter param is `platform_ids` (plural). Cover URLs have unencoded spaces (must URL-encode). Paginated: `{"items": [...], "total": N}`.
- **AddShortcut timing**: Must wait 300-500ms after `AddShortcut()` before setting properties. Use 50ms delay between operations.
- **Large payloads**: Never send bulk base64 data through `decky.emit()` — WebSocket bridge has size limits. Use per-item callables instead.
- **SteamGridDB**: Requires `User-Agent` header — Python's default `Python-urllib` gets 403'd. Use `decky-romm-sync/0.1`.
- **AddShortcut ignores most params**: `SteamClient.Apps.AddShortcut(name, exe, startDir, launchOptions)` ignores startDir and launchOptions (confirmed by MoonDeck plugin). Must use `Set*` calls (`SetShortcutName`, `SetShortcutExe`, `SetShortcutStartDir`, `SetAppLaunchOptions`) after a 500ms delay. Do NOT pass quoted exe paths — the API handles quoting internally.
- **BIsModOrShortcut bypass DROPPED**: Phase 5.6 removed the bypass counter entirely. Shortcuts return `BIsModOrShortcut() = true` (natural state). We own the entire game detail UI via RomMPlaySection + future RomMGameInfoPanel. See `docs/game-detail-ui.md` section 2 for the rationale.
- **Shortcut property re-sync**: Changing exe, startDir, or launchOptions on existing shortcuts may not take effect reliably. Full delete + recreate (re-sync) is required for changes to launch config.
- **RomM 4.6.1 Save API**: `GET /api/saves/{id}/content` does not exist — use `download_path` from save metadata (URL-encode spaces/parens). No `content_hash` in SaveSchema — use hybrid timestamp + download-and-hash. `POST /api/saves` upserts by filename. `GET /api/roms/{id}/notes` returns 500 — read `all_user_notes` from ROM detail instead. `device_id` param is accepted but ignored. See wiki Save-File-Sync-Architecture for full details.
- **Decky callables must be async**: Even if the method body is synchronous, Decky's callable framework requires `async def`. Do not remove `async` from callable methods in main.py.

## File Structure

```
main.py                                   # Plugin entry point, callable routing, bootstrap
py_modules/
  bootstrap.py                            # Composition root — wires adapters and services
  services/
    protocols.py                          # Protocol interfaces (HttpAdapter, SteamConfigAdapter, SaveApiProtocol)
    library.py                            # LibraryService — fetch ROMs, create shortcuts, artwork staging
    saves.py                              # SaveService — upload/download .srm, conflict detection
    playtime.py                           # PlaytimeService — session recording, RomM notes
    downloads.py                          # DownloadService — ZIP extraction, M3U, queue, progress
    firmware.py                           # FirmwareService — registry, downloads, per-core filtering
    steamgrid.py                          # SteamGridService — SteamGridDB fetch, cache, icons
    metadata.py                           # MetadataService — ROM metadata caching (TTL, app_id mapping)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielcopper) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
