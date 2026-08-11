---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OguzOS is a minimal ARM64 (AArch64) operating system written in freestanding C++17 with no standard library. It targets QEMU's `virt` machine (cortex-a72) and UTM on macOS.

## Build Commands

```bash
make              # Build kernel → produces build/oguzos.bin
make run          # Build and run with QEMU text-only (exit: Ctrl+A then X)
make gui          # Build and run with QEMU graphical mode (ramfb + mouse + keyboard)
make debug        # Run with QEMU GDB server (-S -s) for debugging
make dump         # Disassemble the ELF
make clean        # Remove build/ directory
make distclean    # Also remove disk.img (forces fresh filesystem on next boot)
```

**Toolchain:** Requires an AArch64 cross-compiler (`aarch64-elf-g++`, `aarch64-none-elf-g++`, or `aarch64-linux-gnu-g++`). The Makefile auto-detects which is available. Override with `CROSS=aarch64-linux-gnu- make`.

**Important:** After changing filesystem layout or config file formats, run `make distclean` to delete the old `disk.img` — otherwise `load_from_disk()` restores the stale image and skips `fs::init()`.

## Project Structure

```
arch/       — ARM64 bootstrap (boot.S), exception vectors (exception.S), linker script
kernel/     — Kernel entry point (kernel_main)
drivers/    — PL011 UART, Virtio block/net, ramfb, virtio-tablet, virtio-keyboard
fs/         — In-memory hierarchical file system (128 nodes, 4KB/file)
net/        — Network stack (ARP, IPv4, ICMP, UDP, DHCP, DNS, HTTP, NTP)
gui/        — Window manager, desktop, start menu, file explorer
apps/       — GUI applications (.ogz.cpp files): notepad, terminal, task manager, settings, browser, C# IDE, C# GUI host
shell/      — UART shell, shared command library (commands.cpp)
lib/        — String/memory utils, syslog, settings, env vars, file assoc, menu config
lang/       — Mini C# interpreter (console + GUI modes) with widget system
scripts/    — QEMU launcher (run.sh) and UTM image builder (mkimage.sh)
build/      — Generated object files and binaries (gitignored)
```

## Architecture

### Boot Flow

`arch/boot.S` → `kernel_main()` (kernel/kernel.cpp):
1. UART, disk, framebuffer, mouse, keyboard init
2. Network init (virtio-net + DHCP + NTP)
3. Filesystem: load from disk or `fs::init()` (creates `/bin`, `/home`, `/etc`, `/tmp`, `/var`)
4. `settings::load()`, `env::init()`, `assoc::init()`+load, `syslog::init()`
5. Register apps → populate `/bin` with app descriptors → build `/etc/menu` defaults
6. `menu::init()`+load
7. If framebuffer available → `gui::run()`, else → `shell::run()`

### Two Shells — Shared Command Library

The OS has two independent command dispatchers:
- **UART shell** (`shell/shell.cpp` `execute()`) — runs in text-only mode or as serial fallback
- **GUI terminal** (`apps/terminal.ogz.cpp` `term_exec()`) — runs as a windowed app inside the GUI

Both call the same implementations in `shell/commands.cpp` via a callback-based output abstraction:
```cpp
using OutFn = void (*)(void *ctx, const char *text);
// UART: uart_out(nullptr, text) → uart::puts(text)
// Terminal: term_out(state, text) → term_append(state, text)
```

**When adding a new command:** implement it in `commands.cpp`/`commands.h`, then add dispatch entries to BOTH `shell.cpp` AND `terminal.ogz.cpp`. The terminal also needs `uart::capture_start/stop` for commands that print directly to UART (e.g., `net::ping()`).

### Configuration Subsystems

Three runtime-configurable subsystems persist to `/etc/` and auto-save to disk:

| System | File | Header | Purpose |
|--------|------|--------|---------|
| `settings::` | `/etc/settings` | `lib/settings.h` | Timezone, desktop color, keyboard layout |
| `assoc::` | `/etc/filetypes` | `lib/assoc.h` | File extension → app ID mapping |
| `menu::` | `/etc/menu` | `lib/menu.h` | Start menu entries (apps, shortcuts, commands, built-ins) |

All three call `fs::sync_to_disk()` on save. `env::` (environment variables) is in-memory only.

### GUI File Opening Flow

When a file is double-clicked in the explorer:
1. `explorer_activate()` builds the absolute path
2. `gui::open_file(path, content)` checks:
   - `.ogz` extension → `gui::open_app(name)` (launch the app)
   - `assoc::find_for_file(name)` → if app has `on_open_file` callback, open app with file content
   - Fallback → `open_text_viewer(name, content)`

### Start Menu

The start menu is data-driven from `menu::` config (not auto-discovered from the app registry). Entry types: `ENTRY_APP`, `ENTRY_SHORTCUT`, `ENTRY_COMMAND`, `ENTRY_SEP`, `ENTRY_EXPLORER`, `ENTRY_ABOUT`, `ENTRY_SHUTDOWN`, `ENTRY_RESTART`. The GUI calls `build_menu()` at startup to populate the display from `menu::` entries.

## Adding New Source Files

Object files are listed explicitly in the Makefile `OBJS` list (not auto-discovered). Some output names differ from source names to avoid basename collisions:

- `drivers/net.cpp` → `build/netdev.o` (namespace `netdev`, header: `drivers/netdev.h`)
- `net/net.cpp` → `build/netstack.o` (namespace `net`, header: `net/net.h`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oguzgundogdu/OguzOS-arm](https://github.com/oguzgundogdu/OguzOS-arm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
