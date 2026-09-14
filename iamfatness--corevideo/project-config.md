---
trigger: always_on
description: Project notes for Claude Code sessions working in this repository: CoreVideo,
---

# CLAUDE.md

Project notes for Claude Code sessions working in this repository: CoreVideo,
the OBS Studio plugin that pulls Zoom meeting video and audio into OBS as
native sources. Current release: **v0.1.44** (2026-08-22). Update this file in
the same change as any substantive work — docs-updated is part of done.

## Architecture in one paragraph

Two processes. The OBS plugin (`src/`, `obs-zoom-plugin.dll`) and a standalone
engine (`engine/src/`, `ZoomObsEngine.exe`) that links the Zoom Meeting SDK.
They talk over two named pipes (`ZoomObsPlugin_P2E` / `_E2P`, one JSON object
per line; wire format and all shared structs live in `src/engine-ipc.h`) and
move media through named shared-memory regions (`ZoomObsPlugin_<uuid>_video` /
`_audio` / `_share`, capped at `kMaxShmSources = 32` across all three kinds).
The plugin always launches the engine; the engine never outlives it on purpose
(see "process hygiene" below). Media **events are prompts, not payloads**: a
video event means "read the newest frame in the region", an audio event means
"drain everything pending in the ring" — this property is load-bearing for the
dispatch design and must survive refactors.

## macOS (Apple Silicon)

Reunified 2026-08-20 (merge of the `mac-port` line). Same two-process design;
the differences that matter: IPC rides Unix sockets (`/tmp/ZoomObsPlugin_*.sock`)
instead of named pipes; the engine is `ZoomObsEngine.app` inside the plugin
bundle with the Zoom SDK copied into its `Contents/Frameworks` (the SDK loads
sibling bundles from there — rpath does not work); install with
`scripts/make-macos-bundle.sh --build-dir <dir> --link-sdk --install`, never by
hand. POSIX shm names are collapsed to 20 chars on Apple (`shm_platform_name`,
PSHMNAMLEN=31) and a shm object is **sized exactly once** (ftruncate on an
existing object is EINVAL — create-over-held fstats and recreates instead).
The engine's SDK-facing rules are documented in `engine/src/main-macos.mm`:
never subscribe to the local user's own share, detach `renderer.delegate`
before `destroyRender:` (synchronous callback re-entry deadlocked the main
thread for hours), frame callbacks `try_lock` and drop rather than block SDK
threads, heartbeat pings route through the main queue only after
`initSDKWithParams` returns. `ShmAudioSlot::capture_ns` crosses processes on
DIFFERENT clocks on macOS (see the struct comment) — `audio_latency_us` is
untrustworthy there until that is reconciled. Kill a mid-meeting engine only
after checking `{"cmd":"status"}` — it IS the meeting session.

The macOS meeting-status callback must emit `awaiting_admission` on every SDK
status change. `ZoomSDKMeetingStatus_WaitingForHost` and
`ZoomSDKMeetingStatus_InWaitingRoom` map to `active:true`; every other status
maps to `active:false`. The dock holds its existing 120-second join watchdog
while that flag is true and starts a fresh full window after Zoom advances.
Keep the symbolic mapping in `engine/src/macos-admission-state.h`, where the
SDK-backed `CoreVideoMacosAdmissionState` test compiles it against the installed
framework. The real callback calls that header's dispatch seam inside the
fresh-callback/epoch gate; the regression records the seam's outgoing event,
asserts it precedes normal status handling, and drives the watchdog with it.

**Talkback does not exist on macOS**, and the dock says so rather than failing
quietly. `engine-talkback.cpp` is in `ENGINE_SOURCES`, which only the Windows
engine target uses; the macOS engine is `main-macos.mm` and never compiles it.
The dock is cross-platform and builds either way, so without a gate it is a
panel whose every control sends a command nothing answers. One constant,
`kTalkbackPlatformSupported` (`src/zoom-talkback-panel.cpp`), feeds
`TalkbackDockSessionView::platform_supported`,
`TalkbackDockKeyContext::platform_supported`, and the Assign/probe buttons'
own `setEnabled`. Three rulings worth keeping. The `#if defined(__APPLE__)`
lives at that ONE call site and the decision crosses into
`talkback-dock-state.h` as a plain bool, because that header is Qt/OBS-free and
compiles everywhere — so the macOS rendering is pinned by a Windows or Linux CI
run, which is the only way a macOS-only branch is tested by anything this
project runs. `TalkbackDockBannerState::Unavailable` is checked FIRST in
`talkback_dock_banner()` and RETURNS, which is what keeps the "coming to macOS"
wording out of the ON AIR strip structurally — with no talkback engine nothing
can key, so nothing can be live, and Unavailable and Live are unreachable
together by construction rather than by promise. In the key chain it sits
directly below `held_here` and nowhere else: never disabling a button the
operator is holding is the stronger law, and honouring it here costs nothing,
because there is no key to hold. The layout instrument
(`COREVIDEO_TALKBACK_LAYOUT_TEST`) is deliberately NOT gated — its job is to
render every state including the tallest live banner, and gating it would
collapse it to the Unavailable strip on the very platform a developer is most
likely running it on. Mutation-proved in
`tests/talkback-dock-state-test.cpp`: disabling either half of the gate fails
its own assertions, and a default-constructed context must stay supported or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamfatness/CoreVideo](https://github.com/iamfatness/CoreVideo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
