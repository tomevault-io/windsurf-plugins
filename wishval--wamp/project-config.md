---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

This is a macOS app built with Xcode. Open `Wamp.xcodeproj` and build/run from Xcode, or use:

```bash
xcodebuild -project Wamp.xcodeproj -scheme Wamp -configuration Debug build
```

Run the test suite:

```bash
xcodebuild -project Wamp.xcodeproj -scheme Wamp -destination 'platform=macOS' test
```

No linter and no CI/CD are configured. Tests cover `Models/`, `CueSheet/`, and a persistence round-trip only — `AudioEngine`, UI views, `HotKeyManager`, and `Skinning/` rendering are deliberately out of scope (see `docs/superpowers/specs/2026-04-12-testing-design.md`). Test fixtures live in `WampTests/Fixtures/` (sample audio + cue files).

## Architecture

Pure Swift/Cocoa (AppKit) macOS audio player replicating classic Winamp 2.x. No SwiftUI, no storyboards, no XIBs — all UI is programmatic. Zero external dependencies; uses only Apple frameworks (AVFoundation, Combine, Accelerate, MediaPlayer).

### Project Structure

```
Wamp/
├── AppDelegate.swift        — nib-less bootstrap (static func main()), owns singletons & window
├── Audio/
│   └── AudioEngine.swift    — AVAudioEngine graph: PlayerNode → 10-band EQ → Mixer → Output
├── CueSheet/
│   ├── CueSheet.swift           — model: tracks, INDEX positions, encoding metadata
│   ├── CueSheetParser.swift     — parses external `.cue` files
│   ├── CueDecoder.swift         — encoding detection (UTF-8, Shift-JIS, CP-1251, CP-1252)
│   ├── FlacCueExtractor.swift   — pulls embedded CUESHEET from FLAC Vorbis comments
│   └── CueResolver.swift        — expands a cue into virtual `Track`s pointing at the same file
├── Models/
│   ├── PlaylistManager.swift — track list, current index, shuffle, repeat, auto-advance
│   ├── StateManager.swift    — JSON persistence to ~/Library/Application Support/Wamp/
│   ├── Track.swift              — audio file model with metadata parsing via AVURLAsset
│   ├── M3UParser.swift          — `.m3u` / `.m3u8` parser (EXTM3U/EXTINF, BOM, CRLF, Latin-1 vs UTF-8)
│   ├── JumpFilter.swift         — pure prefix → word-boundary → substring ranking for Jump-to-File
│   ├── ITunesLibraryXML.swift   — parser for `iTunes Music Library.xml` exports
│   └── AppleMusicLibrarySource.swift — `ITLibrary`-backed Music.app library reader
├── Skinning/
│   ├── SkinManager.swift        — atomic skin lifecycle (load / unload / publish)
│   ├── SkinModel.swift          — parsed skin (sprites, regions, colors, viscolors)
│   ├── SkinParser.swift + helpers — `.wsz` ZIP unpacking, `IniParser`, `RegionParser`,
│   │                                `ViscolorsParser`, `EqGraphColorsParser`, `PlaylistStyleParser`,
│   │                                `SkinParserUtils` (ZIP/image/nums_ex utilities)
│   ├── SpriteCatalog.swift      — sprite slicing from `main.bmp`, `cbuttons.bmp`, etc.
│   ├── TextSpriteRenderer.swift — bitmap-font text rendering from `text.bmp` / `nums.bmp`
│   ├── SkinProvider.swift       — protocol + `BuiltInSkin` fallback (no skin loaded)
│   └── WinampClassicSkin.swift  — `SkinProvider` impl backed by a parsed `SkinModel`
├── UI/
│   ├── MainWindow.swift      — fixed-width (275px) borderless window with Double-Size scaling
│   ├── MainPlayerView.swift  — time display, volume/balance sliders, transport controls
│   ├── EqualizerView.swift   — 10-band EQ sliders + presets + EQ response curve
│   ├── PlaylistView.swift    — table with drag-drop, search, keyboard nav, double-click-to-play
│   ├── JumpToFileWindow.swift                — Cmd+J incremental search over the playlist
│   ├── ImportMusicLibraryWindowController.swift — sheet for picking Music.app sources to import
│   ├── WinampTheme.swift     — all design tokens (colors, sizes, fonts)
│   └── Components/
│       ├── TitleBarView.swift    — window title bar with pin/minimize/close buttons
│       ├── TransportBar.swift    — play/pause/stop/prev/next buttons
│       ├── LCDDisplay.swift      — retro LCD time display
│       ├── SevenSegmentView.swift — seven-segment digit renderer
│       ├── SpectrumView.swift    — real-time spectrum analyzer visualization
│       ├── EQResponseView.swift  — EQ frequency response curve
│       ├── PlayStateIndicator.swift — play/pause/stop glyph next to the LCD
│       ├── PlaylistSkinScroller.swift — custom NSScroller drawing the skinned thumb from `pledit.bmp`
│       ├── AngularLegacyScroller.swift — flat NSScroller for the unskinned playlist (matches angular chrome)
│       ├── WinampButton.swift    — themed button component
│       └── WinampSlider.swift    — themed slider component
└── Utils/
    └── HotKeyManager.swift   — media keys (play/pause/next/prev) & Now Playing info
```

### Data Flow

`AppDelegate` owns the core singletons and wires them together (it also keeps `@main` plus an explicit `static func main()` — see Key Patterns):

- **AudioEngine** (`ObservableObject`) — playback, 10-band EQ, spectrum data (32 bins via Accelerate), volume/balance/mute
- **PlaylistManager** (`ObservableObject`) — track list, shuffle, repeat modes (off/track/playlist), auto-advance on track finish

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wishval/wamp](https://github.com/wishval/wamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
