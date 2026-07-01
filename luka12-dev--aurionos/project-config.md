---
trigger: always_on
description: This document provides context and guidance for working with the AurionOS codebase. AurionOS is a complete operating system built from scratch in x86 assembly and C - no Linux kernel, no POSIX, no borrowed code.
---

# AurionOS - Claude Development Guide

This document provides context and guidance for working with the AurionOS codebase. AurionOS is a complete operating system built from scratch in x86 assembly and C - no Linux kernel, no POSIX, no borrowed code.

---

## Project Overview

AurionOS is a modern, bootable operating system featuring:

- Custom bootloader and kernel written in x86 assembly
- Desktop environment with windowed GUI
- Dual-mode operation (GUI and DOS modes)
- Custom filesystem with persistence
- Network stack (TCP/IP, DHCP, HTTP/HTTPS)
- Built-in applications (browser, terminal, paint, calculator, games)
- AurionGL 3D graphics library (software rasterization)
- Blaze browser engine (HTML/CSS/JavaScript)

**Target**: Real hardware and virtual machines (QEMU, VMware, VirtualBox)
**Architecture**: x86 32-bit protected mode
**Language**: Assembly (NASM) + C11 (freestanding)

---

## Build System

### Platform Detection

The Makefile automatically detects the platform:
- **Windows**: Native Windows build (rarely used)
- **WSL2**: Linux tools with Windows QEMU for CD-ROM (avoids QEMU 8.2.2 crash)
- **Linux**: Native Linux build

### Build Commands

```bash
# Standard build (creates floppy, ISO, HDD images)
make all

# Debug build (skips installer, pre-installed system)
make all-debug

# Run in QEMU
make run          # Floppy + HDD
make run-iso      # ISO + HDD (CD-ROM boot)
make run-debug    # With CPU debug output

# Clean
make clean        # Preserves HDD (persistent storage)
make clean-all    # Removes everything including HDD
make reset-hdd    # Clear HDD but keep structure
```

### Build Artifacts

- `build/bootload.bin` - 512-byte boot sector
- `build/kernel.bin` - Flat kernel binary
- `build/aurionos.img` - 1.44MB floppy image
- `build/aurionos.iso` - Bootable ISO (El Torito)
- `build/aurionos_hdd.img` - 60MB persistent hard disk
- `build/aurionos_debug.iso` - Debug ISO (skips installer)

### Important Build Notes

1. **Always use WSL on Windows**: `wsl make all`
2. **HDD persistence**: The HDD image preserves user data across reboots
3. **Icon embedding**: Icons are embedded at LBA 20000 in HDD/ISO
4. **Wallpaper embedding**: Wallpaper embedded at specific LBA for installer
5. **Debug builds**: Use `make all-debug` to skip installer and boot directly to desktop

---

## Architecture

### Boot Process

```
BIOS loads bootloader (sector 0)
  ↓
Bootloader loads kernel from disk
  ↓
Switch to 32-bit protected mode
  ↓
Initialize IDT, PIC, memory, drivers
  ↓
Check boot_mode_flag
  ↓
├─ GUI Mode: VESA graphics → Desktop
└─ DOS Mode: VGA text → Shell
```

### Memory Layout

```
0x00007C00 - Bootloader (512 bytes)
0x00010000 - Kernel loaded here
0x000A0000 - VGA memory
0x00100000+ - Extended memory (heap)
```

### Key Components

**Boot Layer**
- `src/bootload.asm` - Bootloader (loads kernel, switches to protected mode)
- `src/kernel.asm` - Kernel entry point
- `src/memory.asm` - Memory management
- `src/interrupt.asm` - IDT and ISRs
- `src/vesa.asm` - VESA VBE mode setup

**Core System**
- `src/shell.c` - DOS mode shell
- `src/commands.c` - Shell command implementations
- `src/desktop.c` - Desktop environment
- `src/window_manager.c` - Window management
- `src/menu_bar.c` - Desktop menu bar
- `src/terminal.c` - Terminal application
- `src/installer.c` - Installation wizard
- `src/boot_screen.c` - Boot splash screen
- `src/login_screen.c` - Login screen
- `src/panic.c` - Kernel panic handler

**Drivers**
- `src/drivers/vbe_graphics.c` - VESA VBE graphics
- `src/drivers/vmware_svga.c` - VMware SVGA acceleration
- `src/drivers/mouse.c` - PS/2 mouse driver
- `src/drivers/ata.c` - ATA disk driver
- `src/drivers/ne2000.c` - NE2000 network card
- `src/drivers/rtl8139.c` - RTL8139 network card
- `src/drivers/bmp.c` - BMP image loader
- `src/drivers/png.c` - PNG image loader
- `src/drivers/icons.c` - Icon management

**Applications**
- `src/gui_apps.c` - GUI application framework
- `src/app_3d_demo.c` - 3D demo application
- `src/Blaze/` - Blaze browser engine
  - `blaze_core.c` - Core engine
  - `blaze_html.c` - HTML parser
  - `blaze_css.c` - CSS parser
  - `blaze_layout.c` - Layout engine
  - `blaze_render.c` - Rendering engine
  - `blaze_js.c` - JavaScript interpreter
  - `blaze_net.c` - Network integration

**Graphics**
- `AurionGL/auriongl.c` - 3D graphics library
- `AurionGL/auriongl.h` - API header
- Software rasterization with z-buffering
- OpenGL-inspired immediate mode API

**Network**
- `src/Network/virtio_net.c` - VirtIO network driver
- `src/Network/wifi_driver.c` - WiFi driver
- `src/Network/http_full.c` - HTTP client
- `src/Network/https_client.c` - HTTPS client
- `src/Network/tls12_client.c` - TLS 1.2 implementation
- `src/tcp_ip_stack.c` - TCP/IP stack
- `src/dhcp_client.c` - DHCP client

**Filesystem**
- `src/filesys.asm` - Filesystem implementation
- `src/iso9660.c` - ISO 9660 CD-ROM filesystem
- Custom persistent filesystem on HDD

---

## Code Style

### Assembly (NASM)

- Intel syntax
- Labels end with colon
- Comments explain WHY, not WHAT
- Use meaningful label names

```asm
; Good
load_kernel:
    mov si, loading_msg

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Luka12-dev/AurionOS](https://github.com/Luka12-dev/AurionOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
