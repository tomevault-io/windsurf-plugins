---
trigger: always_on
description: NipaPlay-Reload is a cross-platform video player built with Flutter/Dart supporting Windows, macOS, Linux, Android, and iOS. The app integrates with Emby/Jellyfin media servers, provides danmaku (bullet comments), and supports multiple player kernels through an abstraction layer.
---

# NipaPlay-Reload AI Agent Guidelines

## Project Overview
NipaPlay-Reload is a cross-platform video player built with Flutter/Dart supporting Windows, macOS, Linux, Android, and iOS. The app integrates with Emby/Jellyfin media servers, provides danmaku (bullet comments), and supports multiple player kernels through an abstraction layer.

## Communication
**CRITICAL**: Always communicate in Chinese (中文) with users unless explicitly requested otherwise.

## Architecture Fundamentals

### Abstraction Layer Pattern (Core Design Philosophy)
The project uses **pluggable architecture** for both video players and danmaku engines:

#### Player Abstraction (`lib/player_abstraction/`)
- **Interface**: `abstract_player.dart` defines `AbstractPlayer` - all player kernels must implement this
- **Adapters**: `mdk_player_adapter.dart`, `media_kit_player_adapter.dart`, `video_player_adapter.dart` wrap specific SDKs
- **Factory**: `player_factory.dart` creates player instances based on `PlayerKernelType` enum from SharedPreferences
- **Adding a new player**: Create adapter → Implement `AbstractPlayer` → Add to `PlayerKernelType` enum → Register in factory → Add UI option in settings

#### Danmaku Abstraction (`lib/danmaku_abstraction/`)
- Similar pattern: `DanmakuRenderEngine` enum (CPU/GPU/Canvas)
- `danmaku_kernel_factory.dart` manages kernel selection
- Three implementations: `danmaku_gpu/`, `danmaku_canvas/`, CPU renderer

### Service Layer Architecture (`lib/services/`)
Singleton services handle external integrations and core functionality:
- **Media Server Services**: `jellyfin_service.dart`, `emby_service.dart` (API clients with auth, library management)
  - Multi-address support via `multi_address_server_service.dart` 
  - Transcode management: `jellyfin_transcode_manager.dart`, `emby_transcode_manager.dart`
  - Playback sync: `jellyfin_playback_sync_service.dart`, `emby_playback_sync_service.dart`
- **Danmaku Services**: `dandanplay_service.dart` (弹弹play API integration), `danmaku_cache_manager.dart`
- **Infrastructure**: `debug_log_service.dart`, `web_server_service.dart` (embedded HTTP server using `shelf`)
- **Platform-specific**: `file_association_service.dart`, `windows_file_association_service.dart`

### State Management
- **Provider pattern** for global state (`lib/providers/`)
  - `ServiceProvider` - centralizes service singletons
  - `WatchHistoryProvider`, `UIThemeProvider`, `JellyfinTranscodeProvider`, etc.
  - All registered in `main.dart` with `MultiProvider`
- Services can extend `ChangeNotifier` (e.g., `ScanService`) for reactive state

### Data Layer (`lib/models/`)
- **Models**: `jellyfin_model.dart`, `emby_model.dart`, `bangumi_model.dart`, `playable_item.dart`
- **Database**: `watch_history_database.dart` (SQLite via `sqflite`/`sqflite_common_ffi` for desktop)
- **Configuration**: Settings stored in `SharedPreferences`

### UI Structure (`lib/pages/`)
- Main navigation: `main.dart` (Material) and `fluent_main_page.dart` (Fluent UI for Windows)
- Key pages: `play_video_page.dart`, `anime_page.dart`, `settings_page.dart`, `media_server_detail_page.dart`
- Reusable components: `lib/widgets/` (organized by theme: `nipaplay_theme/`)

## Development Workflows

### Building
```bash
# Standard Flutter builds
flutter build windows --release
flutter build macos --release
flutter build linux --release
flutter build android --release
flutter build ios --release
flutter build web  # See build_and_copy_web.sh for web assets handling

# Custom scripts
./build-arm64.sh        # Linux ARM64 build
./build_and_copy_web.sh # Builds web + copies to assets/web/
```

### Running & Debugging
```bash
flutter run -d <device>
# Launch arguments supported for file association (desktop/Android)
flutter run lib/main.dart
```

### Testing
- Test directory: `test/` (minimal - contributions welcome)
- Debug logs: Use `DebugLogService()` - initialized early in `main.dart`
  - Accessible via developer options in settings UI

### Key Initialization Sequence (`main.dart`)
1. `HttpClientInitializer.install()` - Self-signed cert trust (desktop)
2. `DebugLogService().initialize()` - Enable logging from startup
3. Platform-specific: `hotKeyManager.unregisterAll()` (desktop), file association handlers
4. `PlayerFactory.initialize()` / `DanmakuKernelFactory.initialize()` - Preload settings
5. Provider setup → `runApp()`

## Project-Specific Conventions

### Code Style
- **SOLID principles**: Functions do one thing, exceptions handled, meaningful variable names
- **No "optimizations" without justification**: If code exists, assume it's needed unless proven otherwise
- **Chinese comments encouraged** for complex logic (team is Chinese-speaking)

### File Naming
- Services: `*_service.dart`
- Providers: `*_provider.dart` 
- Models: `*_model.dart`
- Pages: `*_page.dart`
- Platform conditionals: `*_io.dart` (native), `*_web.dart` (web), `*_stub.dart` (unsupported)

### Platform Handling
- Conditional imports: `import 'path_provider.dart' if (dart.library.html) 'mock_path_provider.dart';`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AimesSoft/NipaPlay-Reload](https://github.com/AimesSoft/NipaPlay-Reload) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
