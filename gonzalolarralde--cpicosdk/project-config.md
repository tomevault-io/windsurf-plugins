---
trigger: always_on
description: This repo has a hardware-oriented example workflow. For this project, build and
---

# Agent Device Workflow

This repo has a hardware-oriented example workflow. For this project, build and
device commands should be run from `Example/` unless noted otherwise.

When a debugging session produces a reusable general workflow lesson, add it to
this file under `Debugging Tips Log`. Classify each tip by area, such as
code/Swift, USB connection, OpenOCD/GDB, serial/RTT logging, build/package
wiring, or another specific category that fits the issue. Keep the tip concrete:
include the symptom, the command or code pattern that helped, and any known
limitation. Keep feature-specific investigation notes in `docs/`.

## Root Device Test Harness

The repo-root device test harness is the main exception to the `Example/`
working-directory rule. Its contributor documentation lives in
`README.md` under `Device Test Harness`, and test sources live under
`Tests/Device/**/*.swift`.

List available device tests from the repo root:

```sh
swift package --disable-sandbox test-in-device --list --allow-writing-to-package-directory --allow-network-connections all
```

Check that device tests generate, compile, and link without programming
hardware:

```sh
swift package --disable-sandbox test-in-device --build-only --allow-writing-to-package-directory --allow-network-connections all
```

Run the physical-device suite from the repo root only after confirming with the
user that a compatible board and CMSIS-DAP/OpenOCD probe are connected and that
it is OK to program the board:

```sh
swift package --disable-sandbox test-in-device --allow-writing-to-package-directory --allow-network-connections all
```

For focused checks, prefer `--filter <TestName>` before running all tests. Run
`--build-only` often when adding tests or changing device-facing CPicoSDK
behavior, traits, concurrency support, generated package wiring, finalization,
or linking. Run the physical tests occasionally when changing OpenOCD/RTT
capture or the harness itself. For documentation-only changes, do not program
the device unless the user asks for it.

## Build

Do not run a repo-root `./build`. Build the example like this:

```sh
cd Example
./build.sh
```

The build artifact used for flashing is:

```text
Example/.build/armv7em-none-none-eabi/release/Example.elf
```

The corresponding UF2 is in the same directory:

```text
Example/.build/armv7em-none-none-eabi/release/Example.uf2
```

## Finding Tool Binaries

Run `./build.sh` from `Example/` first. The build prepares the Pico SDK bundle
and cross toolchain under `Example/.build/plugins/PrepareEnvironmentPlugin`.

Find OpenOCD:

```sh
cd Example
find .build/plugins/PrepareEnvironmentPlugin/outputs \( -type f -o -type l \) \
  \( -name openocd.exe -o -name openocd \)
```

Find the OpenOCD scripts directory:

```sh
cd Example
find .build/plugins/PrepareEnvironmentPlugin/outputs -type d -path '*/openocd/*/scripts'
```

Find GDB and related ARM toolchain utilities:

```sh
cd Example
find .build/plugins/PrepareEnvironmentPlugin/outputs -type f \
  \( -name arm-none-eabi-gdb -o -name arm-none-eabi-addr2line -o -name arm-none-eabi-nm \)
```

Set shell variables from the discovered paths before running debug commands:

```sh
cd Example
ELF=".build/armv7em-none-none-eabi/release/Example.elf"
OPENOCD="$(find .build/plugins/PrepareEnvironmentPlugin/outputs \( -type f -o -type l \) \( -name openocd.exe -o -name openocd \) -print -quit)"
OPENOCD_SCRIPTS="$(find .build/plugins/PrepareEnvironmentPlugin/outputs -type d -path '*/openocd/*/scripts' -print -quit)"
ARM_GDB="$(find .build/plugins/PrepareEnvironmentPlugin/outputs -type f -name arm-none-eabi-gdb -print -quit)"
ARM_ADDR2LINE="$(find .build/plugins/PrepareEnvironmentPlugin/outputs -type f -name arm-none-eabi-addr2line -print -quit)"
ARM_NM="$(find .build/plugins/PrepareEnvironmentPlugin/outputs -type f -name arm-none-eabi-nm -print -quit)"
OPENOCD_HELPERS="$(find "$HOME/.vscode/extensions" -path '*/support/openocd-helpers.tcl' -print -quit 2>/dev/null)"
```

The Cortex-Debug OpenOCD helper script is optional for command-line debugging.
If `OPENOCD_HELPERS` is empty, remove the `-f "$OPENOCD_HELPERS"` argument from
the OpenOCD examples below.

## OpenOCD

The OpenOCD binary and script bundle come from the example build output. Use
the variables from `Finding Tool Binaries`.

```sh
"$OPENOCD" \
  -c "gdb_port 50000" \
  -c "tcl_port 50001" \
  -c "telnet_port 50002" \
  -s "$OPENOCD_SCRIPTS" \
  -f "$OPENOCD_HELPERS" \
  -f interface/cmsis-dap.cfg \
  -f target/rp2350.cfg \
  -c "adapter speed 5000"
```

Leave that command running when you want persistent GDB or telnet access.

If the target has recently hardfaulted or OpenOCD reports
`Error connecting DP: cannot read IDR`, retry with `adapter speed 1000` before
assuming the wiring or probe is wrong. Some wedged states still require a
physical reconnect or power-cycle of the board before OpenOCD can regain DP
access.

If stale OpenOCD processes are holding ports, stop them with:

```sh
pkill -f /openocd.exe
```

## Flash

After building from `Example/`, flash the ELF with:

```sh
"$OPENOCD" \
  -c "gdb_port 50000" \
  -c "tcl_port 50001" \
  -c "telnet_port 50002" \
  -s "$OPENOCD_SCRIPTS" \
  -f "$OPENOCD_HELPERS" \
  -f interface/cmsis-dap.cfg \
  -f target/rp2350.cfg \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gonzalolarralde/CPicoSDK](https://github.com/gonzalolarralde/CPicoSDK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
