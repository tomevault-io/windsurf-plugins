---
trigger: always_on
description: This file explains how to work on this MSPM0G3507 firmware project safely.
---

# AGENTS.md

This file explains how to work on this MSPM0G3507 firmware project safely.
The project uses CMake + GCC + OpenOCD, not a CCS project layout, but the
SysConfig and DriverLib rules still matter.

## Project Shape

- `core/` contains the MSPM0 SDK surface used by this project: startup code,
  linker script, SysConfig output, DriverLib, CMSIS, FreeRTOS, and syscall glue.
- `core/trobot.syscfg` is the source of truth for clocks, pins, peripherals,
  DMA channels, interrupts, and generated initialization names.
- `bsp/` contains board support code for UART, SPI, LCD, flash, time, GPIO, and
  low-level helpers.
- `components/` contains optional reusable modules. `components/utils` is a git
  submodule and is required by the current application. It supplies the C++
  logger, task/queue wrappers, terminal, message, CRC, and VOFA helpers.
- `app/` contains the firmware entry point and application tasks.
- `bsp/include/bsp/` is the public BSP include surface. `bsp/internal/` is
  private to the BSP target and must not be included from `app/` or components.
- `*.cfg` files at the repository root select the OpenOCD probe interface.

## Runtime Architecture

The current boot and initialization sequence is:

1. The GCC startup code initializes `.data`, `.bss`, C/C++ constructors, and
   then calls `main()`.
2. `main()` calls `SYSCFG_DL_init()`, creates the `app_entrance` task, and
   starts the FreeRTOS scheduler.
3. `app_entrance()` calls `bsp_hw_init()`, initializes UART0 and its TX DMA
   queue, enables the board GPIO interrupt, initializes the logger, and creates
   application tasks.
4. `bsp_hw_init()` initializes the ST7735 LCD and verifies the W25Q128 device
   ID. A failed BSP assertion records its expression/message/file/line, breaks
   only when a debugger is attached, and then stops forever.

Keep this order in mind when adding code. A task that logs asynchronously needs
its UART initialized first. LCD and flash users need the board/SPI GPIO state
initialized first. Do not move scheduler-dependent initialization into global
C++ constructors.

The current interrupt ownership is split across layers:

- `GROUP1_IRQHandler` is owned by `app/main/main.cc` for the board key GPIO.
- `UART0_IRQHandler` through `UART3_IRQHandler` are owned by `bsp/src/uart.c`.
- `TIMG8_IRQHandler` is owned by `bsp/src/uart.c`; the generated
  `UART_RX_IDLE` one-shot timer provides the UART RX idle timeout.
- `SVC_Handler`, `PendSV_Handler`, and `SysTick_Handler` are owned by the
  FreeRTOS port.
- All other startup-vector handlers are weak defaults until a module provides
  the exact symbol.

The generated configuration currently uses an 80 MHz CPU clock, a 1 kHz
FreeRTOS tick, UART0 as `UART_DEBUG_INST`, and SPI1 for the LCD and W25Q128.
These are orientation notes only; after any SysConfig change, the generated
header and `FreeRTOSConfig.h` are the authoritative values.

## Build And Flash

Use an ARM embedded GCC toolchain.

```powershell
cmake -S . -B cmake-build-debug -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build cmake-build-debug
```

The normal output files are:

- `cmake-build-debug/trobot.elf`
- `cmake-build-debug/trobot.hex`
- `cmake-build-debug/trobot.bin`
- `cmake-build-debug/trobot.map`

The app and BSP source lists use recursive CMake globs without
`CONFIGURE_DEPENDS`. After adding, removing, or renaming a source file, rerun
the CMake configure command before building. Adding or removing a component
directory also requires reconfiguration.

This project needs an OpenOCD build with TI MSPM0 support. Before flashing,
check which OpenOCD executable is active:

```powershell
where openocd
openocd --version
```

Use the first `where openocd` result to understand which executable will run.
Then verify that it can resolve this project's MSPM0 target scripts. A quick
script-resolution check that should not initialize the adapter is:

```powershell
openocd -f daplink.cfg -c "shutdown"
```

With a DAPLink/CMSIS-DAP probe connected:

```powershell
cmake --build cmake-build-debug --target flash_and_verify
```

For other probes, use the matching root config manually:

```powershell
openocd -f xds110.cfg -c "init" -c "reset halt" -c "program cmake-build-debug/trobot.elf verify reset exit"
openocd -f stlink.cfg -c "init" -c "reset halt" -c "program cmake-build-debug/trobot.elf verify reset exit"
```

If OpenOCD reports that it cannot find `target/ti_mspm0.cfg`, the wrong OpenOCD
is being used or its script search path is wrong. If it reports that it cannot
find a matching CMSIS-DAP device, the MSPM0 scripts were found and the remaining
problem is probe/USB/driver/hardware related.

## SysConfig Rules

Treat `core/trobot.syscfg` as the peripheral configuration source, but do not
modify it by default. Only edit `core/trobot.syscfg` when the user explicitly
asks for a change that requires SysConfig, such as pins, clocks, DMA,
UART/SPI/I2C/ADC/timer setup, or interrupt ownership.

When a user-requested change does require SysConfig:

1. Edit `core/trobot.syscfg` with TI SysConfig when possible.
2. Preserve the metadata comments at the top of the file, including `@cliArgs`,
   `@v2CliArgs`, `@versions`, device, package, and SDK product.
3. Regenerate the SysConfig outputs into `core/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lym12321/trobot_m0](https://github.com/lym12321/trobot_m0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
