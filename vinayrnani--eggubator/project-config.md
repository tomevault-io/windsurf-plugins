---
trigger: always_on
description: Single-file Arduino firmware for a DHT22-based egg incubator with web UI, flash data logging, and OTA updates. All HTML/CSS/JS lives inside `web_ui.h` as PROGMEM string literals — edit raw HTML embedded in C.
---

# AGENTS.md — Egg Incubator (ESP8266)

Single-file Arduino firmware for a DHT22-based egg incubator with web UI, flash data logging, and OTA updates. All HTML/CSS/JS lives inside `web_ui.h` as PROGMEM string literals — edit raw HTML embedded in C.

## Build & Deploy

```bash
# Compile
arduino-cli compile -b esp8266:esp8266:nodemcu -j "$(nproc)" --build-path build/.cache --output-dir build eggubator.ino
cp build/eggubator.ino.bin firmware.bin

# Deploy OTA (user insists on this script only)
./deploy.sh                          # compile + find IP via ping + curl to /update

# Flash USB (Termux/OTG)
./flash.sh                           # auto-detect /dev/ttyUSB*, uses esptool.py

# Full release (bump → compile → commit → tag → GitHub Release → OTA)
./rel.sh [VERSION]                   # auto-increment patch if no VERSION arg
./rel-nd.sh [VERSION]                # same without OTA deploy
```

- `rel.sh` bumps `updates.h` + `version.txt`, compiles, creates GitHub Release with `firmware.bin`, then deploys OTA.
- `firmware.bin` is `.gitignore`d but release scripts `git add` it explicitly.
- `version.txt` must match `FIRMWARE_VERSION` in `updates.h`.

## Hardware Conventions

| Component | Pin | Active |
|-----------|-----|--------|
| Heater    | D1  | LOW = ON |
| Atomizer  | D2  | LOW = ON |
| Fan       | D3  | LOW = ON |
| Servo     | D5  | PWM (544-2450µs) |
| DHT22     | D4  | — |

**Relays are active LOW.** `digitalWrite(pin, HIGH)` = OFF, `LOW` = ON. Getting this wrong can overheat or damage hardware.

Network: mDNS `eggubator.local`, static IP ends in `.72`. Falls back to AP mode (`EGGubator` SSID).

## WiFi — Async State Machine (not blocking)

`wifi_manager.h` uses 5 states — no blocking `delay()` in init:

```
WF_TRY_SAVED → (10s timeout) → WF_TRY_DEFAULT → (10s timeout) → WF_AP
    ↕                               ↕                                  ↕
WF_CONNECTED ← (auto-reconnect) → WF_RECONNECTING (15s grace)   scan every 15s for saved/default SSID
```

- Boot order: `EEPROM.begin(512)` → `loadWifiCredentials()` → `initWiFi()` (non-blocking). Must stay in this order.
- `initWiFi()` returns immediately; `handleWiFi()` drives state from `loop()`.
- Priority: saved EEPROM creds → compile-time defaults (`config.h`) → AP fallback.
- AP mode runs a DNS captive portal and scans every 15s for known networks.
- `MDNS.begin()` deferred to first WiFi connection in loop (not setup).
- Saving WiFi creds via web (`/settings/api?wifiSsid=X&wifiPassword=Y`) writes EEPROM at addr 200 — does NOT reconnect. User must reboot.

## EEPROM Layout

| Region | Address | Magic | Size |
|--------|---------|-------|------|
| SAT drift | 15-23 | — | 9 bytes |
| DeviceSettings | 40 | `0xAB` | 112-byte struct |
| WiFi credentials | 200 | `0xAC` | `WifiSettings` (98 bytes) |

- `loadWifiCredentials()` reads addr 200 — if magic invalid or SSID empty, keeps compile-time defaults.
- `saveWifiCredentials()` writes addr 200.
- Flash logging uses **separate** circular buffer at `0x200000` (256 sectors × 4096 bytes) — EEPROM untouched by logging.

## Verification

1. **Compile** — must succeed with zero errors.
2. **Browser** — `http://eggubator.local/`, dashboard loads, `/status` returns JSON.
3. **Manual Playwright tests** at `test/playwright/test_*.js` — `node test_xxx.js` (device must be reachable).
4. **Mock mode** (no hardware): `/settings/api?enable=1&temp=37.5&hum=55` or `/settings/api?autosim=1`.

## Embedded Assets (no internet needed)

5 gzipped libraries compiled into firmware via `embedded_assets.h`:

| Asset | Path |
|-------|------|
| Chart.js 4.4.7 | `/lib/chartjs/chart.umd.min.js` |
| Dexie.js 3.2.2 | `/lib/dexie/dexie.min.js` |
| Hammer.js | `/lib/hammerjs/hammer.min.js` |
| chartjs-plugin-zoom | `/lib/chartjs-plugin-zoom/chartjs-plugin-zoom.min.js` |
| Bootstrap 5 CSS | `/lib/bootstrap/css/bootstrap.min.css` |

- Served with `Content-Encoding: gzip` and `Cache-Control: public, max-age=31536000, immutable`.
- Registered as routes in `setup()` via `EMBEDDED_ASSETS` table.
- Web UI (`web_ui.h`) loads these paths — they resolve locally, not from CDN.

## SVG Icons & Animation Quirks

- 4 stat cards (heater, atomizer, fan, servo) use **Font Awesome solid SVG paths** with `fill="currentColor"`.
- **SVG class assignment must use `setAttribute('class', ...)`** not `.className =` — SVG's `SVGAnimatedString` silently ignores string assignment.
- Fan rotation uses CSS animation `.svg-spin` with `transform-box: fill-box` for reliable centering.

## File Map

| File | Purpose |
|------|---------|
| `eggubator.ino` | Setup/loop, web handlers, auto-control, EEPROM save/load |
| `config.h` | Pin defs, compile-time WiFi defaults, hysteresis |
| `dht_sensor.h` | DHT22 read + physics simulation (mock/auto-sim) |
| `wifi_manager.h` | Async WiFi state machine + DNS captive portal |
| `logging.h` / `.cpp` | Flash circular buffer (256 sectors at `0x200000`) |
| `sat_manager.h` / `.cpp` | Boot session tracking, absolute time recovery across reboots |
| `updates.h` | OTA check + download from GitHub releases |
| `web_ui.h` | Single file: all HTML/CSS/JS as PROGMEM strings |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vinayrnani/eggubator](https://github.com/Vinayrnani/eggubator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
