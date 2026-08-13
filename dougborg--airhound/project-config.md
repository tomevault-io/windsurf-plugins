---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AirHound is a surveillance detection toolkit built around three portable layers: a standardized **signature database** (JSON Schema in `schemas/signatures.v1.schema.json`), a documented **companion event protocol** (NDJSON with JSON Schemas in `schemas/`), and a **detection library** (`src/lib.rs` — `no_std` Rust, no platform dependencies). ESP32 firmware is the reference platform consumer; a Linux daemon ([#13](https://github.com/dougborg/AirHound/issues/13)) and Kismet companion ([#12](https://github.com/dougborg/AirHound/issues/12)) are planned. Companion apps handle analysis, scoring, GPS tagging, and storage. See [#17](https://github.com/dougborg/AirHound/issues/17) for the full architecture vision.

## Terminology

The project uses these terms consistently across code, schemas, and documentation:

**Data concepts:**
- **Signature** — atomic, stateless matching criterion. Currently 6 types: `mac_oui`, `wifi_ssid`, `ble_name`, `ble_service_uuid`, `ble_manufacturer_id`, `ble_ad_bytes`. Defined in `signatures.v1.schema.json`, compiled defaults in `defaults.rs`.
- **Rule** — named device detection composing signatures with `anyOf`/`allOf`/`not` boolean logic. Defined in `signatures.v1.schema.json`. Rule evaluation engine not yet implemented.
- **Signature database** — portable JSON collection of signatures and rules.

**Processing concepts:**
- **Scan event** — parsed WiFi frame or BLE advertisement. Types: `ScanEvent`, `WiFiEvent`, `BleEvent` in `scanner.rs`.
- **Filter engine** — stateless code evaluating scan events against signatures. Entry points: `filter_wifi()`, `filter_ble()` in `filter.rs`.
- **Filter config** — runtime parameters (RSSI threshold, WiFi/BLE enable). `FilterConfig` in `filter.rs`. Not signature data.
- **Match** — positive result from the filter engine. `FilterResult` in `filter.rs`.
- **Match reason** — which signature type triggered and a human-readable detail. `MatchReason` in `protocol.rs`. The `filter_type` field name is a legacy misnomer (rename to `signature_type` tracked for v2, [#9](https://github.com/dougborg/AirHound/issues/9)).

**Output concepts:**
- **Device message** — NDJSON line from sensor. `DeviceMessage` in `protocol.rs`, schema: `device-message.v1.schema.json`.
- **Host command** — NDJSON line to sensor. `HostCommand` in `protocol.rs`, schema: `host-command.v1.schema.json`.
- **Companion event protocol** — the transport-agnostic NDJSON wire format for device messages and host commands.

**WIDS concepts (planned, [#32](https://github.com/dougborg/AirHound/issues/32)):**
- **Fingerprint alert** — single-frame security anomaly (e.g., malformed IE, zero WPA NONCE). Stateless.
- **Behavioral alert** — multi-frame temporal detection (e.g., deauth flood, evil twin). Requires per-device state. Implemented as code in `wids.rs`, not declarative rules.

## Build Commands

The project builds inside Docker using chip-specific Espressif images (no local ESP toolchain needed). Install `just` (`cargo install just`).

```bash
just docker-build            # Both targets
just docker-build-xiao       # XIAO ESP32-S3 only
just docker-build-m5stickc   # M5StickC Plus2 only
just docker-check            # Type-check both targets
just docker-clean            # Clean artifacts (REQUIRED after dependency changes)
```

To flash (requires `espflash` on host, device connected via USB):
```bash
espflash flash --chip esp32s3 target/xtensa-esp32s3-none-elf/release/airhound
espflash flash --chip esp32 target/xtensa-esp32-none-elf/release/airhound
```

Tests and formatting:
```bash
just docker-test             # Run unit tests (in container)
just test                    # Run unit tests (requires nightly on host)
cargo fmt --check            # Check formatting (requires nightly on host)
just setup-hooks             # Configure git pre-commit + commit-msg hooks
```

The project uses `build-std = ["core", "alloc"]` (passed via justfile, not `.cargo/config.toml`). Commits must follow [Conventional Commits](https://www.conventionalcommits.org/) format.

## Feature Flags

Board features (`xiao`, `m5stickc`) select chip features (`esp32s3`, `esp32`), which enable chip-specific crate features across all `esp-*` dependencies. **Only one board feature should be active at a time.** `xiao` is the default.

The `m5stickc` feature additionally enables display (`mipidsi`, `embedded-graphics`, `embedded-hal-bus`) and buzzer modules.

## Architecture

The project's three portable layers (signature schemas, companion event protocol, detection library) are described in [Architectural Direction](#architectural-direction) below. This section covers the **ESP32 firmware** implementation.

The firmware runs on the Embassy async executor (`esp-rtos`). All tasks are single-threaded cooperative (no preemption). Tasks communicate through static `embassy_sync::Channel`s defined in `main.rs`:

- **SCAN_CHANNEL** (capacity 8 on ESP32, 16 on ESP32-S3) — WiFi sniffer ISR and BLE scan task push raw `ScanEvent`s
- **OUTPUT_CHANNEL** (capacity 8) — Serialized NDJSON `MsgBuffer`s ready for transmission

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dougborg/AirHound](https://github.com/dougborg/AirHound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
