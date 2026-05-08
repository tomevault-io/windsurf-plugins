---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
pnpm tauri dev              # Start full Tauri dev (Vite + Rust, hot-reload)

# Frontend tests (Vitest + jsdom)
npx vitest run              # Run once
npx vitest                  # Watch mode

# Rust tests
cd src-tauri && cargo test  # All backend tests

# Type checking
npx tsc --noEmit            # TypeScript check only

# Build
pnpm tauri build            # Production build (DMG on macOS, NSIS on Windows)
```

## Architecture

Tauri v2 **cross-platform** desktop app: **React 19 frontend** communicating with **Rust backend** via IPC commands. Targets **macOS** and **Windows** — all features must consider both platforms (keyboard shortcuts, platform APIs, window behavior, etc.).

### Frontend → Backend Communication

- Frontend calls Rust via `invoke("command_name", { args })` from `@tauri-apps/api/core`
- Rust emits events to frontend via `app.emit("event-name", payload)` — frontend listens with `listen()`
- Key events: `"clipboard-changed"` (triggers UI refresh), `"recopy-show"` (panel visibility)

### Backend (src-tauri/src/)

| Module | Purpose |
|--------|---------|
| `lib.rs` | App setup: plugin registration, tray menu (i18n), window management, global shortcut, clipboard monitor, blur-to-hide |
| `commands/clipboard.rs` | All Tauri IPC commands: CRUD, paste, favorites, settings, shortcut register/unregister, HUD |
| `db/` | SQLite via SQLx — `models.rs` (types), `queries.rs` (SQL), `mod.rs` (pool init + migrations) |
| `clipboard/mod.rs` | Utilities: SHA-256 hashing, thumbnail generation (400px, async for files), original image storage |
| `platform/macos.rs` | NSPanel integration — floating non-activating panel, HUD panel, main-thread dispatch for AppKit |
| `platform/fallback.rs` | Stubs for non-macOS platforms |

### Frontend (src/)

| Module | Purpose |
|--------|---------|
| `App.tsx` | Router: main panel / settings (`?page=settings`) / HUD (`?page=hud`) / preview (`?page=preview`). Panel animation state machine |
| `stores/` | Zustand stores — `clipboard-store` (items, search, filters), `settings-store` (theme, language, shortcut, retention, panel_position), `toast-store`, `update-store` |
| `components/` | UI: ClipboardList, ClipboardCard, SearchBar, TypeFilter, ViewTabs, TextCard/ImageCard/FileCard/RichTextCard/LinkCard, ItemContextMenu, SettingsPage, PreviewPage, CopyHud, UpdateBanner |
| `hooks/useKeyboardNav.ts` | Arrow keys (group-aware ↑↓), Enter (paste), Cmd+C (copy), Cmd+F (search), Cmd+, (settings), Escape (hide) |
| `hooks/useThumbnail.ts` | Lazy-load thumbnails on demand from backend |
| `lib/paste.ts` | `pasteItem()` / `copyToClipboard()` — invokes Rust paste commands |
| `i18n/` | react-i18next config + locale JSONs (zh, en) |

### Windows

| Window | Size | Behavior |
|--------|------|----------|
| Main panel | Configurable position: bottom/top (full-width × 380px), left/right (380px × full-height) | NSPanel, non-activating, always-on-top, blur-to-hide (configurable), resizable in top/bottom modes |
| Settings | 720×560 (min 600×440), centered | Resizable. Re-registers global shortcut on close |
| Preview | 600×480, adjacent to main panel | Non-activating NSPanel, Quick Look for clipboard items |
| HUD | 140×140, centered | Non-activating NSPanel, 800ms auto-hide, copy feedback |

### Paste Flow (critical path)

1. User presses Enter → `pasteItem(item)` → `invoke("paste_clipboard_item")`
2. Rust writes content to system clipboard (text/image/file/rich_text)
3. `platform_resign_before_paste()` — **sync** main-thread dispatch (NSPanel resigns key window)
4. `simulate_paste()` — osascript Cmd+V with 50ms delay
5. `platform_hide_window()` — **async** main-thread dispatch (avoids deadlock from blur handler)

### NSPanel (macOS)

The main window is converted to NSPanel for non-activating floating behavior. **All AppKit operations must run on the main thread** — Tauri commands execute on tokio worker threads, so `app.run_on_main_thread()` is required. `platform_resign_before_paste` uses `sync_channel` for synchronous waiting; `platform_hide_window` is fire-and-forget to avoid deadlock when called from the blur event handler (which already runs on main thread).

### Data Model

```
ClipboardItem: id (UUID), content_type (plain_text|rich_text|image|file|link),
  plain_text, rich_content?, thumbnail? (binary), image_path? (original file),
  file_path?, file_name?, source_app, source_app_name,
  content_size, content_hash (SHA-256), is_favorited, created_at, updated_at
```

- Images: original saved to `app_data/images/YYYY-MM/{uuid}.png`, thumbnail (400px) stored as blob in DB
- File thumbnails: generated async in background, avoids blocking clipboard processing
- Dedup: SHA-256 hash check before insert — duplicates bump `updated_at` instead
- Size limit: configurable max_item_size_mb (default 10MB), directories and oversized files auto-skipped

### Settings System

Persisted to SQLite `settings` table. Frontend manages via `settings-store.ts`, backend via `get_setting`/`set_setting` commands.

| Setting | Key | Default | Notes |
|---------|-----|---------|-------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shiqkuangsan/Recopy](https://github.com/shiqkuangsan/Recopy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
