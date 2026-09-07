---
trigger: always_on
description: This file is the single source of truth for agent behavior in this repository.
---

# Agent Instructions

This file is the single source of truth for agent behavior in this repository.

## Repository Structure and Build Output

- Keep build/output folders inside the owning project directory:
  - `AlpacaCore/build*`
  - `AlpacaHTTP/build*`
- Never create root-level ad-hoc build directories (examples to avoid: `build-synscan`, `build-temp`, `cmake-build-*` at repo root) unless explicitly requested.
- Keep generated artifacts out of source trees and avoid tracked build-system output files (Makefiles, CMake cache files, etc.) outside approved build folders.

## Core Architecture

- `AlpacaCore` is vendor-neutral Alpaca logic and device behavior only:
  - No HTTP/REST/sockets/JSON transport code.
  - Vendor SDKs isolated to `external/` + `src/vendors/<vendor>/` + `include/alpacacore/vendor/<vendor>/`.
- `AlpacaHTTP` is transport/routing/config/discovery only:
  - No vendor SDK use.
  - No duplicated device logic from `AlpacaCore`.
  - Web UI assets live in `AlpacaHTTP/web/` (HTML, CSS, JS). These are served by the HTTP server and packaged into the `.deb` at `/usr/share/alpacabridge/web/`.
- Call flow is always:
  - `AlpacaHTTP -> AlpacaCore driver -> vendor implementation`.

Supported device types (base drivers in `AlpacaCore/src/drivers/`): Camera, Telescope, FilterWheel, Focuser, Rotator, Dome, Switch, CoverCalibrator, ObservingConditions, SafetyMonitor. These are exactly the 10 ASCOM Alpaca device types — do not invent new top-level device types.

## Language, Style, and Safety

- C++20 preferred, RAII, small focused functions.
- Use `#pragma once` in headers.
- Prefer `enum class`, `std::chrono`, `std::string_view` where appropriate.
- No `using namespace std;` in headers.
- Core/driver layers should avoid heavy framework dependencies.
- License headers must remain AGPL-3.0-or-later and unmodified in all source files.

## Units and Behavior Conventions

- Exposure: seconds
- Angles: degrees
- RA: hours
- Dec: degrees
- Pixel size: microns
- Time: UTC with `std::chrono`

## Target Architecture

- **Linux arm64 only** (ARMv8 — Raspberry Pi 3B+/4/5, Rockchip SBCs, OrangePi, iOptron iMate). amd64/x86_64 is no longer supported, built, packaged, or validated. CMake, `debian/rules`, `build_and_run.sh`, and `install_alpaca_service.sh` all hard-fail on non-arm64 hosts.
- When writing driver code, follow fixed-width integer practices for protocol/SDK structs (`int32_t`, `uint16_t`, etc.) and avoid `long double`. The wider portability concerns (endianness, alignment) no longer matter for our build target, but using fixed-width types still makes wire-protocol code easier to read and harder to misread.
- ConformU validation is performed on arm64 only. Historical amd64/x64 ConformU reports have been deleted from `AlpacaCore/conformu/`.

## Driver Implementation Rules

- Use 3-layer driver pattern:
  1. Alpaca interface (`include/alpacacore/*_driver.h`)
  2. Vendor wrapper (`include/alpacacore/vendor/<vendor>/...`)
  3. Vendor implementation (`src/vendors/<vendor>/...`)
- Do not include raw vendor SDK headers outside wrapper implementation files.
- Convert vendor failures to `AlpacaException`.
- Gold-standard runtime semantics for drivers:
  - Async `connect()/disconnect()` with `get_connecting()`.
  - Synchronous `set_connected()` for compatibility.
  - Useful `get_device_state()` telemetry.
  - Clean thread/task shutdown in destructors — **[Driver concurrency &
    lifecycle](#driver-concurrency--lifecycle-read-before-writing-a-driver) is the
    single most important section in this file; every rule there was learned from a
    review round.**
- Add TODO comments where vendor protocol/SDK behavior is uncertain.

### Driver concurrency & lifecycle (read before writing a driver)

**ConformU rate-offset tests and the position model (PR #221, 2026-08-25).** ConformU measures
`RightAscensionRate`/`DeclinationRate` by sampling RA/Dec BEFORE the rate write and 10 s after it,
with a 5% tolerance — at the 0.05 arcsec/s low rate that is 0.025 arcsec over 10 s. Any position
discontinuity inside the rate setter fails it: a hardware re-anchor (`refresh_position_cache_locked(true)`)
on a MOVING axis shifts the reported position by up to one encoder count (~0.31 arcsec on the Wave
100i) plus start latency, which read as a 27% RA "rate" error, while the stationary Dec axis passed.
Rate setters must re-anchor the dead-reckoning model in place (`anchor_model_locked()`), never on
hardware. Also: ConformU runs ON the SBC over localhost — the dev VM's LAN path has 2-90 ms spikes
that stamp constant `Can*` getters with 0.10x s FAST marks — and a Bash tool timeout kills a child
ConformU mid-slew, so launch it detached (`setsid nohup`) and poll a done marker. Motor-controller
mounts store no site: set SiteLatitude/Longitude first or ConformU aborts "below the horizon".

**Apply this checklist up front.** ConformU is single-threaded and catches *none*
of the races below — code review plus the TSan concurrency stress suite do
(`[stress]` tests under the `sanitizers-tsan` CI job / `RUN_TSAN=1` pre-flight,
issue #101); a miss that neither catches becomes a review round. The rules are
vendor-agnostic; do them in the driver from the start.

**Threads & shutdown**
- Async connect: inherit the shared base —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-astro/AlpacaBridge](https://github.com/open-astro/AlpacaBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
