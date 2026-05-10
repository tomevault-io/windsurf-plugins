---
trigger: always_on
description: Drop-in context file for AI coding agents (Claude Code, Cursor, Copilot, Codex, …) working on this repository. Distilled from the project docs and many hours of human-AI co-development. Read this once before your first edit, refer back when a task touches an unfamiliar area.
---

# AGENTS.md — instructions for AI coding agents

Drop-in context file for AI coding agents (Claude Code, Cursor, Copilot, Codex, …) working on this repository. Distilled from the project docs and many hours of human-AI co-development. Read this once before your first edit, refer back when a task touches an unfamiliar area.

If something here contradicts the user's explicit instruction in the conversation, follow the user.

---

## What this project is

**Pixel Pets** — a family of virtual pets running on M5Stack ESP32 hardware. One source tree, four firmware binaries:

| Env | Hardware | Pet name | Highlights |
|---|---|---|---|
| `cores3` | M5Stack CoreS3 + Module-LLM (AX630C) | Muffin   | Wake-word voice control + offline LLM, front camera, touchscreen, web radio |
| `visu`   | M5Stack CoreS3 (no LLM module)        | Visu     | Same hardware as Muffin, voice removed, camera + touch + web radio still in |
| `core2`  | M5Stack Core2                         | Goo-Goo  | BtnA/B/C + touch, web radio, no voice, no camera |
| `pip` / `pip-s3` | M5StickC PLUS2 (PICO or S3)   | Pip      | Pocket-sized, 135×240, BtnA/B, buzzer audio, no WiFi |

The umbrella project name is **Pixel Pets**. The repo was originally named `muffin` (the flagship pet) and may still be called that locally.

For the gameplay model see [`docs/concept.md`](docs/concept.md), for the software architecture [`docs/architecture.md`](docs/architecture.md), for hardware setup [`docs/hardware.md`](docs/hardware.md).

---

## Quick commands

### Build

```bash
pio run -e cores3                          # default — Muffin (CoreS3 + LLM)
pio run -e visu                            # Visu — CoreS3 without LLM
pio run -e core2                           # Goo-Goo — Core2
pio run -e pip                             # Pip — StickC PLUS2 (ESP32 PICO)
pio run -e pip-s3                          # Pip — StickC PLUS2 (S3 revision)
pio run -e cores3 -e visu -e core2 -e pip-s3   # build all four
```

After a non-trivial change, **build all four** (or at least `core2`, `cores3`, `visu`, `pip-s3`). Single-target builds will not catch capability-flag regressions in code only one target compiles.

For maximum speed, fire each `pio run -e <env>` as an independent background task. They don't conflict.

### Flash

```bash
pio run -e <env> -t upload --upload-port COM<N>
```

`pio device list` shows attached COM ports. CoreS3 normally enumerates as `303A:1001`, Core2 as a CH9102 bridge, StickC PLUS2 (PICO) as a CH9102 too, StickC PLUS2 S3 alternates between `303A:8120` (normal) and `303A:1001` (download mode — hold BtnA + reset). See [`docs/hardware.md`](docs/hardware.md) for the full StickC-S3 download-mode dance.

Two devices on different ports can be flashed in parallel — `pio run -e core2 -t upload --upload-port COM3` and `... --upload-port COM7` simultaneously work fine.

### Monitor

```bash
pio device monitor --port COM<N> --baud 115200
```

Two devices in parallel: open two terminals (or run two `pio device monitor` with different `--port`). Friends-mode debugging needs both pets visible at once.

### Test

```bash
pio test -e native
```

Runs Unity tests against the pure-logic modules ([`src/needs_logic.cpp`](src/needs_logic.cpp)) on the host compiler. CI runs this on every push.

### CI

[`.github/workflows/ci.yml`](.github/workflows/ci.yml) builds the matrix (cores3 / core2 / visu / pip) and runs `pio test -e native`. Any push that breaks one env fails the run. Don't merge through a red CI.

---

## Capability-flag system — required reading

[`src/target_caps.h`](src/target_caps.h) is the single source of truth for what each build can do. Every other module gates code via `#if TARGET_HAS_…`:

```c
TARGET_HAS_LLM            // voice pipeline (cores3 only)
TARGET_HAS_CAMERA         // front camera + face detection + photos (cores3 + visu)
TARGET_HAS_HARD_BUTTONS   // physical BtnA/B/C (core2, pip)
TARGET_HAS_TOUCH          // touchscreen (cores3, visu, core2)
TARGET_HAS_WAV_AUDIO      // WAV speaker (everywhere except pip)
TARGET_HAS_WIFI           // WiFi features (NTP, weather, web radio, ESP-NOW friends)
TARGET_DISPLAY_W / _H     // display dimensions
TARGET_NAME / TARGET_AP_NAME / TARGET_MDNS_NAME   // branding
```

Every env in [`platformio.ini`](platformio.ini) sets exactly one `TARGET_<board>=1` flag (e.g. `-DTARGET_CORES3=1`); the header derives every capability from there. **Adding a new target is one new branch in `target_caps.h` plus one new `[env:…]` section.**

`build_src_filter` in `platformio.ini` excludes hardware-specific `.cpp` files where they don't apply — and call sites in `main.cpp` are also wrapped in `#if TARGET_HAS_…` so the linker doesn't see references to symbols that aren't compiled in. **Both** are required; one without the other is a build error.

When you add a new feature, ask: which capability flag gates it? If the answer is "all targets", no flag needed. If a flag exists, gate on the flag, not on `TARGET_<board>`.

---

## Source-code conventions

### Comments are English-only


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marceld23/Pixel-Pets](https://github.com/marceld23/Pixel-Pets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
