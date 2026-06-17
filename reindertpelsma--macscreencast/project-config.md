---
trigger: always_on
description: This file is for future agents (and humans) working on the codec / rate-control
---

# macscreencast — agent notes

This file is for future agents (and humans) working on the codec / rate-control
stack. The Apple VideoToolbox behavior here is non-obvious and we burned several
iterations on it; this is the cliff notes so nobody repeats them.

## What this project is (one-paragraph orientation)

**macscreencast is a browser-based macOS remote desktop at up to 60fps,
delivered over an SSH tunnel with no third-party relay.** The Python server
runs on a Mac and exposes a WebSocket on `127.0.0.1:6081`; the user opens
`http://localhost:6081/?token=…` after `ssh -L 6081:localhost:6081`. Frames are
captured via ScreenCaptureKit (the GPU compositor, 60fps) when Screen
Recording + Accessibility have been granted, and via Apple's `screensharingd`
(VNC, ~20fps) as a bootstrap path before those permissions exist. Input is
posted via `CGEventPost(kCGHIDEventTap)` (native HID-level) on the SCK path,
or RFB events on the VNC path. The browser decodes H.264/H.265 in WebCodecs
(hardware media engine), so client-side CPU is near-zero.

## Why it exists

Every browser-based macOS remote desktop tool has the same problems: noVNC at
2fps (ZRLE-in-JS), `screensharingd` freezes/HID-idles, broken clipboard on
macOS 15+, broken modifier keys after reconnects. This project goes below
`screensharingd` (SCK + CGEvent) and uses the browser's hardware video
decoder — that combination is the moat.

## Who it's for

- **Cloud-Mac renters** (AWS EC2 Mac, MacStadium, Scaleway, Hetzner, OakHost):
  the strongest fit. SSH-only access, no Mac client to install, no relay.
- **CI / build-farm operators** debugging flaky `macos-latest` runners — see
  `docs/ci.md` for the runner pattern (no LaunchAgent, no bundle).
- **Linux / Windows / ChromeOS users** who need to drive a Mac. Apple Screen
  Sharing is Mac-to-Mac only; this works from anything with a browser.
- **Compliance-sensitive orgs** that can't route through TeamViewer / AnyDesk
  / Chrome Remote Desktop third-party servers.

Less clearly: regular users on their own home Mac (Apple's built-in Screen
Sharing or Tailscale + VNC will usually be fewer moving parts).

## High-level architecture (orient before reading code)

- `server.py` — CLI entry point, asyncio event loop, WebSocket server.
- `mvs/sck.py` — ScreenCaptureKit capture (PyObjC); the 60fps path.
- `mvs/vnc.py` — RFB client to `screensharingd`, ZRLE decode, HID-idle
  workarounds. The bootstrap path.
- `mvs/encoder.py` — VideoToolbox H.264/H.265 (via PyAV).
- `mvs/congestion.py` — the rate-control state machine (the file most of the
  CLAUDE.md notes below relate to).
- `mvs/handler.py` — WebSocket session: ping/pong, capture loop wiring,
  `caffeinate` lifecycle, browser→server input dispatch.
- `mvs/keepalive.py` — compositor keepalive subprocess; prevents WindowServer
  from throttling to ~3Hz on idle screens (affects SCK and VNC equally).
- `mvs/caffeinate.py` — refcounted `caffeinate -di` lifecycle (per WS client).
- `frontend/` — single-page app, WebCodecs decoder, audio, clipboard.
- `setup.sh` / `install.sh` — install/upgrade flows. Tahoe `gui/$UID`
  fallback to foreground `--vnc-only` lives here. See `docs/vnc-bootstrap.md`.
- `tests/tcp_throttle.py` — real-TCP throttle harness (exercises congestion
  controller under realistic backpressure, not mocks).

The user-facing docs are split across `README.md` + `docs/{vnc-bootstrap,
configuration, performance, security, clipboard, ci}.md`; read those for the
"why this design" framing before changing observable behavior.

---

## Current architecture (as of f816736)

- **Encoder**: `hevc_videotoolbox` (default) or `h264_videotoolbox`,
  `constant_bit_rate=0` (VBR). The encoder treats `bit_rate` as a target
  average and OVERSHOOTS 2-5× on complex content — this is documented Apple
  behavior, but it's a real architectural gap for a real-time encoder.
- **Rate enforcement** lives at the network layer, not the encoder layer:
  the controller's wb-aware drain pause (`congestion.py:on_lag` →
  `_drain_until`) clears the queue when lag accumulates, then resumes.
- **GOP**: 99999 (essentially never). The decoder is fully state-locked
  with the encoder over TCP — no broadcast scenario, no need for I-frame
  refresh. Only forced I-frame is at encoder rebuild (codec change /
  resolution change), which is unavoidable.
- **Disconnect prevention**: ping_monitor (handler.py) MUST NOT close
  the connection on pong timeout. On a constrained link, the WS-layer
  ping queues behind buffered video and arrives many seconds late;
  closing on this signal produces a 7-second disconnect-loop in real
  Chrome with DevTools 2Mbps throttle. Use the timeout as a congestion
  signal (`ctrl.on_lag(30000, 0)`) but keep the connection alive.
  Liveness is the browser-side 15s stall detector + the websockets
  library's own 20s/20s keepalive.

## Apple VideoToolbox: things that DON'T work, validated

### 1. `constant_bit_rate=1` (CBR for H.264)

Apple added this in macOS 13 for `h264_videotoolbox` only (not HEVC, not AV1).
Documented as strict CBR via `kVTCompressionPropertyKey_DataRateLimits`.

**Reality**: on simple/medium-complexity content, the encoder collapses
output to ~10% of target rather than padding to the target rate. Verified
on:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reindertpelsma/macscreencast](https://github.com/reindertpelsma/macscreencast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
