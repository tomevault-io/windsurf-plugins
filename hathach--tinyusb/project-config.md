---
trigger: always_on
description: TinyUSB is a cross-platform USB Host/Device stack for embedded systems: memory-safe (no dynamic allocation) and thread-safe (ISR events deferred to task context).
---

# TinyUSB Agent Instructions

TinyUSB is a cross-platform USB Host/Device stack for embedded systems: memory-safe (no dynamic allocation) and thread-safe (ISR events deferred to task context).

Reference these instructions first; fall back to search/bash only when reality diverges.

## Behavioral Guidelines

Bias toward caution over speed. For trivial tasks, use judgment.

- **Think first** — state assumptions; ask if unclear; present alternatives instead of picking silently.
- **Simplicity** — no features, abstractions, flexibility, or error handling beyond what was asked. If 200 lines could be 50, rewrite.
- **Surgical changes** — touch only what the task requires; match existing style; don't refactor working code; mention unrelated dead code rather than deleting it. Remove only orphans *your* changes created.
- **Goal-driven** — turn tasks into verifiable goals ("write failing test, make it pass"). For multi-step work, state a brief `step → verify` plan.
- **Worktrees** — default to a git worktree for any branch or multi-step work; never switch the shared primary checkout's branch. Sessions run concurrently: switching the primary checkout mid-flight disrupts other sessions and can silently point a review, build, or commit at the wrong diff. Only trivial one-shot fixes may skip this. Standard location: `.worktrees/<branch-name>` at the repo root (gitignored), e.g. `git worktree add .worktrees/my-branch -b my-branch`.

## Ground Rules

- **Language/style:** C99, 2-space indent (no tabs), snake_case helpers, `UPPER_CASE` macros. Public APIs use `tud_`/`tuh_`; macros use `TU_`. Headers self-contained with `#if CFG_TUSB_MCU` guards.
- **Safety:** no dynamic allocation; defer ISR work to task context; use `TU_ASSERT()` for error checks; always check return values; include order: C stdlib → tusb common → drivers → classes.
- **Layout:** `src/` core, `hw/{mcu,bsp}/` MCU+BSP, `examples/{device,host,dual}/`, `test/{unit-test,fuzz,hil}/`, `docs/`, `tools/`.
- **Commits/PRs:** imperative mood, scoped changes, link issues, include test/build evidence. After opening a PR, drive it to green: address automated review comments (Copilot/Codex/Claude) and fix failing CI, pushing follow-ups until checks pass and threads resolve. Useful: `gh pr checks <num> --watch`, `gh pr view <num> --comments`.
- **Formatting/lint:** `clang-format` (`.clang-format`), `codespell` (`.codespellrc`); run `pre-commit run --all-files` before submitting.

## Bootstrap

```bash
sudo apt-get install -y gcc-arm-none-eabi          # ARM toolchain (2-5 min, one-time)
python3 tools/get_deps.py [FAMILY|-b BOARD]        # fetch deps into lib/, hw/mcu/ (<1 s)
. $HOME/code/esp-idf/export.sh                     # Espressif only: before any build/flash/monitor
```

## Build

Single example (CMake+Ninja, recommended, 1-3 s):
```bash
cd examples/device/cdc_msc && mkdir -p build && cd build
cmake -DBOARD=raspberry_pi_pico -G Ninja -DCMAKE_BUILD_TYPE=MinSizeRel .. && cmake --build .
```

All examples for a board (15-20 s; some objcopy failures are non-critical). The build dir **must** be `cmake-build-<board>` — HIL tests expect that exact name:
```bash
cd examples
cmake -B cmake-build-raspberry_pi_pico -DBOARD=raspberry_pi_pico -G Ninja -DCMAKE_BUILD_TYPE=MinSizeRel . && cmake --build cmake-build-raspberry_pi_pico
```

- **Make:** `cd examples/device/cdc_msc && make BOARD=raspberry_pi_pico all`
- **Espressif** (ESP-IDF examples only, e.g. `cdc_msc_freertos`): after `export.sh`, `cd examples/device/cdc_msc_freertos && idf.py -DBOARD=espressif_s3_devkitc build`
- **Options** (CMake `-D…` / Make `…=…`): `CMAKE_BUILD_TYPE=Debug`/`DEBUG=1`; `LOG=2` (`LOGGER=rtt` for RTT); `RHPORT_DEVICE=1`; `RHPORT_DEVICE_SPEED=OPT_MODE_FULL_SPEED`

## Flash

```bash
ninja cdc_msc-jlink       # CMake; Make: make BOARD=<board> flash-jlink
ninja cdc_msc-openocd     # CMake; Make: make BOARD=<board> flash-openocd
ninja cdc_msc-uf2         # CMake; Make: make BOARD=<board> all uf2
ninja -t targets          # list CMake targets
```
Espressif (after `export.sh`): `idf.py -DBOARD=<board> flash` / `… monitor`.

## GDB Debugging

Look up `JLINK_DEVICE` / `OPENOCD_OPTION` in `hw/bsp/*/boards/*/board.cmake` (CMake) or `board.mk` (Make).

Terminal 1 — start a gdbserver:
```bash
JLinkGDBServer -device stm32h743xi -if SWD -speed 4000 -port 2331 -nogui   # JLink → :2331
openocd -f interface/stlink.cfg -f target/stm32h7x.cfg                     # OpenOCD → :3333
openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "adapter speed 5000"   # rp2040/rp2350
```
Terminal 2 — connect (`<port>`: 2331 JLink, 3333 OpenOCD):
```bash
arm-none-eabi-gdb build/your_app.elf
(gdb) target remote :<port>    # then: monitor reset halt → load → continue
```
**RTT:** build `LOG=2 LOGGER=rtt`, run JLinkGDBServer with `-RTTTelnetPort 19021`, then `JLinkRTTClient` (`timeout 20s JLinkRTTClient > rtt.log` for non-interactive capture).

## Testing

**Unit (Ceedling, Unity+CMock, ~4 s):**
```bash
sudo gem install ceedling
cd test/unit-test && ceedling test:all        # or ceedling test:test_fifo
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hathach/tinyusb](https://github.com/hathach/tinyusb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
