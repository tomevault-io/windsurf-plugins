---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Stashy is a native iOS/tvOS SwiftUI application for browsing and managing a Stash media server. It supports multiple server configurations, custom filtering, video playback, and features like StashTok (reels-style viewing).

## Build Commands

### iOS Target
```bash
xcodebuild -project stashy.xcodeproj -scheme stashy -destination 'generic/platform=iOS' build
```

### tvOS Target
```bash
xcodebuild -project stashy.xcodeproj -scheme stashyTV -destination 'generic/platform=tvOS' build
```

### Running Tests
```bash
# Run all tests
xcodebuild test -project stashy.xcodeproj -scheme stashy -destination 'platform=iOS Simulator,name=iPhone 15'

# Run specific test class
xcodebuild test -project stashy.xcodeproj -scheme stashy -destination 'platform=iOS Simulator,name=iPhone 15' -only-testing:stashyTests/SpecificTestClass
```

## Architecture Overview

### Core Patterns

**Singleton Managers**: The app uses shared singleton instances for cross-cutting concerns:
- `AppearanceManager.shared` - Manages theme/tint colors and O-counter icon selection, persists to UserDefaults
- `ServerConfigManager.shared` - Multi-server config storage, active server selection
- `TabManager.shared` - Tab visibility, ordering, dashboard rows, and reels mode configuration
- `KeychainManager.shared` - Secure API key storage (iOS only, not tvOS)
- `GraphQLClient.shared` - Centralized network client with SSL handling for local servers
- `ImageCacheManager.shared` - Dual-tier image cache (memory + disk, server-scoped)

**Navigation**: `NavigationCoordinator` is passed as an `@EnvironmentObject` throughout the app. It manages:
- Tab selection and deep linking
- Navigation stack resets (via UUID-based `.id()` modifiers)
- Cross-tab navigation (e.g., opening a performer from a scene detail)
- Remote state injection for filters/sorts

**Main ViewModel**: `StashDBViewModel` is a large (~6000 lines) `@MainActor` class that handles:
- Server connectivity and status (staggered init: Filters → Statistics → Ready)
- Statistics fetching and saved filters from Stash server
- Scene metadata (performers, studios, tags, galleries)
- O-counter (play count) tracking with optimistic updates
- Nested managers: `DownloadManager`, `HandyManager` (TheHandy device), `ButtplugManager` (Intiface/funscript)

**Domain ViewModels**: Specialized view models in `stashy/ViewModels/`:
- `ScenesViewModel` - Scene browsing, filtering, sorting
- `PerformersViewModel` - Performer listing
- `GalleriesViewModel` - Gallery browsing
- `StudiosViewModel` - Studio filtering and browsing
- `TagsViewModel` - Tag management

**Repository Layer**: `stashy/Repositories/` contains data access objects:
- `SceneRepository`, `PerformerRepository`, `StudioRepository`, `GalleryRepository`, `TagRepository`, `FilterRepository`
- These encapsulate GraphQL queries and data fetching logic

**Pagination**: `stashy/Utilities/PaginatedLoader.swift` provides a generic `PaginatedLoader<T>` with `loadInitial()`, `loadMore()`, `refresh()`, and `reset()` methods. Includes convenience static builders for each content type.

### Data Flow

1. **Network Layer**: `GraphQLClient` handles all GraphQL requests
   - Actor-based design for thread safety
   - Custom `URLSessionDelegate` for self-signed SSL certificates on local networks
   - Automatic retry logic for "database is locked" errors (common with SQLite-backed Stash)
   - Supports async/await, Combine, and completion handler APIs

2. **Image Loading**: `ImageCacheManager` provides dual-tier caching:
   - Memory cache: 300 MB, 300 items max
   - Disk cache: ~30-day TTL, server-scoped (separate cache per server ID to prevent data leakage on server switch)
   - Stable cache key normalization: strips timestamp params but retains `width`/`height`/`size` params
   - `CustomAsyncImage` view component for easy image loading with fallbacks
   - Auto-cleanup runs every 4 hours

3. **GraphQL Queries**: Stored as `.graphql` files in `graphql/` directory
   - Loaded at runtime via `GraphQLQueries.loadQuery(named:)`
   - Thread-safe in-memory caching after first load (concurrent DispatchQueue)
   - Contains fragment files (`fragment_*.graphql`) for reusable field sets
   - `queryWithFragments` method for composing queries with fragments

4. **Server Configuration**:
   - Multi-server support via `ServerConfig` (Codable, persisted in UserDefaults)
   - Each server has: name, address, port, protocol (HTTP/HTTPS), streaming quality settings
   - API keys stored in Keychain (iOS) with UserDefaults fallback
   - Server-specific settings use suffix pattern: `"key_\(serverID)"`
   - Switching servers posts `"ServerConfigChanged"` notification, triggering app-wide data reset

5. **Settings Architecture** (refactored 2026-02-06):
   - All settings views live in `stashy/Settings/`
   - Main entry point: `SettingsView.swift` (replaces old ServerConfigView)
   - Modular sections: `ServerListSection`, `PlaybackSettingsSection`, `ContentSettingsSection`
   - Separate views: `ServerDetailView`, `DashboardSettingsView`, `ReelsModeSettingsView`, `AppearanceSettingsView`, etc.

### Platform Differences


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1letzgo/stashy](https://github.com/1letzgo/stashy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
