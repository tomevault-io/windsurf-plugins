---
trigger: always_on
description: StreamClient is a native Apple streaming client for PVR/DVR servers. Supports iOS, iPadOS, tvOS, and macOS from a single codebase using SwiftUI.
---

# CLAUDE.md - StreamClient

## Project Overview

StreamClient is a native Apple streaming client for PVR/DVR servers. Supports iOS, iPadOS, tvOS, and macOS from a single codebase using SwiftUI.

### Variants
- **StreamClient - For NextPVR** (scheme: `NextPVR`) — targets NextPVR server
- **StreamClient** (scheme: `DispatcharrPVR`) — targets Dispatcharr server (Django REST API)

## Tech Stack

- **Platform**: iOS 16+ / macOS 15+ / tvOS 18+
- **Language**: Swift 6.0
- **UI Framework**: SwiftUI
- **Architecture**: MVVM with @Observable
- **Minimum Deployment**: iOS 16.0
- **Package Manager**: Swift Package Manager
- **Video Playback**: MPV (libmpv) via Metal rendering
- **Networking**: URLSession with async/await
- **Data Sync**: iCloud Key-Value Store (NSUbiquitousKeyValueStore)

## Project Structure

```
NexusPVR/
├── NexusPVRApp.swift    # App entry point (@main)
├── ContentView.swift    # Main content view with server config check
├── Core/
│   ├── Models/
│   │   ├── Channel.swift          # Channel model (id, name, number, hasIcon)
│   │   ├── Program.swift          # EPG program model with computed airing/progress
│   │   ├── Recording.swift        # Recording model + RecordingStatus enum
│   │   ├── Session.swift          # API response models + ServerConfig
│   │   └── UserPreferences.swift  # User preferences + PlayerStats
│   ├── Services/
│   │   ├── NextPVRClient.swift    # Main API client with authentication
│   │   ├── ImageCache.swift       # In-memory image cache
│   │   └── MD5Hasher.swift        # MD5 hashing for authentication
│   └── Extensions/
│       └── Notification+Extensions.swift  # Notification names (preferencesDidSync, recordingsDidChange)
├── Design/
│   └── Theme.swift      # Colors, spacing, typography, corner radius, animation, platform-specific styles
├── Features/
│   ├── Guide/           # EPG grid view (GuideView, GuideViewModel, ProgramCell, ProgramDetailView, GuideScrollHelper)
│   ├── LiveTV/          # Live TV channel list (LiveTVView, LiveTVViewModel)
│   ├── Player/          # MPV video player (PlayerView containing MPVPlayerCore + MPVContainerView)
│   ├── Recordings/      # Recording management (RecordingsListView, RecordingsViewModel, RecordingRow, RecordingDetailView)
│   ├── Settings/        # App settings (SettingsView, ServerConfigView, KeywordsEditorView)
│   └── Topics/          # Keyword-based program discovery (TopicsView, TopicsViewModel, TopicProgramRow)
└── Navigation/
    ├── AppState.swift   # Global app state (Tab enum: guide, recordings, topics, settings)
    └── NavigationRouter.swift  # Platform-adaptive navigation (iOS, tvOS, macOS variants)
```

## Key Patterns

### Swift
All struct and enum are in their own swift file.

### Environment Objects
- `NextPVRClient` - API client, injected via `.environmentObject()`
- `AppState` - Global state for playback, navigation

### View Models
- Use `@MainActor` and `ObservableObject` for view models
- All view models follow this pattern: `GuideViewModel`, `LiveTVViewModel`, `RecordingsViewModel`, `TopicsViewModel`

### Platform Conditionals
Use `#if os(tvOS)` / `#if os(macOS)` for platform-specific code:
```swift
#if os(tvOS)
// tvOS-specific UI
#else
// iOS/macOS UI
#endif
```

### Theme System
All styling goes through `Theme.*`:
- **Colors**: `accent`, `accentSecondary`, `background`, `surface`, `surfaceElevated`, `surfaceHighlight`, `textPrimary`, `textSecondary`, `textTertiary`, `success`, `warning`, `error`, `recording`, `guideNowPlaying`, `guidePast`, `guideScheduled`
- **Spacing**: `spacingXS` (4), `spacingSM` (8), `spacingMD` (16), `spacingLG` (24), `spacingXL` (32)
- **Corner Radius**: `cornerRadiusSM` (8), `cornerRadiusMD` (12), `cornerRadiusLG` (20)
- **Animation**: `animationDuration` (0.25), `springAnimation`
- **Platform sizes**: `cellHeight`, `channelColumnWidth`, `hourColumnWidth`, `iconSize` (all differ per platform)
- **Typography**: Font extensions (`displayLarge`, `displayMedium`, `headline`, `subheadline`, `body`, `caption`, `footnote`) + tvOS-specific (`tvTitle`, `tvHeadline`, `tvBody`, `tvCaption`)
- **View Modifiers**: `CardStyle`, `AccentButtonStyle`, `SecondaryButtonStyle`

## NextPVR API

The app communicates with NextPVR server via JSON API:

### Authentication
- Endpoint: `session.initiate` → `session.login`
- Uses PIN-based auth with MD5 hashing: `md5(":" + md5(PIN) + ":" + salt)`
- Stores SID in session, auto-reauthenticates on 401 responses

### Key Endpoints (via NextPVRClient)

**Channels & EPG:**
- `getChannels()` - List all channels
- `getListings(channelId:)` - EPG data for a channel
- `getAllListings(for channels:)` - Batch fetch EPG data for multiple channels
- `channelIconURL(channelId:)` - Get channel icon URL

**Recordings:**
- `getAllRecordings()` - Returns 3-tuple: `(completed: [Recording], recording: [Recording], scheduled: [Recording])`
- `getRecordings(filter:)` - Get recordings with filter ("ready", "recording", "pending")
- `scheduleRecording(eventId:)` - Schedule a recording
- `cancelRecording(recordingId:)` - Cancel a scheduled or in-progress recording
- `setRecordingPosition(recordingId:, positionSeconds:)` - Set resume position for playback

**Streaming:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Drvolks/StreamClient](https://github.com/Drvolks/StreamClient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
