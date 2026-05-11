---
trigger: always_on
description: This guide defines how an AI coding agent should propose, write, and review firmware for this project (an Arduino/ESP32-based OLED flight display). It emphasizes **validated assumptions**, **production-readiness**, and **hardware-aware optimizations**.
---

# AGENTS.md — Engineering Agent Operating Guide (Arduino / ESP32)

This guide defines how an AI coding agent should propose, write, and review firmware for this project (an Arduino/ESP32-based OLED flight display). It emphasizes **validated assumptions**, **production-readiness**, and **hardware-aware optimizations**.

---

## 1) Agent Mission & Scope

- **Mission:** Produce maintainable, production-ready C++/Arduino code and docs that run reliably on target boards and displays with tight memory/CPU budgets.
- **Scope:** Rendering, parsing, networking, data caching, and device I/O for an OLED flight display running on ESP32-class hardware.

---

## 2) Operating Principles

1. **Assumption Discipline**
   - Minimize assumptions. When a blocking ambiguity exists, state the question **and** proceed with the best default, clearly labeled as:  
     `ASSUMPTION: <what>  • RISK: <impact>  • HOW TO VERIFY: <steps>`
   - When non-blocking, document assumptions inline as comments and add `TODO(verify)` with exact verification steps.

2. **Production Readiness First**
   - Every deliverable must compile, handle errors, and degrade gracefully under partial failures (Wi-Fi down, API slow, I2C glitch).
   - Include timeouts, retries with backoff + jitter, and guard across all I/O.
   - Prefer **deterministic** behavior over cleverness.

3. **Hardware Awareness**
   - Optimize for ESP32-WROOM-32 class devices (typical: 240 MHz, ~320 KB free heap at runtime, no PSRAM by default).
   - Treat RAM as scarce; treat flash and CPU as constrained.
   - Respect bus limits (I²C 100–400 kHz, SPI up to tens of MHz but limited by wiring/display).

4. **Safety & Robustness**
   - No dynamic allocation in hot paths or repeatedly within `loop()`. Pre-allocate statically or use arenas.
   - No blocking `delay()` in production paths; use `millis()`-based scheduling.
   - ISRs must be tiny: no heap, no logging, set flags only.

---

## 3) Target Profile & Budgets

- **Board:** ESP32-WROOM-32 (no PSRAM assumed).
- **Display:** SSD1322 OLED (256×64) via SPI (NHD-5.5-25664UCG3). Driver: U8g2 full framebuffer.
  - **SPI pins:** CS=5, DC=16, RST=17, SCLK=18, MOSI=23.
  - **Frame budget:** aim ≤ 15 ms/update (≈66 FPS max; typical ≤ 20 FPS).
  - **Redraw strategy:** partial/incremental when possible; avoid full-screen clears each frame.
- **Networking:** Wi-Fi STA, TLS optional; memory pressure increases with HTTPS.
- **Timing goals:** No single task in `loop()` > 5 ms. Aggregate work slice < 10 ms typical.

---

## 4) Required Deliverable Format

Every code response must include:

1. **Header block**: target board, libraries, FQBN, memory notes, and tested toolchain.
2. **Explicit assumptions** and how to validate each.
3. **Build instructions** (Arduino IDE + `arduino-cli`).
4. **Config surface**: constants with sane defaults, overridable via header or `config.h`.
5. **Error handling**: timeouts, retries, and user-visible diagnostics.
6. **Resource accounting**: expected flash/RAM use (“~xx KB static; ~yy KB heap at idle”).
7. **Test/Verification steps**: serial assertions, visual checks, and a smoke test.
8. **Roll-back plan**: how to disable a feature via compile flag if issues occur.

---

## 5) Coding Standards (concise)

- **C++**: `constexpr`, `enum class`, `span`/bounds checks; avoid Arduino `String` in hot paths.
- **Formatting**: 2-space indent, K&R braces, ≤100 cols. Provide `clang-format` compliance.
- **Memory**: Use `PROGMEM`, `F()` for literals, `static` buffers, `snprintf` with bounds.
- **Concurrency**: Prefer message-passing flags; protect shared state with minimal critical sections.
- **Timing**: Cooperative scheduling via `millis()`. No busy-waits. No `delay()` in logic.

---

## 6) Reliability Patterns (must use)

- **Watchdog-friendly**: ensure `loop()` stays responsive; feed WDT when long tasks are chunked.
- **I²C/SPI Resilience**: bus scan on boot (optional), re-init on error, bounded retries.
- **Wi-Fi**: state machine with **exponential backoff + jitter**, DNS and TLS timeouts, offline cache path.
- **Logging**: leveled macros (`LOG_ERROR`, `LOG_WARN`, `LOG_INFO`, `LOG_DEBUG`) with compile-time enable.
- **Metrics**: optional lightweight counters for frame time, heap watermark, retries; printed on demand.

---

## 7) Performance & Footprint Guidance

- **Rendering**
  - Precompute glyph positions; avoid per-frame string concatenation.
  - Use partial updates; dirty-rect redraw vs full-buffer flush when library permits.
  - Avoid float in hot loops; pre-convert to fixed-point or scaled integers.
- **Parsing**
  - Prefer streaming/iterative parsing; cap JSON sizes; validate fields and ranges strictly.
- **Networking**
  - Limit concurrency; reuse connections when safe; cap payload sizes; compress or trim JSON.
- **Storage**
  - Use small, fixed-size caches with eviction; persist only essential state.

---

## 8) Production Checklists

### 8.1 Pre-Implementation
- [ ] Confirm target board/FQBN and display controller + bus pins.
- [ ] Confirm supply voltage & current headroom for display and peripherals.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [filbot/flight-display](https://github.com/filbot/flight-display) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
