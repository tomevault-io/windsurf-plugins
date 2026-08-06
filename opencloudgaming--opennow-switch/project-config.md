---
trigger: always_on
description: 1. Keep streaming latency low and frame delivery bounded.
---

# AGENTS.md

## Core Priorities

1. Keep streaming latency low and frame delivery bounded.
2. Preserve reliability across reconnects, partial streams and decoder fallback.
3. Keep controller, touch and keyboard input predictable.

Choose correctness and recovery over a local shortcut. Performance changes must
retain bounded queues and must not weaken the existing resynchronization paths.

## Repository Layout

- `app/src/` contains the launcher, typed GeForce NOW API surface and shared
  application state.
- `app/src/gfn/` contains the GeForce NOW implementation, split by
  authentication, catalog, cloud sessions, regions and persistence.
- `app/src/webrtc/` contains signaling negotiation, media, input, diagnostics
  and peer-session lifecycle behind `webrtc_session.hpp`.
- `app/src/stream/` owns audio, video decoding and rendering. Deko3D is the
  Switch path; OpenGL is the host/reference path.
- `app/src/StreamView.cpp`, `stream_view_input.cpp` and
  `stream_view_overlay.cpp` own stream lifecycle/input dispatch, keyboard and
  NTE automation, and stream overlays respectively.
- `app/src/settings_tab*.cpp` splits settings layout/state, page construction,
  account actions and stream/preference actions.
- `resources/` contains RomFS fonts, translations, icons and UI assets.
- `tests/` contains small host-side policy and parsing tests.
- `scripts/build-switch-msys2.sh`, `build-switch.ps1` and
  `scripts/package-release.ps1` are the supported build and packaging entry
  points. `scripts/build-forwarder-installer.ps1` rebuilds the bundled HOME
  forwarder installer.
- User-facing documentation lives at `https://opennow.zortos.me`; keep the
  repository README focused on source, build and attribution information.
- `extern/` is vendored third-party code. Do not edit it unless a task
  explicitly requires an upstream dependency patch.

## Module Boundaries

- Keep NVIDIA/GFN HTTP, authentication, account persistence and catalog parsing
  behind `gfn_client.hpp`; implementations belong in the focused files under
  `app/src/gfn/`. UI views should consume typed models instead of rebuilding
  requests.
- Keep CloudMatch orchestration in `gfn/cloud_session.cpp`, request and response
  protocol handling in `gfn/cloud_session_protocol.cpp`, and trace/persisted
  active-session state in `gfn/cloud_session_state.cpp`. Shared declarations
  stay private in `gfn/cloud_session_internal.hpp`.
- Keep signaling and peer-connection state under `app/src/webrtc/`. WebSocket
  framing belongs in `WebSocketClient.*` and the `SignalingClient` wrapper.
- Keep stream-setting defaults and persistence in `stream_settings.*`; put
  independently testable selection rules in focused `*_policy.hpp` headers.
- Keep cover download, inspection and deletion in `cover_image_cache.*`;
  settings UI code must not traverse or mutate cache directories directly.
- Keep settings page construction separate from immediate account and
  stream/preference actions. Do not grow `settings_tab.cpp` back into a single
  all-purpose implementation.
- Keep platform-specific rendering behind `IVideoRenderer`. Do not introduce
  Deko3D details into UI code or assume OpenGL exists on Switch.
- Preserve the public data-channel report formats and the Xbox-compatible input
  contract when changing controller code.

## Streaming and Concurrency

- The network worker owns `peer_connection_loop`; the decoder worker owns decode
  submission; the UI thread polls signaling and renders completed frames.
- Keep `decoder_queue_` bounded. On congestion, dropping stale work and
  requesting a keyframe is preferable to growing latency.
- Do not hold `peer_mutex_` while performing unrelated file or UI work. Maintain
  the established lock ordering when touching peer, decoder queue or logging
  state.
- Avoid busy polling. If a worker has no work, use the existing backoff policy
  or a condition variable while keeping wake-up latency explicit.
- Diagnostic logging is opt-in. Do not add per-packet or per-frame logging to
  the normal gameplay path.

## Persistence and Security

- Runtime data belongs under the paths provided by `app_paths.*`; do not add
  repository-local account, token or log files.
- Cache paths must derive from `AppHomePath()` rather than repeating the current
  `sdmc:/switch/SwitchNOW` location.
- Keep account, settings and credential writes atomic and preserve backup
  recovery behavior.
- Never log bearer tokens, passwords, session cookies or decrypted credentials.
- TLS verification, authentication headers and persisted-data compatibility are
  security boundaries, not refactoring conveniences.

## Maintainability

- Prefer a small typed helper beside its owning subsystem over a broad utility
  module.
- Split large translation units by ownership and lifecycle, not arbitrary line
  count. Keep internal cross-file contracts in narrowly scoped `*_internal.hpp`
  headers and retain local-only helpers in anonymous namespaces.
- Reuse existing policy headers for behavior that can be tested without Switch
  hardware.
- Runtime-visible product version text lives in `app/src/app_version.hpp`.
  When releasing, keep CMake metadata, packaging defaults, README examples and
  the HOME forwarder installer version synchronized with it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenCloudGaming/OpenNOW-Switch](https://github.com/OpenCloudGaming/OpenNOW-Switch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
