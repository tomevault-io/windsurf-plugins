---
trigger: always_on
description: Rivulet is a tvOS media client for Plex and IPTV. It uses SwiftUI with MPV for video playback with HDR passthrough.
---

# Rivulet - Claude Context

Rivulet is a tvOS media client for Plex and IPTV. It uses SwiftUI with MPV for video playback with HDR passthrough.

## Quick Reference

- **Platform**: tvOS 26+ (Apple TV)
- **Language**: Swift 6
- **UI Framework**: SwiftUI
- **Video Player**: MPV via libmpv (Metal rendering, HDR passthrough)
- **Design Guide**: See `Docs/DESIGN_GUIDE.md` for UI/UX patterns

## Project Structure

```
Rivulet/
├── Models/
│   ├── Plex/           # Plex API models (PlexMetadata, PlexStream, etc.)
│   └── SwiftData/      # Persistent models (Channel, EPGProgram, PlexServer)
├── Services/
│   ├── Plex/           # PlexNetworkManager, PlexAuthManager, PlexDataStore
│   ├── Playback/       # MPVPlayerWrapper, AVPlayerWrapper, MediaTrack
│   ├── LiveTV/         # PlexLiveTVProvider, IPTVProvider, LiveTVDataStore
│   ├── IPTV/           # M3UParser, XMLTVParser, DispatcharrService
│   ├── Cache/          # CacheManager, ImageCacheManager
│   └── Focus/          # FocusMemory (tvOS section focus restoration)
├── Views/
│   ├── Player/         # UniversalPlayerView, PlayerControlsOverlay
│   │   ├── MPV/        # MPVPlayerView, MPVMetalViewController
│   │   └── PostVideo/  # Post-playback summary overlays
│   ├── Plex/           # PlexHomeView, PlexLibraryView, PlexDetailView
│   ├── LiveTV/         # ChannelListView, GuideLayoutView, LiveTVPlayerView
│   ├── Settings/       # SettingsView, SettingsComponents
│   ├── Components/     # CachedAsyncImage, GlassRowStyle
│   ├── TVNavigation/   # TVSidebarView, NavigationEnvironment
│   └── Root/           # SidebarView
└── Docs/
    └── DESIGN_GUIDE.md # Comprehensive UI/UX documentation
```

## Key Architectural Patterns

### Focus Management (tvOS)

Uses standard SwiftUI focus primitives with `FocusMemory` for section-level restoration. No custom focus scope manager — focus isolation is handled by system mechanisms:

- **`fullScreenCover`** — automatic focus isolation for overlays/popups
- **`TabView` with `sidebarAdaptable`** — system-managed sidebar/content focus
- **`@FocusState` + `.onAppear`** — setting initial focus in presented views
- **`FocusMemory`** — remembers and restores focus within scrollable sections

```swift
// Section focus memory
.focusSection()
.remembersFocus(key: "uniqueSectionKey", focusedId: $focusedItemId)

// Initial focus in fullScreenCover (no Namespace/resetFocus needed)
.onAppear {
    focusedUserId = profileManager.selectedUser?.id
}
```

### Video Player Architecture

- **UniversalPlayerView**: Main SwiftUI player container
- **UniversalPlayerViewModel**: Manages playback state, markers, post-video logic
- **MPVPlayerWrapper**: Bridges MPV to Swift (Combine publishers for state)
- **MPVMetalViewController**: UIViewController hosting Metal layer for MPV rendering

**Playback States**: `.idle`, `.loading`, `.playing`, `.paused`, `.buffering`, `.ended`, `.failed`

### Plex Metadata Hierarchy

For TV shows:
- **Show** (`grandparentRatingKey`) → **Season** (`parentRatingKey`) → **Episode** (`ratingKey`)
- Episode has `index` (episode number) and `parentIndex` (season number)

**Note**: Items from "Continue Watching" hub may lack parent metadata. Use `PlexNetworkManager.getMetadata()` to fetch full details.

### Glass UI Style

All focusable rows use consistent styling (see `Docs/DESIGN_GUIDE.md`):

```swift
// Background
.fill(isFocused ? .white.opacity(0.18) : .white.opacity(0.08))
.strokeBorder(isFocused ? .white.opacity(0.25) : .white.opacity(0.08), lineWidth: 1)

// Scale
.scaleEffect(isFocused ? 1.02 : 1.0)

// Animation
.animation(.spring(response: 0.3, dampingFraction: 0.7), value: isFocused)
```

## Common Tasks

### Presenting a Focus-Isolated Overlay

Use `fullScreenCover` — it provides automatic focus isolation without manual scope management:

```swift
.fullScreenCover(isPresented: $showOverlay) {
    MyOverlayView(isPresented: $showOverlay)
        .presentationBackground(.clear)  // See-through to content behind
}
```

In the presented view, use `@FocusState` with `.onAppear`:
```swift
@FocusState private var focusedItem: String?

.onAppear {
    focusedItem = defaultItemId
}
.onExitCommand {
    isPresented = false
}
```

### Fetching Next Episode

```swift
// Get episodes in current season
let episodes = try await networkManager.getChildren(
    serverURL: serverURL,
    authToken: authToken,
    ratingKey: metadata.parentRatingKey  // Season key
)

// Find next episode
let next = episodes.first(where: { $0.index == currentEpisodeIndex + 1 })
```

### Adding Settings

Use components from `SettingsComponents.swift`:
- `SettingsRow` - Navigation with chevron
- `SettingsToggleRow` - On/Off toggle
- `SettingsPickerRow` - Cycles through options
- `SettingsActionRow` - Action button (supports destructive)

### Image Loading

Always use `CachedAsyncImage` for remote images:
```swift
CachedAsyncImage(url: imageURL) { phase in
    switch phase {
    case .success(let image): image.resizable()
    case .empty: ProgressView()
    case .failure: Image(systemName: "photo")
    }
}
```

## Build & Run

```bash
# Build for tvOS Simulator
xcodebuild -scheme Rivulet -destination 'platform=tvOS Simulator,name=Apple TV' build

# Build for device

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [l984-451/Rivulet](https://github.com/l984-451/Rivulet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
