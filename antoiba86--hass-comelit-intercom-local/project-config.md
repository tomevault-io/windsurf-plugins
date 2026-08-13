---
trigger: always_on
description: Home Assistant custom component for the **Comelit 6701W** WiFi video intercom. Communicates entirely locally via the **ICONA Bridge TCP protocol** on port 64100 — no cloud dependency.
---

# Comelit Local — Project Guide

## Overview

Home Assistant custom component for the **Comelit 6701W** WiFi video intercom. Communicates entirely locally via the **ICONA Bridge TCP protocol** on port 64100 — no cloud dependency.

## Project Structure

```
custom_components/comelit_intercom_local/
  __init__.py        — HA integration setup; registers both card JS static paths + Lovelace resources
  config_flow.py     — UI config flow with auto token extraction + options flow (enable_notifications)
  coordinator.py     — DataUpdateCoordinator; owns shared TCP client, RTSP server, video session, VIP listener, keepalive loop
  button.py          — Door open + Start/Stop video button entities; door button stops video after 10s delay
  camera.py          — Camera entity; is_streaming property; stop-video + state-change callbacks
  event.py           — Doorbell ring / missed call event entities
  protocol.py        — Wire protocol: 8-byte header, message types, binary payloads
  channels.py        — Channel definitions (UAUT, UCFG, CTPP, PUSH)
  client.py          — AsyncIO TCP client for ICONA Bridge; TCP keepalives; 120s read timeout
  auth.py            — Authentication flow (UAUT channel)
  token.py           — Token extraction from device HTTP backup endpoint
  config_reader.py   — Device configuration retrieval (UCFG channel)
  ctpp.py            — Shared CTPP init/handshake sequence (ctpp_init_sequence); used by door, video, VIP listener
  door.py            — Door open: open_door_fast (reuse open CTPP) + open_door_standalone (transient channel)
  push.py            — Push notification listener (PUSH channel); send_push_keepalive
  vip_listener.py    — Persistent VIP event listener on CTPP channel: doorbell_ring, door_opened, renewal ACK
  camera_utils.py    — Camera/RTSP URL discovery
  video_call.py      — Video call signaling on shared client; owns/borrows CTPP; async_open_door_on_ctpp
  rtp_receiver.py    — UDP/TCP RTP receiver: H.264 FU-A→PyAV→JPEG + PCMA audio routing; IDR logging
  rtsp_server.py     — Local RTSP server: H.264; RTCP Sender Reports; PLAY gating; disconnect_clients
  models.py          — Data models (Door, Camera, DeviceConfig, PushEvent)
  exceptions.py      — Custom exceptions
  const.py           — Constants (domain, platforms, defaults)
  www/
    comelit-intercom-card.js   — Custom Lovelace card (play-button UI, auto-stop on navigation)
    comelit-doorbell-card.js   — Doorbell notification card (ring alert, Answer/Dismiss, live stream)

tests/
  test_protocol.py        — Unit tests for wire protocol
  test_client.py          — Unit tests for TCP client
  test_ctpp.py            — Unit tests for ctpp_init_sequence
  test_door.py            — Unit tests for open_door_fast / open_door_standalone
  test_rtp_receiver.py    — Unit tests for RTP receiver
  test_rtsp_server.py     — Unit tests for RTSP server
  test_video_call.py      — Unit tests for video call session
  test_video_signaling.py — Unit tests for video signaling protocol
  test_camera.py          — Unit tests for camera entity
  test_coordinator.py     — Unit tests for coordinator
  test_vip_listener.py    — Unit tests for VIP event listener (39 tests)
  test_event_entity.py    — Unit tests for doorbell event entity (14 tests)
  test_button.py          — Unit tests for button entities
  test_push.py            — Unit tests for push channel
  test_integration.py     — Integration tests (requires real device)
  conftest.py             — Shared fixtures

postman/             — Postman collection documenting HTTP + TCP requests
```

## Setup & Development

**Requirements:** Python 3.11+, Home Assistant 2024.1+ (for HA integration)

**Always use `uv` for Python** — never use `pip` or `python3` directly.

```bash
# Install dev dependencies
uv pip install -e ".[dev]"

# Run unit tests (no device needed)
PYTHONPATH=. uv run python -m pytest tests/test_protocol.py tests/test_client.py tests/test_ctpp.py tests/test_door.py tests/test_rtp_receiver.py tests/test_rtsp_server.py tests/test_video_call.py tests/test_video_signaling.py tests/test_camera.py tests/test_coordinator.py tests/test_vip_listener.py tests/test_event_entity.py tests/test_button.py tests/test_push.py -v

# Run integration tests (requires real device on LAN)
COMELIT_HOST=192.168.1.111 COMELIT_TOKEN=<token> uv run python -m pytest tests/test_integration.py -v -s
```

## ICONA Bridge Protocol

All communication is raw TCP on port **64100**. Every message has an 8-byte header:

```
[0x00 0x06] [body_length LE16] [request_id LE16] [0x00 0x00]
```

### Channels and Flow

1. **UAUT** — Authentication: open channel → send JSON access request with token → expect code 200
2. **UCFG** — Configuration: request config → parse doors, cameras, apt_address
3. **PUSH** — Notifications: registers FCM token; also used as keepalive probe (re-send push-info every 90s — device ACKs with JSON, resetting the idle timer)
4. **CTPP** — Persistent channel for VIP events (doorbell ring, door opened) and door control; shared across VIP listener, video session, and standalone door open (see Door Control below)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antoiba86/hass-comelit-intercom-local](https://github.com/antoiba86/hass-comelit-intercom-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
