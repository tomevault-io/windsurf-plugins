---
trigger: always_on
description: Guidance for autonomous coding agents working in `crosspoint-reader-cjk`.
---

# AGENTS.md

Guidance for autonomous coding agents working in `crosspoint-reader-cjk`.
This repository is a CJK-focused fork of CrossPoint Reader. A successful upstream merge must preserve the fork's user-visible behavior, not merely compile.

## 1. Project and hardware

- Firmware: PlatformIO + Arduino, primarily C++20 (`-std=gnu++2a`) with Python build/test utilities.
- Main target: Xteink X4/X3, ESP32-C3, 16 MB flash, no PSRAM.
- Additional CI target: Seeed Studio XIAO ePaper Display Board / "Sticky", ESP32-S3.
- Main entry point: `src/main.cpp`; recovery entry point: `src/recovery/RecoveryMain.cpp`.
- The C3 is memory-constrained. The display buffer is required baseline memory, not a leak. Avoid file-sized buffers, repeated hot-path allocation, and unbounded containers.
- Exceptions are disabled. Allocation and I/O failures must be handled explicitly.

## 2. Clone and submodules

Clone recursively:

```bash
git clone --recursive <repo-url>
cd crosspoint-reader-cjk
git submodule update --init --recursive
```

Required submodules:

- `freeink-sdk/`: mandatory for all firmware builds. `platformio.ini` consumes SDK libraries through `symlink://freeink-sdk/...`, including display, input, storage, power, board configuration, UI, icons, and secure networking.
- `freeink-sdk/libs/assets/Icons/lucide`: nested source asset submodule used when regenerating Lucide-derived SDK icons. It is not needed to compile already-generated icon headers, but recursive initialization keeps the SDK checkout complete and matches CI.
`.gitmodules` names this fork's SDK update branch; the repository gitlink pins the reproducible SDK commit. Do not replace that gitlink with upstream SDK HEAD or edit the SDK incidentally. If an SDK change is required, verify the API in the checked-out submodule, commit it in the SDK repository first, and then update this repository's gitlink in a separate, intentional change.

Quick verification:

```bash
git submodule status --recursive
```

A leading `-` means the submodule is not initialized. A leading `+` means the checkout differs from the recorded gitlink.

## 3. Tooling and first build

Required tools:

- Python 3 (CI uses 3.14)
- PlatformIO Core (`pio`)
- clang-format 21+
- Pillow for built-in CJK font generation

The first build is slower because `custom_sdkconfig` rebuilds Arduino/ESP-IDF components. On macOS, retain any machine-specific CMake/toolchain workaround in gitignored `platformio.local.ini`; never commit that file.

Enable the repository's local Git hooks once per checkout:

```bash
./bin/install-git-hooks
```

The pre-commit hook formats tracked C/C++ changes, and the pre-push hook runs the
same cppcheck command as CI. A push is blocked when cppcheck reports any low, medium,
or high defect. For an exceptional one-off push when PlatformIO cannot run, explicitly
use `CROSSPOINT_SKIP_PRE_PUSH_CHECKS=1 git push`; CI still remains authoritative.

If an interrupted core rebuild reports multiple definitions of `app_main`, follow the cleanup command documented in `platformio.ini`. Do **not** use `git clean -fdX`, because it can delete local configuration and other intentionally ignored assets.

## 4. PlatformIO environments

Use the environment that matches the artifact or test:

| Environment | Purpose | Languages / notes |
|---|---|---|
| `default` | Normal ESP32-C3 development build | EN, Simplified Chinese, Traditional Chinese, Japanese; debug serial logging |
| `gh_release` | Simplified Chinese release | EN, SC, JA |
| `gh_release_rc` | Simplified Chinese release candidate | Inherits the SC release language set |
| `gh_release_tc` | Traditional Chinese release | EN, TC, JA |
| `gh_release_rc_tc` | Traditional Chinese release candidate | Inherits the TC release language set |
| `slim` | Size-focused C3 build | All four shipping languages; serial logging disabled |
| `device_test` | Physical-device automation | Test-only serial input injection; never publish this artifact |
| `recovery` | Minimal SD recovery firmware | English-only, separate source filter |
| `sticky` | ESP32-S3 Sticky development/CI build | Different MCU/toolchain; not interchangeable with X4 firmware |

Common commands:

```bash
pio run -e default
pio run -e sticky
pio run -e gh_release
pio run -e gh_release_tc
pio run -e recovery
pio run -e default --target upload
pio device monitor
```

The registered upload target is partition-aware for Xteink C3 environments. It writes application partitions and updates OTA selection while preserving the bootloader, partition table, and data partitions such as NVS, SPIFFS, and coredump; it does not preserve old application images in factory/OTA slots. Do not bypass `scripts/register_safe_upload.py` / `scripts/upload_ota_slots.py` with an arbitrary whole-flash command unless the task explicitly requires and validates the complete partition layout.

After physical tests with `device_test` or screenshot-only configurations, remove temporary overrides and restore a `default` development firmware to the device.

## 5. Build-time scripts and generated files

`platformio.ini` wires these scripts into every applicable build:

- `scripts/patch_wolfssl.py`: applies/checks the constrained-heap wolfSSL integration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CrossPoint-CJK/crosspoint-reader-cjk](https://github.com/CrossPoint-CJK/crosspoint-reader-cjk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
