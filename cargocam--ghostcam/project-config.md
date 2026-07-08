---
trigger: always_on
description: This repo contains the open-source camera-side stack for [Ghostcam](https://ghostcam.fly.dev): a Go daemon that runs on a Raspberry Pi, the shared wire-contract types it uses to talk to a server, and the Debian / image-build glue that produces installable artefacts. The daemon captures H.264 + Opus via `rpicam-vid | ffmpeg`, uploads fMP4 HLS segments to S3 via server-presigned URLs, publishes live A/V via WHIP/WebRTC, and POSTs telemetry every 10 s. There's no server in this repo — the camera ta
---

# CLAUDE.md — Ghostcam Camera Development Guide

## What is this project?

This repo contains the open-source camera-side stack for [Ghostcam](https://ghostcam.fly.dev): a Go daemon that runs on a Raspberry Pi, the shared wire-contract types it uses to talk to a server, and the Debian / image-build glue that produces installable artefacts. The daemon captures H.264 + Opus via `rpicam-vid | ffmpeg`, uploads fMP4 HLS segments to S3 via server-presigned URLs, publishes live A/V via WHIP/WebRTC, and POSTs telemetry every 10 s. There's no server in this repo — the camera talks to one over HTTPS using the contract documented in `common/`.

The camera was Python from 2026-05-12 to 2026-05-14 (the `go-camera-rewrite` cutover). The Python port unblocked iteration on telemetry/upload/motion code but the live-relay slice (NAL framing + WebSocket transport) wanted to stay in Go, and once WHIP/pion was the chosen live wire format, Python was no longer earning its keep.

## Documentation Policy

When making changes to the codebase, **always update the relevant README and CLAUDE.md** to reflect those changes.

## Repository Layout

```
ghostcam/
├── camera/             Go camera daemon (package main). Single static linux/arm64
│   │                   binary; cross-compiled by release.yml. No cgo, no runtime
│   │                   deps beyond ffmpeg + rpicam-vid on the Pi.
│   ├── capture.go         ffmpeg orchestrator. rpicam-vid → tee → fMP4 segments +
│   │                      WHIP fanout. Synthetic mode uses testsrc2+sine.
│   ├── publisher.go       pion v4 WHIP client. Reads H.264 + OGG-Opus from named
│   │                      pipes, packetizes via media.Sample, posts to the
│   │                      configured server's WHIP endpoint. Multi-slice access
│   │                      units are coalesced via `first_mb_in_slice` detection.
│   ├── abr.go             Adaptive bitrate controller (opt-in via --abr). Samples
│   │                      pion's outbound packet loss, ratchets a 4-tier ladder
│   │                      (854×480 500k ↔ 1920×1080 4M) with fast-down /
│   │                      slow-up / cooldown, then trips requestPipelineRestart
│   │                      so capture respawns rpicam-vid at the new tier.
│   ├── firmware_stability.go Two-gate rollback (#106). After install, daemon
│   │                      touches /var/ghostcam/boot_ok on first healthy boot
│   │                      and increments healthy_minutes each minute thereafter;
│   │                      ExecStartPre rolls back to ghostcam-camera.prev when
│   │                      either gate is unmet after a fresh install.
│   ├── power_mode.go      Three power modes. `live` = always on; `standby` =
│   │                      capture runs but WHIP publisher only opens on a viewer
│   │                      Redis flag (saves ~50% cellular at idle); `sleep` =
│   │                      capture suspended, telemetry every 5 min for wake.
│   ├── battery_rules.go   Level-triggered evaluation of operator-supplied rules
│   │                      (lowest-threshold-wins) layered over the manual power
│   │                      mode.
│   ├── battery.go         BatteryReader interface + no-op default. Real drivers
│   │                      are registered at startup based on the
│   │                      GHOSTCAM_BATTERY_HAT env var.
│   ├── battery_pisugar_linux.go  PiSugar 3 / 3 Plus driver. Reads register 0x2A
│   │                      over /dev/i2c-1 at slave 0x57, polled every 30 s;
│   │                      cached %% feeds telemetry's battery_pct and the
│   │                      battery_rules evaluator.
│   ├── bt_onboarding_linux.go  GATT peripheral. Advertises `Ghostcam-<8hex>`,
│   │                      accepts the same provisioning JSON as the QR path.
│   │                      Raced with ScanQR in provisioning.go.
│   ├── sim_imsi_linux.go  Reads modem IMSI via `mmcli -m 0`, threads through
│   │                      ProvisionRequest so the server knows which SIM is
│   │                      bound to which camera.
│   ├── network_linux.go   nmcli wrapper. Connects WiFi from the BT-onboarding
│   │                      payload, sets autoconnect-retries=0 on every created
│   │                      connection so a single WPA rekey hiccup can't brick
│   │                      a headless wifi-only Pi.
│   └── sensors_*.go       Build-tag-gated. Real sensors on linux && !synthetic;
│   └── network_*.go       host-arch stubs everywhere else.
│   └── qr_*.go
├── common/             Shared Go types. Camera↔server contract: TelemetryEntry,
│                       QRPayload, ProvisionRequest/Response, PresignedURLs,
│                       CameraCommand, etc. The single source of truth for the
│                       wire. Other modules import as

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cargocam/ghostcam](https://github.com/cargocam/ghostcam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
