---
trigger: always_on
description: A cross-platform video player built with Flutter.
---

# DreamPlayer

A cross-platform video player built with Flutter.

## Goal

A video player app supporting:
- **Android** (primary, tested on user's Android phone — CPH2573, Android 16) and **iOS/iPad** (user's iPad Pro M2)
- **All audio codecs**: DTS, DTS-HD, E-AC3, AC3, TrueHD, etc.
- **Dolby Vision** where the display supports it
- **FFmpeg-based** decoding engine

## Current status

- App **UI skeleton** done (library, player, settings; dark theme).
- **HDR / codec on-screen display** done (Dolby Vision, HDR10+, HDR10, SDR; E-AC3, DTS-HD, TrueHD, AAC, ...).
- **Responsive layout** — no overflow on phones/tablets/landscape/large text.
- **Native refresh rate** selected at startup (verified 120 Hz on device).
- **DOLBY VISION PLAYBACK WORKS on Android via ExoPlayer/Media3 PlatformView.**
  Verified on-device: the DV P8 test file (`dolby-vision-people`) decodes on the
  Qualcomm hardware **`c2.qti.dv.decoder`** at 4K 3840x2160@60 fps with zero
  dropped frames, correct colors (no mpv pink/green), audio via
  `c2.dolby.eac3.decoder` / Media3 `FFmpegAudioRenderer`. Implementation:
  native `SurfaceView` PlayerView in a Flutter `AndroidView` (hybrid-composition
  fallback keeps its own SurfaceFlinger layer → real HDR to the display) +
  `MethodChannel`/`EventChannel` per view. `ExoPlayerController.open()` issued
  before the platform view attaches is queued and flushed in `_attach`.
  **Gotcha fixed:** the backend must `setState` after creating the controller,
  or the buttons/video layer stay frozen in the pre-init state.
- **iOS/iPad playback via AetherEngine (2026-08)** — the raw **AVPlayer**
  platform view was swapped for an **AetherEngine**-backed one
  (`ios/Runner/AvPlayerView.swift`, `UiKitView` on the Dart side) behind the
  exact same `dreamplayer/exo_<id>` method/event channel contract, so the Dart
  `ExoPlayerController` is unchanged. AetherEngine adds what AVPlayer alone
  cannot: **FFmpeg demux of MKV/TS/AVI/WebM**, **DTS/DTS-HD/TrueHD/E-AC3 audio**
  (AudioToolbox + libavcodec), **Dolby Vision / HDR10(+) via the native AVPlayer
  path** for Apple containers. `engine.bind(view:)` mounts `AetherPlayerView`
  (own `AVPlayerLayer` → real HDR where the panel supports it; iPad Pro M2
  does). Engine added as an SPM dependency (`project.pbxproj`, pinned
  `upToNextMajorVersion` from 6.21.0) — Xcode auto-resolves FFmpegBuild's
  dynamic FFmpeg xcframeworks into the app bundle. **CI-green** (run on commit
  `82b3dd9`). **Verified on-device (2026-08):** local/Documents files AND SMB streams
  play on the iPad Pro M2 via the former in-app SMB browser (AMSMB2) — since
  hidden from the UI (2026-08-13, see "SMB / network shares"), NAS playback is
  via CX/Files "Open with".
  **Minimum iOS 18.0** (`IPHONEOS_DEPLOYMENT_TARGET = 18.0`; builds for
  iOS 18 through the latest, iPhone and iPad).
  - Channel mapping: state 1/2/3/4 (idle/buffering/ready/ended); DV surfaces as
    `dvhe.<profile>.06` so Dart's `dv`-prefix detection fires; `colorTransfer`
    6 for HDR10/10+/DV, 7 for HLG. Audio/subtitle tracks pushed via
    `currentTracks`; `selectAudioTrack`/`selectSubtitleTrack`/`clearSubtitle`
    mapped 1:1 to engine calls.
  - **SMB audio-track switch fix (2026-08)**: `selectAudioTrack` on an SMB
    stream used to fail with "Demuxer: open failed (Operation not permitted
    (-1))". AetherEngine's reload reuses the RETAINED custom `SMBIOReader`
    (`keepCustomReader: true`); the old session teardown calls
    `SMBIOReader.cancel()` which marks the CURRENT in-flight read cancelled —
    when it lands on the new probe's first read, that read aborts with -1
    (EPERM). `AvPlayerView.setAudioTrack` now detects SMB streams
    (`isSMBStream`/`smbToken`) and calls `reopenSMBStream(audioIndex:)`
    instead: `SMBClient.reconnect(for:)` mints a FRESH `SMBConnection` on the
    same server/share/path (swapping it into the registry, returning the
    displaced stale connection), then `engine.load(source:startPosition:
    options:audioSourceStreamIndex:)` rebuilds at the current playhead with the
    requested track. The stale connection is closed only AFTER the engine
    finishes swapping readers, so the running session is never interrupted
    mid-teardown. All SMB readers use `ownsSource: false` — SMBClient owns
    connection lifetime (closed on `closeShare`).
  - **SMB buffering / read-ahead fix (2026-08)**: while SMB audio switching was
    fixed, SMB streams still hit the buffering spinner mid-playback. Root cause:
    `SMBIOReader` bridges every FFmpeg read to a SYNCHRONOUS SMB round-trip
    (256 KB per read, zero prefetch), so a Wi-Fi latency spike starves the
    engine's loopback HLS producer and AVPlayer drops into
    `waitingToPlayAtSpecifiedRate. `AvPlayerView` now loads all SMB streams
    through a new `BufferedSMBReader` (`ios/Runner/BufferedSMBReader.swift`), a
    read-ahead sliding-window `IOReader` (32 MiB window / 4 MiB chunks fetched
    on a background `Task.detached` prefetch): demux reads are served from
    memory while the next chunk streams from the NAS, so latency only bites on
    a seek or a full window drain. Same idea as Nova's 48 MB ring buffer. It
    keeps `SMBIOReader`'s lifecycle contract (`cancel()` only bumps a cancel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mangeshghodke/DreamPlayer](https://github.com/mangeshghodke/DreamPlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
