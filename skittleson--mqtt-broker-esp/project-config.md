---
trigger: always_on
description: This file is read first by AI coding agents (Claude, Cursor, Aider, etc.)
---

# AGENTS.md — working with this repo as an AI coding agent

This file is read first by AI coding agents (Claude, Cursor, Aider, etc.)
before they make changes. It documents the conventions a human contributor
would learn the hard way over a few PR cycles. Follow it.

If something here disagrees with `README.md` or `docs/`, the human-facing
docs win — update this file to match, don't drift.

---

## 1. What this project is

A standalone **MQTT 3.1.1 broker** that runs entirely on an ESP32-S3,
written in C against ESP-IDF v5.5. No external MQTT library — the broker
is a single C codebase using lwIP BSD sockets. Ships a Tasmota-style web
portal, an SNTP server, Tasmota-style scheduled-publish timers, and an
optional Ethernet+NAPT gateway mode.

**Design goal:** 10-year deployment lifetime on a $10 chip with zero
maintenance. Every change should preserve that — see §7 ("Constraints").

---

## 2. Build & deploy loop

The canonical iterate-on-the-device workflow is:

```bash
# 1. Enter ESP-IDF environment (once per shell)
source $IDF_PATH/export.sh
# or, if IDF_PATH isn't set:
source /home/spencerkittleson/lvgl_micropython/lib/esp-idf/export.sh

# 2. Build
idf.py build
# Output binary: build/mqtt_broker.bin
# Track binary size against the previous tag — see §7.

# 3. Deploy via OTA to the live test device
PORTAL_AUTH=support:dockyard make ota
# Wraps: fetch /api/csrf with auth → POST build/mqtt_broker.bin to
# /ota-upload?csrf=<token>. Device auto-reboots on success.

# 4. Verify it came back on the new firmware
sleep 12 && curl -s -u support:dockyard http://192.168.22.100/api/status \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['firmware'])"
```

**Live test device:** `http://192.168.22.100/` (Waveshare ESP32-S3-ETH).
Portal Basic Auth: `support:dockyard`. Treat as the source of truth for
"does this actually work" — `idf.py build` succeeding is necessary but
not sufficient.

**Never flash via USB** in normal iteration. OTA is the canonical path
because it's what users will use, and it catches regressions (partition
table, app-format, rollback) that USB flash skips.

---

## 3. Code map

```
main/
  main.c            Entry. WiFi → portal → ntp → timers → broker, in that order.
  mqtt_broker.{c,h} The broker. Owns clients[], subs[], retained store, $SYS publishes,
                    and the thread-safe publish queue (broker_publish_local /
                    tester pub queue). Single broker_task — almost everything
                    runs in select() loop.
  mqtt_parser.{c,h} Wire-format encode/decode for the MQTT 3.1.1 packets.
  portal.{c,h}      The HTTP server. ~3700 lines. Route dispatch by string
                    match on req.path. CSRF + Basic Auth on all gated routes.
                    Page-render uses snprintf into a 16 KB heap buffer.
  portal_ws.{c,h}   /tester WebSocket (live MQTT subscribe + publish via UI).
  ntp.{c,h}         SNTP client + SNTPv4 server. Owns the POSIX TZ env via
                    setenv("TZ", ns.tz, 1) + tzset() at boot.
  timers.{c,h}      Tasmota-style scheduled publishes (16 slots, 1 Hz task).
                    Stores a compact JSON blob in NVS "mqtt_cfg"/"timers".
                    Resolves local time via newlib's localtime_r so DST is free.
  tz_presets.{c,h}  Curated ~40-entry table of POSIX TZ strings for the
                    /settings dropdown. Hand-maintained from IANA tzdata.
  wifi_connect.{c,h} WiFi station + AP. Owns NVS "mqtt_cfg" credentials.
  eth_connect.{c,h} Optional W5500 SPI Ethernet + NAPT (CONFIG_MQTT_BROKER_ETHERNET).
  csrf.{c,h}        Per-boot random token. csrf_verify(req.csrf_header, req.body)
                    is required for every state-changing request.
  version.h         FW_VERSION, FW_NAME. Bumped per release.

tools/              Python scripts (Playwright/pytest helpers).
  capture_*.py      Headless screenshot generators → docs/screenshots/.
                    Always parameterized via PORTAL_URL + PORTAL_AUTH env.

tests/              (none yet — test_broker.py / test_ntp.py live at repo root)
test_broker.py      pytest. Drives mosquitto_pub/sub against the live device.
test_ntp.py         pytest. Asserts SNTP client + server behaviour.

docs/
  api.md            Path/method/auth table + JSON schemas + curl examples.
  architecture.md   Task layout, memory map, partitions, scaling notes.
  screenshots/      Portal UX screenshots, grouped by feature subdir.
  *-audit-*.md      Per-version UX audit reports + fix sequencing.
plan-*.md           Design docs for upcoming features. Written before code.
changelog/          One file per release: CHANGELOG-v<X.Y.Z>.md.
CHANGELOG.md        Top-level summary that points at changelog/*.
```

---

## 4. NVS namespaces (avoid clobbering)

Read-write zones with their owners. Do not write keys outside an
established namespace without updating this table.

| Namespace  | Owner          | Notable keys                                                                                                                                                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skittleson/mqtt_broker_esp](https://github.com/skittleson/mqtt_broker_esp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
