---
trigger: always_on
description: Architecture context for working in this repo. See also `README.md` for the
---

# CLAUDE.md

Architecture context for working in this repo. See also `README.md` for the
flashing guide, and the original plan in
`~/.claude/plans/la-idea-es-hacer-optimized-gem.md` if you need the full
reasoning behind the design decisions.

## What this is

ESP32 firmware (Seeed reTerminal E1001) that, every hour: connects to WiFi,
reads its battery, requests an already-rendered image from the `my-assistant`
backend (`../my-assistant` in this same checkout), paints it on the 800×480
4-gray e-ink panel, and goes back to sleep. No real `loop()`: `setup()` does
the entire cycle and always ends in `esp_deep_sleep_start()` — with one
exception: if there's no saved WiFi/endpoint config yet (or the user has
wiped it with a physical reset gesture), `setup()` enters a provisioning
portal (SoftAP + web) instead of the normal cycle — see "Design decision:
initial provisioning" below.

## API contract (my-assistant)

`GET {API_BASE_URL}/api/v1/display?battery=<1-100>`, header
`Authorization: Bearer <API_AUTH_TOKEN>`. Source of truth:
`../my-assistant/internal/display/codec.go`.

Response `application/octet-stream`, all big-endian:

```
offset  size  field
0       4     magic "EINK"
4       1     format version (1)
5       2     width  (uint16 BE) -> 800
7       2     height (uint16 BE) -> 480
9       1     bits per pixel (2)
10      ...   packed pixels, 2 bits/pixel, 4 pixels/byte, MSB-first,
              row-major, no row padding. 0=black, 1=dark gray,
              2=light gray, 3=white.
```

800×480 → exactly 96000 bytes of payload (96010 total), verified by
spinning up the real server (`go run ./cmd/server` in `../my-assistant`) and
doing a real `curl` during this firmware's development — not just reading
the code.

**The response comes with no `Content-Length`, using `Transfer-Encoding:
chunked`** (confirmed with `curl -D -` against the real server): Go's
`net/http` stops being able to precompute `Content-Length` as soon as the
handler writes more than the small internal buffer it uses to decide that,
and a ~96KB body always exceeds it. This was discovered on the first real
flash (it failed with `TOO_LARGE` because `HTTPClient::getSize()` returns
`-1` with no `Content-Length`). That's why `display_client.cpp` doesn't use
`getSize()` at all: it reserves a fixed-size buffer (`HTTP_MAX_RESPONSE_BYTES`)
in PSRAM and uses `HTTPClient::writeToStream()` with its own `Stream`
(`MemoryStream`) that dumps into it — `writeToStream()` requires a
`Stream*`, not a `Print*`, even though it only ever writes into it (hence
`MemoryStream`'s empty read-side methods). It decodes chunked encoding
transparently, so it would work the same if the backend ever switched to a
fixed `Content-Length`.

`display_client.cpp` validates this byte by byte before ever touching the
display (magic, version, bpp, exact dimensions, payload length). It rejects
any width/height other than 800×480: the physical panel can't show any
other resolution, so there's no point trying to support one.

## Design decision: "raw" driver instead of GxEPD2/Seeed_GFX

`eink_driver.cpp` is a direct port (not a wrapper) of Seeed's official
example
[`examples/base/GxEPD2_reTerminal_E1001_Gray4`](https://github.com/Seeed-Projects/OSHW-reTerminal-Series-E-D/blob/main/examples/base/GxEPD2_reTerminal_E1001_Gray4/GxEPD2_reTerminal_E1001_Gray4.ino)
in `Seeed-Projects/OSHW-reTerminal-Series-E-D`. Despite the example's name,
**it doesn't use GxEPD2 or Seeed_GFX** — it's a hand-rolled UC8179 driver
over plain SPI.

**Why**: that example packs its framebuffer (`Gray4Canvas`) exactly the same
way `codec.go` does — 2bpp, 4px/byte, MSB-first, row-major, no padding, same
0=black..3=white convention. That means **the HTTP response body can be
passed straight through** to the panel's bit-plane upload routine
(`eink::drawFrame`), with no decoding into an intermediate framebuffer and
no dragging in a large library (GxEPD2/TFT_eSPI) just for "one full-screen
refresh per hour." The grayscale LUT tables and UC8179 command sequence
(`0x01` power, `0x30` PLL, `0x82` VCOM, `0x06` booster, `0x04` power-on,
`0x00` panel setting, `0x61` resolution, `0x20-0x24` LUTs, `0x10`/`0x13`
DTM1/DTM2, `0x12` refresh, `0x02`/`0x07` sleep) are copied verbatim from
Seeed's example — they're opaque calibration data, not something to derive
by hand.

**Difference from the vendor version**: the original example has no
timeout on waiting for the BUSY pin (`checkBusy()` can hang forever). Here,
`waitBusy()` has a bounded timeout (`EINK_BUSY_TIMEOUT_MS`, 15s) — a stuck
panel can't leave the device hung, draining the battery indefinitely.

`BOARD_SCREEN_COMBO = 520` (UC8179, 800×480) is the constant Seeed uses
across all its official examples to identify this specific panel.

## Design decision: time — RTC (PCF8563) + SNTP correction

The E1001 has a hardware **PCF8563** RTC (I2C 0x51, SCL=GPIO20, SDA=GPIO19)
with its own coin-cell battery, independent of the ESP32's internal RTC
(which only survives *deep sleep*, not a full reset/EN or a total loss of
power).

Approach (`main.cpp` + `rtc_pcf8563.cpp`):
1. On boot: read the PCF8563 → `settimeofday()` immediately (fast, no
   network needed). If the VL (voltage-low) flag is set, the time isn't
   trustworthy and is ignored.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pianista215/e1001-my-assistant-firmware](https://github.com/pianista215/e1001-my-assistant-firmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
