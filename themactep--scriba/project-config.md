---
trigger: always_on
description: handles the NAND case correctly.
---

# Scriba — AGENTS.md

A direct reference for AI coding agents working on Scriba. No fluff. No hedging.

## What Is This

Scriba is a **flash programming tool for IP camera chips**. It talks SPI NOR, SPI NAND,
and EEPROM through cheap USB programmers (CH341A, EZP2019). Two targets:

- **CLI** — native C binary, `gcc` + `libusb-1.0`, runs on Linux.
- **Web** — same C code compiled to WASM via Emscripten, WebUSB bridge in JS,
  Bootstrap UI, Vite bundler. Runs in Chromium-based browsers.

Origin: forked from [SNANDer](https://github.com/McMCCRU/SNANDer) by McMCC,
modified by Droid-MAX, heavily reworked by Paul Philippov (`themactep`).
License: GPL-2.0-or-later.

---

## Architecture

### Layering (bottom → top)

```
┌──────────────────────────────────────────────────┐
│  main.c  (CLI)         web_main.c  (WASM API)    │  ← entry points
├──────────────────────────────────────────────────┤
│  flashcmd_api.c   —  chip-type dispatch           │  ← picks NOR/NAND/EEPROM
├──────────────────┬───────────────┬───────────────┤
│  spi_nor_flash   │ spi_nand_flash│ *eeprom.c      │  ← chip families
│                  │   + protocol  │   + microwire  │
│                  │   + tables    │                │
├──────────────────┴───────────────┴───────────────┤
│  spi_controller.c  —  programmer dispatch         │  ← CS, read/write bytes
├──────────────────────┬───────────────────────────┤
│  ch341a_spi.c        │  ezp2019_spi.c             │  ← USB programmer drivers
│  libusb-1.0          │  libusb-1.0               │
└──────────────────────┴───────────────────────────┘
```

### Entry Points

| File | Role |
|---|---|
| `src/main.c` | CLI: `getopt_long`, dispatch to `flashcmd_api`, `do_verify` helper |
| `web/src/web_main.c` | WASM: flat C API (`scriba_read_flash`, etc.), no CLI parsing |

Both use the same `struct flash_cmd` interface — `flash_read` / `flash_write` /
`flash_erase` function pointers set by `flash_cmd_init()`.

### `struct flash_cmd` (in `flashcmd_api.h`)

```c
struct flash_cmd {
    int (*flash_read)(unsigned char *buf, unsigned long from, unsigned long len);
    int (*flash_erase)(unsigned long offs, unsigned long len);
    int (*flash_write)(unsigned char *buf, unsigned long to, unsigned long len);
};
```

`flash_cmd_init()` probes in order:
1. SPI NOR (`snor_init`) — if it returns >0 bytes
2. SPI NAND (`snand_init`) — fallback
3. EEPROM variants (I2C, Microwire, SPI) — if `-E` was given on CLI

Whichever succeeds first fills in the `cmd` struct and the caller never knows
which flash type is underneath.

### Programmer Dispatch

`spi_controller.c` holds a global `programmer_type` enum:
- `PROGRAMMER_CH341A = 0`
- `PROGRAMMER_EZP2019 = 1`
- `PROGRAMMER_AUTO = 2` (tries EZP first, then CH341A)

Every `SPI_CONTROLLER_*` call checks this global and routes to the right
driver. There is no vtable or function-pointer indirection at this layer —
just `if/else` on the global. The flash layers call `SPI_CONTROLLER_*`
functions, never the driver functions directly.

### USB Drivers

- **ch341a_spi.c** (26 KB, most complex): bit-bangs SPI over CH341A USB-I2C/SPI
  bridge IC. Also has I2C (`ch341a_i2c.c`), GPIO (`ch341a_gpio.c`), and
  Microwire (`bitbang_microwire.c`) support for EEPROM variants.
- **ezp2019_spi.c** (22 KB): packet protocol over bulk USB endpoints for the
  EZP2019/2023 programmer family. Simpler than CH341A — dedicated SPI commands.

Both expose the same three functions used by `spi_controller`:
`*_spi_init()`, `*_spi_shutdown()`, `*_spi_send_command(writecnt, readcnt, writearr, readarr)`.

### Global State

Several globals cross-cut the codebase. All defined in `main.c` and declared
`extern` where needed:

| Variable | File | Meaning |
|---|---|---|
| `programmer_type` | `spi_controller.c` | Which programmer is active |
| `debug_enabled`, `trace_enabled` | `main.c` | USB/SPI debug logging |
| `bsize` | `spi_nor_flash.c` | Flash block erase size (set at init) |
| `ECC_fcheck`, `ECC_ignore`, `OOB_size`, `Skip_BAD_page`, `_ondie_ecc_flag` | NAND subsystem | ECC control flags |
| `eepromsize`, `mw_eepromsize`, `seepromsize`, `spage_size`, `org`, `fix_addr_len` | EEPROM subsystem | EEPROM parameters |
| `spi_chip_info` | NOR subsystem | Pointer to detected chip's table entry |

No global mutex. Single-threaded. No reentrancy concerns for the CLI, but the
WASM build uses ASYNCIFY (Emscripten) — libusb calls are async-suspended.

---

## Source File Map

```
src/
├── main.c                     CLI entry, option parsing, operation dispatch
├── flashcmd_api.c / .h        Flash-type detection, cmd struct init
├── spi_controller.c / .h      Programmer abstraction, CS, byte r/w
├── ch341a_spi.c / .h          CH341A USB-SPI driver
├── ch341a_i2c.c / .h          CH341A I2C bit-bang (EEPROM)
├── ch341a_gpio.c / .h         CH341A GPIO helpers
├── ezp2019_spi.c / .h         EZP2019/2023 USB-SPI driver
├── spi_nor_flash.c / .h       SPI NOR commands, chip table, erase/write
├── spi_nand_flash.c / .h      SPI NAND core
├── spi_nand_flash_protocol.c  NAND protocol layer
├── spi_nand_flash_tables.c    NAND chip ID table (73 KB — largest file)
├── spi_nand_flash_defs.h      NAND command/register definitions
├── spi_nand_flash_feature.h   NAND feature register helpers
├── timer.c / .h               Progress display, elapsed time

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [themactep/scriba](https://github.com/themactep/scriba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
