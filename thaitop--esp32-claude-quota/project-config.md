---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A desk display for Claude Code's two rate-limit windows, plus weather and crypto, running on the ESP32-2432S028R ("Cheap Yellow Display": ESP32-WROOM-32, 2.8" 240x320 ILI9341, XPT2046 resistive touch), drawn with LVGL 9. Two halves:

- **`firmware/`** — C++/Arduino for the ESP32 (PlatformIO).
- **`bridge/`** — Python on the Mac running Claude Code. Fetches utilization from claude.ai and serves it over the LAN.

Read these before non-trivial work — they carry the reasoning the code assumes:
- **`CONTEXT.md`** — the shared vocabulary both halves use (Quota Window, Utilization, Trust, Staleness, Snapshot, Feed, Ramp, …). Terms have one name on both sides of the wire; keep it that way, and honour the `_Avoid_` lists.
- **`docs/adr/`** — the four decisions that constrain everything else (see Invariants below).
- **`README.md`** — build/flash/run and troubleshooting for someone starting from bare hardware.

## Commands

Firmware (run from `firmware/`; `pio` lands at `~/.local/bin/pio`):

```bash
~/.local/bin/pio run                       # build
~/.local/bin/pio run --target upload       # build + flash (auto-picks port)
~/.local/bin/pio device monitor            # serial @ 115200
~/.local/bin/pio run -t clean              # REQUIRED after editing include/lv_conf.h (PlatformIO caches the LVGL build)
```

Redirect uploads to a file and check the exit code — `pio` writes hundreds of lines and a piped `tail`/`grep` in an agent shell truncates from the wrong end, returning the header (dep graph + memory table) while the `SUCCESS` line never arrives, which looks exactly like a failed flash:

```bash
~/.local/bin/pio run --target upload > /tmp/upload.log 2>&1; echo "exit=$?"
grep -E "Hash of data|Hard resetting|SUCCESS|FAILED" /tmp/upload.log
```

Bridge (use a Python with a working TLS trust store — Homebrew's, **not** the python.org build):

```bash
python3 bridge/fetch_usage.py --check      # verify key file paths/permissions, no key printed
python3 bridge/fetch_usage.py              # write bridge/usage-cache once
python3 bridge/fetch_usage.py --interval   # loop on a 60s timer (the fetcher's production mode)
python3 bridge/fetch_usage.py --raw        # print the real claude.ai payload (for debugging field renames)
python3 bridge/quota_bridge.py             # serve /quota + /history on :8787
python3 bridge/quota_bridge.py --once      # print one payload and exit
python3 bridge/quota_bridge.py --no-tokens # skip the jsonl transcript scan (much faster)
```

There is **no test suite and no host simulator** in the repo despite `model.h`/`https.h` comments describing host-compilable seams — that layering is real (see below) but the harness that would exercise it is not checked in.

## Architecture

### Two processes on the Mac, deliberately split (do not merge them)

- **`fetch_usage.py`** is the only component that ever holds the `sessionKey` cookie (a full account credential). It hits the undocumented `GET https://claude.ai/api/organizations/<org-id>/usage` on a 60s timer and writes a KEY=VALUE cache (`bridge/usage-cache`). The key is read from `~/.config/claude-quota/session-key` (mode 600, outside the repo) on **every** pass, so an expired key is fixed by pasting a fresh one — no restart.
- **`quota_bridge.py`** reads that cache and serves it as JSON on the LAN. It makes **zero outbound calls** and touches **no credential** — a file reader with an HTTP interface.

The split decouples the display's poll rate (20s) from the claude.ai call rate (60s): a second display or a faster poll changes nothing about what leaves the machine. Folding the fetch into the bridge would tie them together and put a credential in the LAN-facing process.

### Firmware layering (`firmware/src/`)

The seam is **`model.h`**: the `AppModel` struct and its snapshots. `net/` fills these structs in; `ui/` reads them. Neither side knows about the other — `ui/` never includes `<WiFi.h>` or HTTP; `net/` never touches LVGL. `model.h` deliberately uses `<cstdint>` not `<Arduino.h>` so it could compile on the host. Preserve this: when a screen needs a fact about the radio or a feed, route it through the model (see `recordLink()` in `main.cpp`), don't reach across the seam.

- **`net/poller.cpp`** — round-robin scheduler. Runs **at most one fetch per pass**, for whichever feed is most overdue. `main.cpp` calls `net::service()` each loop.
- **`net/bridge.cpp`** — the Bridge Feed (quota + history). History runs on its own 10-min sub-timer *inside* the Bridge Feed's slot, so there is still only one request in flight ever.
- **`net/https.cpp`** — shared one-shot TLS GET for Weather/Crypto/Stock; tears the client down the moment the body parses.
- **`net/stock.cpp`** — the Stock Feed. Finnhub's quote endpoint takes one symbol, so this fetches **one ticker per pass** and advances a cursor — the same "sub-schedule inside one slot" trick as bridge history, keeping one request in flight (ADR-0003). Needs a soft `FINNHUB_TOKEN` (ADR-0004).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thaitop/esp32-claude-quota](https://github.com/thaitop/esp32-claude-quota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
