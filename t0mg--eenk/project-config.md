---
trigger: always_on
description: This file provides context and instructions for AI coding agents working on the eenk ecosystem.
---

# AGENTS.md — eenk Project Guide

This file provides context and instructions for AI coding agents working on the eenk ecosystem.

---

## Ecosystem Overview

The eenk project is an interactive fiction runtime for the **Xteink X4 and X4Pro** e-ink devices. It consists of three repositories:

| Repository | Description |
|-----------|-------------|
| **[eenk](https://github.com/t0mg/eenk)** | Core C++ firmware for the ESP32-C3 hardware and SDL desktop simulator. This is the **primary** repository. |
| **[eenky](https://github.com/t0mg/eenky)** | Electron + Vue 3 IDE for authoring Ink stories, running the SDL simulator, and flashing firmware to the device via USB. Fork of inkle's Inky. |
| **[inkcpp](https://github.com/t0mg/inkcpp)** | Fork of the inkcpp C++ Ink runtime with eenk-specific patches (branch: `eenk-patches`). Used as a submodule by both eenk and eenky. |

---

## Repository Architecture & Submodule Rules

The repositories are cross-linked via git submodules:

```
eenk (primary repo)
├── lib/freeink-sdk  → Free-Ink/freeink-sdk (main branch)
└── lib/inkcpp       → t0mg/inkcpp (eenk-patches branch)
```

eenky is a **separate repository** (not a submodule of eenk). It has its own submodule structure:

```
eenky
├── eenk/    → t0mg/eenk (submodule pointing at the primary repo)
└── inkcpp/  → t0mg/inkcpp (eenk-patches branch)
```

> [!CAUTION]
> **Never modify `lib/freeink-sdk` submodule files.**
> `freeink-sdk` is an upstream hardware SDK. Do NOT make changes inside `lib/freeink-sdk/`. Any host/native build shims, wrappers, or mock headers must be placed in `src/hal/sdl/mock/` or within `eenk`'s own source tree, never directly in `freeink-sdk`.

> [!IMPORTANT]
> **Submodule Pointer & CI Release Synchronization**:
> In Git, `eenky` pins a specific commit SHA of the `eenk` submodule. GitHub Actions CI (`build-executables.yaml`) checks out this exact pinned commit.
> Whenever you make changes in `eenk` (such as firmware, simulator binaries, `WritingForEenk.md`, or documentation):
> 1. Commit and push changes in the primary `eenk` repository.
> 2. In `eenky`, run the one-step updater from `app/`:
>    ```sh
>    cd app
>    npm run update-submodules
>    ```
>    *(This runs `git submodule update --remote`, rebuilds `eenk-sim.exe` & `inkcpp_cl.exe`, synchronizes `eenk_version.txt`, and rebuilds `CombinedDocumentation.md` & `embedded.html`).*
> 3. Verify submodule status via `git submodule status` (a `+` prefix indicates your local submodule commit differs from the committed pointer).
> 4. Stage and commit the updated submodule pointer and generated files in `eenky`:
>    ```sh
>    git add eenk app/main-process/ink/eenk_version.txt app/resources/Documentation/
>    git commit -m "Update eenk submodule, binaries, and documentation"
>    git push origin master
>    ```
> Failing to commit the updated submodule pointer in `eenky` will cause GitHub Actions packaging workflows to package old documentation and old simulator binaries.

---

## eenk Repository (Firmware & Simulator)

### Build System

eenk uses **PlatformIO** with four build environments defined in `platformio.ini`:

| Environment | Target | Purpose |
|------------|--------|---------|
| `native` | Host (MinGW/SDL2) | Desktop simulator, renders to SDL2 window |
| `esp32c3` | ESP32-C3 | Main firmware for Xteink X3 and X4 hardware (app0) |
| `esp32c3_updater` | ESP32-C3 | Minimal OTA updater firmware for X4 (app1) |
| `esp32s3` | ESP32-S3 | Main firmware for Xteink X4 Pro hardware (app0) |

**Build commands:**
```sh
pio run -e native          # Desktop simulator
pio run -e esp32c3         # Main firmware (X4)
pio run -e esp32s3         # Main firmware (X4 Pro)
pio run -e esp32c3_updater # OTA updater (X4)
pio test -e native         # Unit tests (native only)
```

> [!NOTE]
> The X4 Pro has **no separate updater environment** — it uses symmetric A/B OTA partitions (both `app0` and `app1` are full ~7.9 MB firmware slots). OTA on the X4 Pro is handled by swapping the active slot via `esp_ota_set_boot_partition` directly from the main firmware.

> [!IMPORTANT]
> **After making changes, always build `native`, `esp32c3`, and `esp32s3` targets** to verify cross-platform compatibility. The native build uses host g++ with SDL2; the ESP32 builds use the ESP-IDF Arduino framework.

### Partition Layout

The ESP32-C3 (X4) and ESP32-S3 (X4 Pro) both use 16 MB flash chips, but their partition schemes differ.

**X4 (16 MB) Layout (`partitions.csv`):**
| Partition | Type | Offset | Size | Purpose |
|-----------|------|--------|------|---------|
| `nvs` | data | 0x9000 | 20 KB | Non-volatile storage (settings, boot state) |
| `otadata` | data | 0xE000 | 8 KB | OTA boot selection metadata |
| **`app0`** | app (ota_0) | 0x10000 | **7 MB** | Main firmware — full interactive fiction runtime |
| **`app1`** | app (ota_1) | 0x710000 | **1 MB** | OTA Updater — minimal recovery/update firmware |
| `ink_cache` | data (FAT) | 0x810000 | ~7.9 MB | Story file cache (FAT filesystem) |
| `coredump` | data | 0xFF0000 | 64 KB | Crash dump storage |

**X4 Pro (16 MB) Layout (`partitions_x4pro.csv`):**
| Partition | Type | Offset | Size | Purpose |
|-----------|------|--------|------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [t0mg/eenk](https://github.com/t0mg/eenk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
