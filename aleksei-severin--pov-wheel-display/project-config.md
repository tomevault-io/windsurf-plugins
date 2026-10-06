---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
Если мой запрос на русском языке, то отвечай тоже на русском. Комментарии к коду должны быть тоже на русском языке.
Если у тебя есть предложения, или ты не согласен с моим предложением - напиши об этом и предложи лучший вариант реализации, опиши его сильные стороны.
В конце пиши очень краткий комментарий для коммита на английском по всем внесенным в код изменениям, если таковые были.

## Project Overview

POV (Persistence of Vision) Wheel Display — an ESP32-S3 embedded system that drives 528 SK9822-A addressable LEDs across a 6-arm spinning rotor. Six Hall effect sensors (one per arm) synchronize LED rendering to rotation and detect rotation direction. Features a web UI and OTA updates.

**Hardware revision: V5.** Earlier revisions used a BQ25792 charger, BH1750 lux sensor, ICM45605 IMU, an I2C bus, a TXU0104 level shifter, a DRV5032 wake sensor, a single Hall sensor, and a configurable 1–8 arm count. All of that is gone — do not reintroduce it. The arms now radiate exactly from the axis centre, so the per-LED angular correction for an off-axis spoke (the old **Hub Offset** setting, `global_spoke_offset`) is gone too: every LED on a face shares one angle, and `fillSectorIntoBuffer()` resolves the box weights once per face instead of once per LED.

## Build & Upload Commands

```bash
# Build
pio run -e cable

# Upload via USB (firmware + LittleFS image)
pio run -e cable --target upload
pio run -e cable --target uploadfs

# Upload OTA to specific devices
pio run -e wheel_dc14 --target upload
pio run -e wheel_3 --target upload    # mDNS: pov-wheel-5e6f.local

# Serial monitor
pio device monitor -e cable -b 115200
```

No automated tests exist. Validation is done via the serial monitor and the web UI at the device IP or `http://<hostname>.local`. `pio run -e cable` is the fastest compile-level check.

## Architecture

### Rendering Pipeline

1. Six Hall sensors trigger `hallInterruptHandler()` (shared handler, sensor index passed via `attachInterruptArg`). One fixed magnet on the fork + six rotating sensors = **6 events per revolution, 60° apart**.
2. The ISR is integer-only (no FPU in ISRs on Xtensa). It records per-sensor timestamps, the full-revolution period, and votes on rotation direction from the firing order.
3. `renderingTask` (Core 1, prio 2) turns those events into a continuous rotor angle:
   `sector0 = anchor_deg + ω·Δt + ½·α·Δt² + angle_offset`
   - **ω** comes from the interval between two firings of the *same* sensor — exactly one revolution, so sensor placement error cannot bias it. Six sensors give six staggered revolution measurements per turn.
   - **α** (angular acceleration) is estimated from successive ω measurements. This is what keeps the image from drifting under acceleration/braking — without it the error is ½·α·T², tens of degrees on hard braking. **The estimate is smoothed adaptively (`HALL_ALPHA_K` / `HALL_ALPHA_K_FAST`), and the adaptive part is the whole point.** A plain slow filter kills the noise and the real acceleration with it — α is precisely what stops the image drifting under acceleration and braking, and lagging it by a revolution defeats it exactly where it matters. So the filter is slow on a steady spin and opens as soon as a reading disagrees with the current estimate by more than `HALL_ALPHA_FAST_DEG` of per-revolution correction. That threshold comes from a reading on real hardware, and the first attempt at deriving it was wrong by a factor of fifty: a PLL residual of ~0.3° suggested raw-α noise near 5° per revolution, so the threshold was set to 6 — and a live wheel then logged `aT2 = 0.1` on a steady spin. The sensors are orders of magnitude better than that estimate implied, so a threshold of 6 essentially never opened, the filter stayed slow permanently, and it throttled exactly the acceleration it is adaptive for. 0.5 separates the measured noise (0.1) from any real acceleration with room to spare.

**The PLL has a second stage for the same reason.** On a steady spin the residual is small and the gentle quarter-pull is right — it smears residual sensor spread instead of stepping six times a revolution. Under acceleration the residual turns *systematic*, and a first-order loop against a constant residual settles at `(1-K)/K` = 3× that residual — which is the drift seen while speeding up. Above `HALL_PLL_ERR_FAST` (1°, comfortably above the measured 0.3°) the gain rises to `HALL_PLL_K_FAST`, leaving steady-state behaviour untouched. The `aT2` field in the log is that quantity — well under the threshold on a steady spin, clearly above it while accelerating.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aleksei-severin/POV_Wheel_Display](https://github.com/aleksei-severin/POV_Wheel_Display) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
