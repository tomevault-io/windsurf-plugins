---
trigger: always_on
description: **DankPod** is an iPod Classic emulator iOS application written in Swift. It recreates the iconic iPod Classic user interface — including the click wheel, gradient navigation bar, and hierarchical menu navigation — as a native iOS app that plays music from the device's Apple Music library.
---

# Project Knowledge Base

## 1. Project Overview

**DankPod** is an iPod Classic emulator iOS application written in Swift. It recreates the iconic iPod Classic user interface — including the click wheel, gradient navigation bar, and hierarchical menu navigation — as a native iOS app that plays music from the device's Apple Music library.

- **Purpose**: Fun/hobby project emulating the iPod Classic experience on modern iPhones
- **Author**: Alistair Pullen (Apullen Developments Ltd.)
- **Created**: May 22, 2020
- **License**: MIT
- **Status**: Work in progress — Music browsing/playback works; Photos, Videos, Extras, and Settings sections are not implemented; queue system is partially implemented via force touch

### Key Features
- Full iPod Classic click wheel with circular gesture recognition
- Music library browsing (Playlists, Artists, Albums, Songs, Genres)
- Now Playing screen with album art, progress bar, and volume control
- Shuffle Songs mode
- Play/pause from any screen
- Force touch (3D Touch) support for additional actions
- Haptic feedback on all click wheel interactions
- Battery level indicator in iPod-style nav bar
- Partial queue management system (via force touch + hold combos)

## 2. Architecture Overview

### High-Level Architecture

The app follows a **UIKit programmatic UI** pattern (no storyboards used at runtime). The architecture is built around a central `BasePodView` that manages the iPod chrome (nav bar + click wheel), with child view controllers swapped into a screen content area.

```
┌─────────────────────────────────────────┐
│  AppDelegate                            │
│  ├── static music: Music                │ ← Global music service singleton
│  └── static baseVC: MainViewController  │ ← Global base view controller
│       ├── NavBar (gradient, title, icons)│
│       ├── Screen area (child VC content) │
│       └── Click Wheel (gesture input)   │
│            └── ClickWheelProtocol       │ ← Delegate pattern to active VC
└─────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility |
|-----------|---------------|
| **AppDelegate** | App lifecycle; holds static references to `Music` and `MainViewController` |
| **SceneDelegate** | iOS 13+ scene lifecycle; creates window and sets `SplashScreenViewController` as root |
| **SplashScreenViewController** | 2-second splash screen → presents `MainViewController` |
| **MainViewController** | Subclass of `BasePodView`; manages `UINavigationController` for screen content |
| **BasePodView** | iPod chrome: nav bar with gradient/title/icons, click wheel with gesture recognizers |
| **Music** | Wrapper around `MPMusicPlayerController.systemMusicPlayer`; all library queries and playback |
| **ClickWheelProtocol** | Protocol for screen VCs to receive click wheel events |
| **Screen VCs** | Individual screens (Home, Music, Songs, Albums, etc.) implementing `ClickWheelProtocol` |

### Data Flow

1. **Input**: User touches on click wheel → `CircleGestureRecogniser` / `ForceTouchTapGestureRecogniser` / `UILongPressGestureRecognizer`
2. **Routing**: `BasePodView` determines which button zone was touched and calls the appropriate `ClickWheelProtocol` method on `clickWheelDelegate`
3. **Processing**: Active screen VC handles the event (scroll list, navigate, play/pause, etc.)
4. **Music Queries**: Screen VCs call `AppDelegate.music.getXxx()` methods which query `MPMediaQuery`
5. **Playback**: `PlaybackViewController` sets queue on `MPMusicPlayerController.systemMusicPlayer` and controls playback

### Navigation Flow

```
SplashScreenViewController → MainViewController
                              └── UINavigationController
                                   └── HomeViewController ("iPod")
                                        ├── Music → MusicViewController
                                        │   ├── Playlists → PlaylistsViewController → SongsViewController → PlaybackViewController
                                        │   ├── Artists → ArtistsViewController → AlbumsViewController → SongsViewController → PlaybackViewController
                                        │   ├── Albums → AlbumsViewController → SongsViewController → PlaybackViewController
                                        │   ├── Songs → SongsViewController → PlaybackViewController
                                        │   └── Genres → GenresViewController → SongsViewController → PlaybackViewController
                                        ├── Photos → (stub: MusicViewController)
                                        ├── Videos → (stub: MusicViewController)
                                        ├── Extras → (stub: MusicViewController)
                                        ├── Settings → (stub: MusicViewController)
                                        ├── Shuffle Songs → PlaybackViewController (all songs shuffled)
                                        └── Now Playing → PlaybackViewController (current queue)
```

## 3. Tech Stack

| Category | Technology |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pullerz/DankPod](https://github.com/Pullerz/DankPod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
