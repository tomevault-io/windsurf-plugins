---
trigger: always_on
description: > This file is the source of truth for all LLM agents (Claude, Gemini) working on this codebase.
---

# Freegosy — Agent Map
> This file is the source of truth for all LLM agents (Claude, Gemini) working on this codebase.
> Read this before touching any file. Update this file if you create or split any file.

## Project Overview
Freegosy is a cross-platform Flutter app for browsing a RomM library, downloading ROMs via HTTP, and launching emulators. Built with Riverpod for state management.

## Rules (MANDATORY)
- No file exceeds 600 lines. If adding code would exceed this, split the file first and update this map.
- All RomM API calls go through `romm_service.dart` only. Never call the API directly from UI or providers.
- All emulator logic goes through the strategy pattern. Never hardcode emulator behavior in UI.
- New emulator = new file in `core/emulator/strategies/`, register in `strategy_registry.dart` only.
- New save strategy = new file in `core/save/strategies/`, register in `save_sync_service.dart` only.
- New screen = new file in `ui/screens/`.
- New reusable widget = new file in `ui/widgets/`.
- Providers are thin — they call services, they do not contain business logic.
- Never use `Platform.environment` or `Platform.isWindows` directly — causes conflicts with flutter/foundation.dart. Use `import 'dart:io' as io;` and `io.Platform` or `defaultTargetPlatform == TargetPlatform.windows`.
- Use `PlatformInfo` for cross-platform abstraction (file: `core/platform/platform_info.dart`). Accept it as a constructor parameter, never use `dart:io Platform` directly in services.
- ROM name sanitization: `!` must be included in the sanitization regex across all services (`extraction_service`, `directory_service`, `rom_lookup_service`) for consistent folder naming.
- Windows games are folder-based: `findMainRomInFolder()` returns the folder path for `windows`/`pc`/`win` slugs, not a file.

## File Map

### Entry Points
- `lib/main.dart` — App entry point. Initializes Riverpod ProviderScope. Calls app.dart.
- `lib/app.dart` — MaterialApp setup, theme, initial route, navigation shell.

### Core — RomM
- `lib/core/romm/romm_service.dart` — All RomM HTTP calls (Dio). Methods: getPlatforms(), getGames(), getAllGames(), getGamesPage(offset, limit, platformId, search), getSaves(), uploadSave(), getLatestSave(), downloadSave(), pruneOldSaves(), getRecentlyPlayed(), getRandomGame(), updateRomProps(), refreshToken(), fetchToken(). Includes silent re-authentication interceptor. Upload uses slot `'freegosy'` (not timestamped), `autocleanup: true`, `autocleanupLimit: 5`, `overwrite: 'force'`.
- `lib/core/romm/romm_models.dart` — Data models: Game, Platform (with fsSlug, displayName, gamesCount and flexible parsing), SaveFile, RomMConfig.
- `lib/core/romm/rom_constants.dart` — Platform slug-to-extension mappings. Windows/PC/Win slugs have empty extension lists (folder-based platforms). PSX/PS2 include `.chd`.

### Core — Save Sync
- `lib/core/save/save_strategy.dart` — Abstract base class SaveStrategy. Methods: getSaveDir(), getSaveFiles(), restoreSave(). Helpers: backupSave() keeps max 3 clean versions (.bak, .bak1, .bak2), getRomStem().
- `lib/core/save/save_sync_service.dart` — SaveSyncService. Methods: pushSaves(), pullSave(), getStrategyForSlug(). getStrategyForSlug() checks StrategyRegistry user preferences first before falling back to platform slug defaults. Wires all strategies to RommService. Exposes windowsSaveStrategy for external access. Has 60s pull cooldown per game ID (in-memory `Map<String, DateTime> _lastPullCheck`). Filename normalization strips RomM timestamp tags `[_timestampPattern]`.
- `lib/core/save/backup_entry.dart` — BackupEntry plain model + hand-written BackupEntryAdapter (typeId=1). Registered in main.dart.
- `lib/core/save/backup_repository.dart` — BackupRepository. Opens the 'freegosy_backups' Hive box. Methods: getEntries(), addEntry() (enforces 8-cap rotation + disk cleanup), removeEntry(), markAsSynced(), getUnsyncedEntries().
- `lib/core/save/backup_service.dart` — BackupService. Methods: createImmediate() (reuses ZipFileEncoder pipeline from SaveSyncService, writes to getApplicationSupportDirectory()/backups/), restore() (extracts chosen zip back to save dir using archive package).
- `lib/core/save/background_sync_queue.dart` — BackgroundSyncQueue. Processes unsynced local backups serially with a 5-second throttle. Triggered on app startup and network reconnection.
- `lib/core/save/strategies/ares_save_strategy.dart` — Ares emulator save strategy. Per-platform extension classification (confirmed/defaulted/log-only). Stem-prefix filename matching. Fresh-install directory creation. 30 platform slugs mapped to folder names.
- `lib/core/save/strategies/retroarch_save_strategy.dart` — RetroArch save strategy. Handles dual-stem matching for states.
- `lib/core/save/strategies/dolphin_save_strategy.dart` — Dolphin save strategy (GC/Wii).
- `lib/core/save/strategies/eden_save_strategy.dart` — Eden/Switch save strategy. Resolves title ID.
- `lib/core/save/strategies/azahar_save_strategy.dart` — Azahar/3DS save strategy. Zip-based sync for SDMC data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abduznik/Freegosy](https://github.com/abduznik/Freegosy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
