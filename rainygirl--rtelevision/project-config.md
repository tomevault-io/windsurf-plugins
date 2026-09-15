---
trigger: always_on
description: Notes for anyone - person or coding agent - working on the code. End-user
---

# AGENTS.md

Notes for anyone - person or coding agent - working on the code. End-user
installation instructions live in `README.md` (and its Italian, Japanese and
Korean translations); keep those free of developer detail. Everything else
belongs here.

## What the project is

R Television plays free TV streams from around the world (the
[iptv-org](https://github.com/iptv-org/iptv) playlist): a channel tree on the
left, video on the right.

```
+---------------------------+----------------------------------+
| search                    |                                  |
| [ category | country ]    |              video               |
| v General            2733 |                                  |
|   v News              985 |                                  |
|     KR Arirang TV         +----------------------------------+
|   v Sports            429 | ❚❚  ■  ★   Arirang TV   🔊──  ⤢  |
+---------------------------+----------------------------------+
 11,035 channels · offline cache · 2026-09-12 10:06
```

One portable C++17 core, one native front end per platform:

| Platform | Front end | Media backend | Bundled |
|---|---|---|---|
| macOS arm64 / x86_64 | Cocoa (Objective-C++) | libVLC 3.0.23 | yes, 343 plugins |
| Linux x86_64 | GTK3 | libVLC 3.0.9 from the distribution | yes, 353 plugins |
| Haiku x86_64 / x86 | BeAPI | libVLC 3.0.23 | yes, unpacked from HaikuPorts |
| Haiku arm64 | BeAPI | FFmpeg 6.1.2 | yes, cross-built |

VLC never has to be installed by the user; every build carries its own media
library.

## Repository layout

```
shared/core/          portable C++17: no UI, no platform SDK calls
  Channel.h           the channel model
  M3UParser           #EXTINF and #EXTVLCOPT parsing
  PlaylistStore       downloading and the offline cache policy
  ChannelIndex        category tree, country grouping, search
  Favorites           favorites, persisted
  AppController       ties store, index, favorites and player together
  MediaPlayer         backend interface, VideoFrameSink, AudioSink
    VlcMediaPlayer      libVLC
    FFmpegMediaPlayer   FFmpeg, where there is no libVLC (Haiku arm64)
    NullMediaPlayer     no playback; everything else still works
  HlsRelay            local relay for live streams slower than their bitrate
  RelayedMediaPlayer  MediaPlayer decorator that routes eligible HLS through it
  HttpClient          HTTP interface
    CurlHttpClient      libcurl (macOS, Linux)
    HaikuHttpClient     Haiku network services
  Paths, Country      data directories, flags, country names
  Strings             the interface text in four languages
shared/tools/         rtv-cli, the headless driver
shared/common.mk      core source lists and flags for every platform Makefile
platforms/macos/      Cocoa front end, Makefile, install.sh, fetch-libvlc.sh
platforms/linux/      GTK3 front end, Makefile, install.sh, fetch-libvlc.sh
platforms/haiku/      BeAPI front end (BSoundPlayer audio), Makefile, install.sh
resources/            seed playlist, app icon and the script that draws it
scripts/              build-ffmpeg-haiku.sh
third_party/          vendored libVLC / FFmpeg trees (fetched or built)
docs/PORTING.md       porting history and measurements (Korean)
licenses/             full LGPL-2.1 and GPL-2.0 texts
```

## Architecture rules

- `shared/` has no UI code, no `#import`, no platform SDK calls. POSIX (files,
  sockets, threads) is fine; it is common to all three targets.
- A platform picks its media and HTTP backends by naming one source file each
  in its Makefile (`CORE_SRC_VLC` / `CORE_SRC_FFMPEG` / `CORE_SRC_NULL`,
  `CORE_SRC_HTTP_CURL` / `CORE_SRC_HTTP_HAIKU` in `shared/common.mk`). No
  `#ifdef` in the core for backend selection.
- Core callbacks (`MediaPlayer::StateCallback`, `AppController::RefreshCallback`,
  relay progress) arrive on worker threads. Front ends hop to their UI thread
  first: `dispatch_async(main_queue)` on macOS, `g_idle_add` on Linux,
  `BMessenger::SendMessage` on Haiku.
- All Makefiles build with `-MMD -MP` and include the `.d` files. Without it a
  changed header silently leaves stale objects with the wrong struct layout.

| Boundary | File | macOS | Linux | Haiku |
|---|---|---|---|---|
| Data directory | `core/Paths.cpp` | `~/Library/Application Support/RTelevision` | `$XDG_DATA_HOME/RTelevision` | `~/config/settings/RTelevision` |
| Video output | `core/MediaPlayer.h` | `attachVideoView(NSView*)` | `attachVideoView(XID)` | `attachVideoSink(VideoFrameSink*)` |
| HTTP | `core/HttpClient.h` | `CurlHttpClient` | `CurlHttpClient` | `HaikuHttpClient` |
| Display language | `core/Strings.h` | `[NSLocale preferredLanguages]` | `LANG` / `LC_ALL` | `BLocaleRoster` |

## Channel list and offline cache

The list comes from `https://iptv-org.github.io/iptv/index.m3u` (override with
`RTV_PLAYLIST_URL`) and is kept in three places so the app survives that URL
going away:

1. the cache, `playlist.m3u`
2. the last copy known to be good, `playlist.bak.m3u`
3. a snapshot shipped inside the app, `resources/seed-playlist.m3u`

The window opens from the cache and refreshes in the background, so it works
with no network. Downloads are conditional on `ETag` and `Last-Modified`. A

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rainygirl/rtelevision](https://github.com/rainygirl/rtelevision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
