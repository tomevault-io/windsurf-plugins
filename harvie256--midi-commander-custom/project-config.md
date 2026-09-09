---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Replacement firmware for the MeloAudio MIDI Commander (a MIDI foot controller), plus the host-side tooling to configure it. Three pieces that must stay in sync:

1. **Firmware** — `MIDI_Commander_Custom/`, an STM32CubeIDE project for an STM32F103RET (256 KiB+ flash, 12 MHz crystal, SSD1306 OLED on I2C1 @0x3C, MIDI DIN out on USART2).
2. **Config tool** — `python/CSV_to_Flash.py`, packs a CSV config into the firmware's binary layout and pushes it over USB MIDI SysEx.
3. **DFU packaging** — `DFU/`, Windows ST DfuSe tools plus `BuildDFUAutomation.py` to wrap a build into a `.dfu`.

There is no test suite anywhere in the repo. Verification is done on hardware.

## Building

**Firmware:** STM32CubeIDE only — there is no Makefile or CMakeLists. Import `MIDI_Commander_Custom/` as an existing project. Two build configurations:

| Config | Linker script | Flash origin | Use |
|---|---|---|---|
| `Debug` | `STM32F103RETX_FLASH.ld` | `0x8000000` | ST-Link debugging via SWD (see `HardwareNotes.txt` for the P3 pinout) |
| `DFU Release` | `STM32F103RETX_FLASH_DFU.ld` | `0x8003000` | Loading over the stock ST DFU bootloader |

The DFU offset lives in **two** places that must move together: `ORIGIN` in `STM32F103RETX_FLASH_DFU.ld` and `VECT_TAB_OFFSET` (`0x3000`) in `Core/Src/system_stm32f1xx.c`. `DFU Release` emits `.hex`, `.bin`, and `.elf`.

**Flashing a build:**
- Windows: `python DFU/BuildDFUAutomation.py` (needs `pywinauto`; drives `DfuFileMgr.exe` through its GUI, consumes the `.hex`), then `DFU/DownloadToMidiCommandByDFU.bat`.
- macOS/Linux: `dfu-util --alt 0 -s 0x8003000 --download "MIDI_Commander_Custom/DFU Release/MIDI_Commander_Custom.bin"` — pass the load address explicitly since `dfu-util` can't build a `.dfu`.
- Device enters DFU mode by holding `bank down` + `D` while pressing power.

**Python tooling:**

```
python3 -m pip install -r python/requirements.txt
python3 python/CSV_to_Flash.py <config.csv>
```

On Linux, `python-rtmidi` needs `libjack-dev` and `libasound2-dev` first — see `python/linux-prerequisites.sh`. Python is formatted with Black and type-checked with mypy in basic mode (`.vscode/`); mido's dynamic symbols require `# type: ignore` on its call sites.

## Architecture

### Firmware control flow

Bare-metal super-loop, no RTOS. `main()` initialises peripherals and then does nothing but call `handle_switches()` forever. The real work is split between SysTick and that loop:

- **SysTick (1 kHz), in `stm32f1xx_it.c`** calls two hand-added hooks: `sw_scan()` and `ssd1306_tick()`. Switch pins are spread across GPIOA/B/C and not all are interrupt-capable, so switches are *polled* — each tick XORs the port against the previous state, ORs changes into `port_X_switches_changed`, and sets a 10 ms `debounce_counter`. The `f_sys_config_complete` flag gates scanning until init finishes.
- **Main loop** consumes those change flags in `switch_router.c` and dispatches MIDI.

Anything that blocks in the main loop delays switch handling and MIDI timing — this is why display updates are DMA-driven and why `midi_cmds.c` warns against drawing to the screen from the send path.

### Configuration storage and layout

Settings live in the MCU's **internal flash** (not the external EEPROM the stock firmware uses, so both firmwares coexist). `Core/Src/flash_midi_settings.c` fixes the region at `FLASH_BASE + 128 KiB`, spanning `FLASH_SETTINGS_NO_PAGES` (3) × 1 KiB pages, exposed as three pointers:

```
pGlobalSettings  +0     32 bytes   16 setting bytes + 16 ASCII config name
pBankStrings     +32    96 bytes   8 banks × (4-char large name + 8-char small name)
pSwitchCmds      +128   rest       8 pages × 8 switches × 10 commands × 4 bytes
```

Command lookup is pure pointer arithmetic — `get_rom_pointer(page, sw, cmd)` in `switch_router.c` and the equivalent inline expressions in `handle_switches()`. `MIDI_ROM_CMD_SIZE`, `MIDI_NUM_COMMANDS_PER_SWITCH`, and `MIDI_ROM_KEY_STRIDE` in `flash_midi_settings.h` define the stride.

### The C/Python wire contract

The 4-byte per-command encoding is **written** by `python/lib/cmdBinaryPacker.py` and **decoded** by `Core/Src/midi_cmds.c`. Nothing enforces agreement, so a change on one side needs a matching change on the other. Duplicated constants to keep aligned:

- Command-type nibbles (`CMD_PC_NIBBLE` etc.) — `Core/Inc/midi_defines.h` ↔ `cmdBinaryPacker.py`
- `MIDI_NUM_COMMANDS_PER_SWITCH` (10) — `flash_midi_settings.h` ↔ `cmdBinaryPacker.py`
- `GLOBAL_SETTINGS_*` indices — `midi_defines.h` ↔ `settingsBinaryPacker.py`
- `FLASH_SETTINGS_NO_PAGES` ↔ `ALLOWED_NUM_FLASH_PAGES` in `CSV_to_Flash.py` (the tool only warns on overflow)

Encoding conventions worth knowing: byte 0 is command nibble | channel; the toggle flag is the top bit of byte 1; the "no value" sentinel is `0x80` (used to suppress bank-select bytes in PC commands and the off-value in CC); note/PB duration is byte 3 in 10 ms units.

### Config transfer protocol (USB SysEx)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harvie256/midi-commander-custom](https://github.com/harvie256/midi-commander-custom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
