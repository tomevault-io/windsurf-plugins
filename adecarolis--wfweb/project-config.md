---
trigger: always_on
description: Headless Qt5 C++ app for controlling amateur radio transceivers (Icom, Kenwood, Yaesu).
---

# wfview / wfweb Development Guide

## Project Overview
Headless Qt5 C++ app for controlling amateur radio transceivers (Icom, Kenwood, Yaesu).
The only user interface is web-based (browser). There is no desktop GUI.
Built from `wfweb.pro`.

---

## Build

```bash
# Linux - ALWAYS use wfweb.pro, NOT wfview.pro
qmake wfweb.pro && make -j$(nproc)

# Flags: -b (daemon), -l (logfile), -s (settings file)
```

See `BUILDING.md` for platform-specific prerequisites (Linux/Windows/macOS).

---

## Architecture

### Core Signal Chain
```
WebSocket commands (from browser)
    -> cachingQueue (deduplication, prioritization)
    -> rigCommander (command encoding, CI-V protocol)
    -> serial port (USB) or icomUdpHandler (LAN)
    -> radio
```

### Key Components
| Component | Role |
|-----------|------|
| `wfmain` | Main app logic, web server init |
| `cachingQueue` | Command queue with dedup and priority |
| `rigCommander` | CI-V command encoding/decoding |
| `webServer` | HTTP + WebSocket server (separate QThread) |
| `audioConverter` | Codec conversion (rig format <-> PCM) |
| `icomUdpHandler` | LAN UDP transport (3 channels) |
| `radeProcessor` | RADE V1 modem encode/decode (separate QThread) |
| `freedvProcessor` | Classic FreeDV codec (700D/700E/1600, separate QThread) |
| `freedvReporter` | Socket.IO client for FreeDV Reporter (qso.freedv.org) |
| `rade_text` | Self-contained RADE EOO callsign encoder/decoder (C) |

### Web Server
- HTTP/HTTPS on port 8080, WebSocket on 8081 (configurable via `WebServerPort` in prefs)
- Set port to 0 to disable; enabled by default
- Frontend: vanilla JS SPA in `resources/web/`, bundled via Qt resources (`resources/web.qrc`)
- Backend: `src/webserver.cpp`, `include/webserver.h`
- REST API documented in `REST_API.md`

### Audio Binary Protocol
| Direction | MsgType | Format |
|-----------|---------|--------|
| RX (rig->browser) | `0x02` | `[0x02][0x00][seq_u16LE][rateDiv_u16LE][PCM_Int16LE...]` |
| TX (browser->rig) | `0x03` | `[0x03][0x00][seq_u16LE][reserved_u16LE][PCM_Int16LE...]` |

### Port Conventions
- 8080: HTTPS/web browser (with audio/mic support)
- 8081: HTTP REST API (scripts/microcontrollers)
- 50001-50003: UDP rig server mode

---

## Critical API Patterns (cachingQueue)

These patterns have caused bugs before. Follow them exactly:

```cpp
// ALWAYS use addUnique() for set commands (not add())
// add() creates duplicates; addUnique() deduplicates by command type
queue->addUnique(queuePriority::priorityImmediate, funcType, payload);

// Gain values MUST be QVariant::fromValue<ushort>(), NOT <uchar>()
queue->addUnique(pri, funcSetAfGain, QVariant::fromValue<ushort>(level));

// modeInfo.data MUST be explicitly set to 0 (no data mode)
// The default 0xff is INVALID and causes mode selection failures
modeInfo.data = 0;

// modeInfo.filter should default to 1 (FIL1)
modeInfo.filter = 1;
```

---

## Key Files

| File | Purpose |
|------|---------|
| `wfweb.pro` | Qt project file |
| `src/wfmain.cpp` | Main app, web server init |
| `src/webserver.cpp` | Web server backend |
| `include/webserver.h` | Web server header |
| `include/prefs.h` | Preferences struct (contains `webPort`) |
| `resources/web/index.html` | Web frontend SPA |
| `resources/web.qrc` | Qt resource file for web assets |
| `resources/ft8ts/dist/ft8ts.mjs` | FT8/FT4 decoder module |
| `src/radeprocessor.cpp` | RADE V1 modem processing |
| `src/freedvprocessor.cpp` | Classic FreeDV codec processing |
| `src/freedvreporter.cpp` | FreeDV Reporter Socket.IO client |
| `src/rade_text.c` | RADE EOO text encoder/decoder (self-contained C) |
| `include/rade_text.h` | RADE text public API |
| `include/spotreporter.h` | Base class for reporter services |
| `CHANGELOG` | Release changelog |

---

## FreeDV / RADE Architecture

### Processing Threads
- `freedvProcessor` and `radeProcessor` each run in their own QThread
- Cross-thread communication uses Qt signal/slot (queued connections)
- For synchronous cross-thread calls (e.g., EOO generation on PTT-off), use
  `BlockingQueuedConnection` with a shared member variable — do NOT use `Q_RETURN_ARG`

### RADE EOO Callsign Flow
- **TX**: `prepareTxEooBits()` encodes callsign via `rade_text_generate_tx_string()` on first TX frame.
  On PTT-off (`setPTT: false`), `generateEooAudio()` is called synchronously via `BlockingQueuedConnection`,
  the EOO audio is appended to `freedvTxBuffer`, and radio unkey is delayed 300ms so the frame plays out.
- **RX**: `rade_rx()` detects EOO → `rade_text_rx()` decodes → `rxCallsign` signal → webserver → browser
- **Important**: The callsign arrives AFTER sync is lost (EOO is the last frame). Do not gate
  reporter or UI updates on `freedvSync`.

### FreeDV Reporter
- Socket.IO Engine.IO v4 client over QWebSocket → qso.freedv.org
- Connects ONLY when FreeDV/RADE mode is active; disconnects on mode switch to SSB
- Reporter settings (callsign, grid, enabled) are saved regardless of mode
- RADE mode reports as `"RADEV1"` to the reporter service

### Web UI Command Flow
- `enableMic` — mic on/off (ALSA setup/teardown, DATA MOD switch)
- `setPTT` — radio TX on/off (CI-V command). EOO generation happens here, NOT in enableMic
- These are separate commands: mic stays enabled across PTT cycles


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adecarolis/wfweb](https://github.com/adecarolis/wfweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
