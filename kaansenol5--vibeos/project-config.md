---
trigger: always_on
description: **IMPORTANT DISK RULES:**
---

# VibeOS - Claude Context

**IMPORTANT DISK RULES:**
- Never look at the disk
- The disk always works
- `make run` is the only way to compile and run the code - the user will run it
- Trust `make run` - the user will tell you if it is broken
- Do NOT use hdiutil for anything

## Project Overview
VibeOS is a hobby operating system built from scratch for aarch64 (ARM64), targeting QEMU's virt machine. This is a science experiment to see what an LLM can build.

## The Vibe
- **Aesthetic**: Modern macOS-inspired (draggable windows, dock, menu bar)
- **Philosophy**: Simple, educational, nostalgic
- **NOT trying to be**: Linux, production-ready, or modern

## Current State (Last Updated: Session 40)
- [x] Bootloader (boot/boot.S) - Sets up stack, clears BSS, jumps to kernel
- [x] Minimal kernel (kernel/kernel.c) - UART output working
- [x] Linker script (linker.ld) - Memory layout for QEMU virt
- [x] Makefile - Builds and runs in QEMU
- [x] Boots successfully! Prints to serial console.
- [x] Memory management (kernel/memory.c) - malloc/free working, dynamic heap sizing
- [x] DTB parsing (kernel/dtb.c) - Detects RAM size at runtime from Device Tree
- [x] String functions (kernel/string.c) - memcpy, strlen, strcmp, strtok_r, etc.
- [x] Printf (kernel/printf.c) - %d, %s, %x, %p working
- [x] Framebuffer (kernel/fb.c) - ramfb device, 800x600
- [x] Bitmap font (kernel/font.c) - 8x16 VGA-style font
- [x] Console (kernel/console.c) - Text console with colors on screen
- [x] Virtio keyboard (kernel/keyboard.c) - Full keyboard with shift support
- [x] Shell (kernel/shell.c) - In-kernel shell with commands
- [x] VFS (kernel/vfs.c) - Now backed by FAT32, falls back to in-memory
- [x] Coreutils - ls, cd, pwd, mkdir, touch, rm, cat, echo (with > redirect)
- [x] ELF loader (kernel/elf.c) - Loads PIE binaries with full relocation support
- [x] Process management (kernel/process.c) - Process table, context switching, scheduler
- [x] Cooperative multitasking - yield(), spawn() in kapi, round-robin scheduler
- [x] Kernel API (kernel/kapi.c) - Function pointers for programs to call kernel
- [x] Text editor (kernel/vi.c) - Modal vi clone with normal/insert/command modes
- [x] Virtio block device (kernel/virtio_blk.c) - Read/write disk sectors
- [x] FAT32 filesystem (kernel/fat32.c) - Read/write, full LFN (long filename) support
- [x] Persistent storage - 64MB FAT32 disk image, mountable on macOS
- [x] Interrupts - GIC-400 working! Keyboard via IRQ, boots at EL3 (Secure)
- [x] Timer - 10ms tick (100Hz), used for uptime tracking
- [x] System Monitor - GUI app showing uptime and memory usage
- [x] TextEdit - GUI text editor with Save As modal
- [x] RTC - PL031 real-time clock at 0x09010000, shows actual date/time
- [x] Date command - /bin/date shows current UTC date/time
- [x] Menu bar - Apple menu with About/Quit, File menu, Edit menu
- [x] About dialog - Shows VibeOS version, memory, uptime
- [x] Power management - WFI-based idle, mouse interrupt-driven, 100Hz UI refresh
- [x] Virtio Sound - Audio playback via virtio-sound device, WAV and MP3 support
- [x] Music Player - GUI music player with album/track browser, pause/resume, progress bar
- [x] Floating point - FPU enabled, context switch saves/restores FP regs, calc uses doubles
- [x] Networking - virtio-net driver, Ethernet, ARP, IP, ICMP working!
- [x] Ping command - `/bin/ping` can ping internet hosts (1.1.1.1, etc.)
- [x] UDP + DNS - hostname resolution via QEMU's DNS server (10.0.2.3)
- [x] TCP - full TCP state machine with 3-way handshake, send/recv, close
- [x] HTTP client - `/bin/fetch` can make HTTP requests to real websites!
- [x] Web Browser - `/bin/browser` GUI browser with HTML rendering, works on HTTP sites
- [x] TLS 1.2 - Full HTTPS support via TLSe library, ECC key exchange working!
- [x] HTTPS client - `/bin/fetch` can make HTTPS requests to google.com, etc!
- [x] Raspberry Pi Zero 2W support - boots on real hardware!
- [x] Pi SD card (EMMC) driver - FAT32 filesystem works, userspace runs!
- [x] Pi USB host (DWC2) - Device enumeration working, HID keyboard via hub works!

## Architecture Decisions Made
1. **Target**: QEMU virt machine, aarch64, Cortex-A72
2. **Memory start**: 0x40000000 (QEMU virt default)
3. **UART**: PL011 at 0x09000000 (QEMU virt default)
4. **Stack**: 64KB, placed in .stack section after BSS
5. **Toolchain**: aarch64-elf-gcc (brew install)
6. **Compiler flags**: -mstrict-align (prevent unaligned SIMD), FPU enabled
7. **Process model**: Win3.1 style - no memory protection, programs run in kernel space

## Roadmap (Terminal-First)
Phase 1: Kernel Foundations - DONE
1. ~~Memory management~~ - heap allocator working
2. ~~libc basics~~ - string functions, sprintf
3. ~~Display~~ - framebuffer, console, font
4. ~~Keyboard~~ - virtio-input with shift keys
5. ~~Shell~~ - in-kernel with basic commands
6. ~~Filesystem~~ - in-memory VFS

Phase 2: Programs - MONOLITH APPROACH
7. ~~ELF loader~~ - working but abandoned
8. ~~Kernel API~~ - kapi struct with function pointers
9. **DECISION**: Monolith kernel - all commands built into shell
   - Tried external programs, hit linker issues with 6+ embedded binaries
   - Win3.1 vibes - everything in one binary is fine

Phase 3: Apps (DONE)
10. ~~Text editor~~ - vi clone with modal editing (normal/insert/command modes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaansenol5/VibeOS](https://github.com/kaansenol5/VibeOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
