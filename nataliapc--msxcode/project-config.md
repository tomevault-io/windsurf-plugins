---
trigger: always_on
description: **msxcode** is an MSX 2/2+/tR application written in C (SDCC compiler) that acts as an AI assistant client
---

# AGENTS.md — MSXcode Project Guide

## Project Overview

**msxcode** is an MSX 2/2+/tR application written in C (SDCC compiler) that acts as an AI assistant client
running on real MSX hardware. It communicates with AI backends over TCP/IP UNAPI and displays a TUI
(Text User Interface) using a custom ANSI widget library.

- **Target hardware**: MSX 2/2+/TurboR (Z80 compatible)
- **OS**: MSX-DOS 2.x or higher
- **Compiler**: SDCC 4.2.0 via Docker (`nataliapc/sdcc:4.2.0`)
- **Output**: `dsk/msxcode.com` (MSX-DOS executable)
- **Screen**: V9938 / V9958 VDP, Screen 7 (G7, 512×212, 16 colors), 85×26 text columns via msx2ansi

---

## Directory Structure

```
tool_msxcode/
├── src/                    Main application source files (C/ASM)
│   ├── msxcode.c           Entry point, main loop
│   ├── mod_disposable.c    Init, screen setup, printHeader() — DISPOSABLE segment
│   ├── mod_commandLine.c   Command-line argument parsing
│   ├── mod_help.c          Help screen
│   ├── heap.c              Heap allocator
│   ├── crt0msx_msxdos_advanced.s  Custom CRT0 for MSX-DOS
│   └── libs/               Library source files
│       ├── acmp_*.c        ANSI TUI widget implementations
│       ├── vdp_*.c/.s      VDP helper routines
│       └── utils_*.c       Utility functions
├── includes/               Header files
│   ├── ansi_components.h   TUI widget API declarations
│   ├── ansi_codes.h        ANSI/VT52/VT100 escape code macros
│   ├── msx2ansi.h          msx2ansi library API
│   ├── msx_const.h         MSX system constants and variables
│   └── ...
├── contrib/                Third-party libraries (source + prebuilt)
│   └── msx2ansi/           ANSI rendering library for V9938
│       ├── src/
│       │   ├── msx2ansi.asm        Main ANSI engine (Z80 ASM)
│       │   ├── msx2ansibuffer.asm  Buffer helper
│       │   └── msx2ansi.sh         Build script (runs inside Docker)
│       └── msx2ansi.lib    Built library (copied to libs/ by make)
├── externals/              Git submodules
│   ├── sdcc_msxconio/      conio (console I/O) library
│   └── sdcc_msxdos/        MSX-DOS 2 API library
├── libs/                   Compiled libraries used by the linker
│   ├── msx2ansi.lib        ANSI rendering engine
│   ├── ansi_components.lib TUI widgets
│   ├── vdp.lib             VDP helpers
│   ├── utils.lib           Utility functions
│   ├── conio.lib           Console I/O
│   └── dos.lib             MSX-DOS 2 API
├── dsk/                    Disk image folder (mounted in openMSX)
│   └── msxcode.com         Final compiled executable
├── plan/                   Implementation plans (PRPs)
│   ├── PRP001_acmp_label.md
│   ├── PRP002_acmp_panel.md
│   ├── PRP003_acmp_progressBar.md
│   ├── PRP004_acmp_lines.md
│   ├── PRP005_acmp_badge.md
│   ├── PRP006_acmp_menu.md
│   ├── PRP007_acmp_confirm.md
│   ├── PRP008_acmp_inputBox.md
│   ├── PRP009_acmp_textArea.md
│   └── PRP010_hpost.md
├── obj/                    Compiler output (.rel, .ihx, .map, .com)
├── res/                    Resources (logo, help text — compressed with zx0)
├── emulation/              openMSX boot scripts
└── Makefile                Main build system
```

---

## Build System

All compilation uses SDCC 4.2.0 via Docker (`nataliapc/sdcc:4.2.0`).
`hex2bin` must be installed locally to convert `.ihx` → `.com`.

### Build the full project (most common)

```sh
make
```

Compiles everything: contrib libs → externals → src libs → application.
Output: `dsk/msxcode.com`

### Rebuild only the application (after changing src/*.c)

```sh
make
```

The Makefile detects changed files automatically via dependencies.

### Choosing the right clean target (IMPORTANT — avoid unnecessary cleans)

Use the minimal clean needed to avoid wasting compilation time. Combine targets when multiple
areas changed. **Never use `make clean` unless everything is broken.**

| What changed | Clean command before `make` |
|---|---|
| Only `src/*.c` / `src/*.s` (main program files) | `make cleanprogram` |
| Only `src/libs/*.c` or `includes/*.h` (project libs) | `make cleanprogram` |
| Only `res/` (resources / help text) | `make cleanres` |
| Only `contrib/msx2ansi/src/msx2ansi.asm` | `make cleancontrib` |
| Only `contrib/UNAPI_TCPIP/src/*.c` | `make cleancontrib` |
| Any other contrib lib | `make cleancontrib` |
| `externals/sdcc_msxconio` or `externals/sdcc_msxdos` | `make cleanlibs` |
| Mix of contrib + project libs | `make cleancontrib cleanprogram` |
| Mix of resources + program | `make cleanres cleanprogram` |
| Everything broken / first build | `make clean` |

**`cleanprogram`** removes only the compiled `.rel` files for the main program and project libs
(listed in `REL_LIBS`), forcing them to relink without touching extern/contrib artifacts.

**`cleancontrib`** removes only contrib build artifacts (`contrib/` subtree + `libs/` copies of
contrib libs such as `msx2ansi.lib`, `unapi_tcpip.lib`). Always required after editing any file
under `contrib/`.

**`cleanlibs`** removes all libs including externals (`conio.lib`, `dos.lib`) plus everything
`cleancontrib` removes. Only needed when externals change.

**`cleanres`** removes compiled resource `.c` files and `utils.lib` (which embeds the resources).
Required after editing anything in `res/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nataliapc/msxcode](https://github.com/nataliapc/msxcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
