---
trigger: always_on
description: Single consolidated technical doc for the ESP32-TWAI-CAN library: API, usage
---

# ESP32-TWAI-CAN — Technical Reference (Agents & Developers)

Single consolidated technical doc for the ESP32-TWAI-CAN library: API, usage
patterns, and conventions. For the project/owner front page see `README.md`.
**Keep it in sync with the code** — the library is one header + one .cpp, so
there is no second doc to drift against.

## What This Library Does

Minimal Arduino-style wrapper over the ESP-IDF TWAI (CAN 2.0) controller
driver. `CanFrame` **is** `twai_message_t` on single-controller chips — there
is no conversion layer. `TwaiCAN` handles driver install/start/stop/uninstall,
speed/pin/queue configuration, status counters, and bus recovery. A global
`ESP32Can` singleton instance is provided and is what Gauge.S uses everywhere;
multi-controller chips (ESP32-C6) additionally get `CAN1`/`CAN2` instances
(see "Driver Paths").

**It does NOT**: decode protocols (OBD-II/KWP live in CarDataS), do acceptance
filtering beyond the config you pass in, or support CAN FD.

## Features

- Thin: `readFrame()`/`writeFrame()` map 1:1 to `twai_receive`/`twai_transmit`
- Sane defaults: 500 kbps, TX pin 5 / RX pin 4, TX/RX queues of 5,
  accept-all filter, normal mode
- `TwaiSpeed` enum with chip-conditional entries; `convertSpeed()` maps
  numeric kbps to the enum
- Re-entrant `begin()` — safe to call again to change speed/pins (calls
  `end()` internally first)
- Status: TX/RX queue depth, RX/TX error counters, missed/failed frame
  counters, bus error counter, controller state
- `recover()` / `restart()` helpers for BUS_OFF and STOPPED states
- Optional debug logging via `LOG_TWAI*` macros (zero cost by default)
- IRAM-safe read/write when `CONFIG_TWAI_ISR_IN_IRAM` is set

## Hardware

Requires an external 3.3V CAN transceiver (TI SN65HVD230 or similar
recommended). Tested on ESP32 and ESP32-S3; builds for any ESP32 variant with
a TWAI peripheral (S2, C3, ...). The speed-enum low-bitrate entries are
compile-time gated on `SOC_TWAI_BRP_MAX` / `CONFIG_ESP32_REV_MIN_FULL`.

## Driver Paths (legacy vs new, v1.1.0)

Selected at compile time in the header:

```cpp
#if defined(SOC_TWAI_CONTROLLER_NUM) && (SOC_TWAI_CONTROLLER_NUM > 1) && __has_include(<esp_twai_onchip.h>)
#define TWAI_CAN_NEW_DRIVER 1   // esp_twai.h handle-based driver (IDF 5.5+)
#else
#include "driver/twai.h"        // legacy single-instance driver
#endif
```

- **Legacy path** (ESP32, S2, S3, C3 — Gauge.S): existing frame/config API;
  `begin()` accepts custom `twai_filter_config_t*`/`twai_general_config_t*`/
  `twai_timing_config_t*`.
- **New path** (chips with >1 TWAI controller, currently ESP32-C6 —
  CanBridge.S): each `TwaiCAN` instance creates its own `twai_node_handle_t`
  via `twai_new_node_onchip()`; the driver hands out the next free controller,
  so two instances = both controllers. Differences:
  - `fConfig`/`gConfig`/`tConfig` params of `begin()` are IGNORED (accept-all
    filter; timing derived from the bitrate — exact 12.5 k support included).
  - RX: an internal FreeRTOS queue of `CanFrame` is filled from the ISR
    `on_rx_done` callback via `twai_node_receive_from_isr()`, preserving the
    blocking `readFrame(timeout)` semantics of the legacy path.
  - TX: the new driver reads the payload asynchronously, so `writeFrame()`
    copies frames into per-instance shadow slots freed from the `on_tx_done`
    ISR callback. Slot claims/releases are protected by a per-instance critical
    section, so task callers may write concurrently. Copying and driver waits
    are outside the lock. Serialize lifecycle/configuration against frame I/O.
  - `CanFrame` preserves named frame fields, NOT the legacy binary layout
    (the legacy header is not included). Do not cast or serialize it as
    `twai_message_t`. `TWAI_STATE_*` constants retain their numeric values.
  - `canState()` maps the new error states onto the legacy numbers and reports
    `TWAI_STATE_RECOVERING` while a `recover()` initiated recovery is pending.
    The state-change ISR clears completed recovery even if no task samples the
    intermediate running state before another bus-off. Recovery bookkeeping
    uses the existing per-instance lock; the IDF recovery command is nonblocking.
  - `inTxQueue()` counts busy TX shadow slots; `rxMissedCounter()` counts
    RX queue-full drops; `txFailedCounter()` counts failed/`txShadow`-starved
    writes.

## Types

### Additive Observation and Diagnostics

Existing frame I/O, lifecycle signatures, defaults, and `TWAI_STATE_*` numeric
values are retained. The new-driver path additionally provides:

```cpp
void setFrameObserver(TwaiFrameObserver observer, void* context);
bool getDiagnostics(TwaiDiagnostics* out);
```

Set the optional observer before `begin()`. It is called in ISR context for RX
and successfully completed TX, with a frame reference, a transmitted flag,
millisecond timestamp, and the caller's context. Copy the frame before returning;
do not retain its reference. Return whether a higher-priority task was woken.
The callback and its dependencies must be IRAM-safe when cache-safe TWAI ISR
support is enabled. No observer is installed by default.

Diagnostics distinguish accepted/completed/bus-failed/rejected TX and expose
the raw driver error state, RX drops, and error counters. Existing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [handmade0octopus/ESP32-TWAI-CAN](https://github.com/handmade0octopus/ESP32-TWAI-CAN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
