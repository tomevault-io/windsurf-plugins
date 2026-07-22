---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Apple IIgs hardware simulation written in Verilog using the Verilator simulator. The project implements a cycle-accurate emulation of the Apple IIgs computer system, including the 65C816 CPU, video graphics controller (VGC), sound system (ES5503), and various peripherals.

The goal is to run this code on an FPGA. Don't make any changes to memory or clocks that aren't FPGA friendly. Make sure to use clock enables. Some clocks may be labeled incorrectly as clocks, but are actually enables.

## Essential Build Commands

**Always run from the `vsim/` directory due to relative file paths:**

```bash
cd vsim/
make                    # Build the simulation (ROM3 and ROM1 both built in)
make SOUND=stub        # Build with stubbed sound system
make clean             # Clean build artifacts
```

**ROM version is selected at runtime, not build time** (both ROMs are compiled
into the sim):

```bash
./obj_dir/Vemu                 # ROM3 (default)
./obj_dir/Vemu --rom 1         # ROM1  (also accepts --rom rom1 / --rom 3 / --rom rom3)
```

**Running the simulation:**
```bash
# Basic windowed simulation
./obj_dir/Vemu

# Show help and available options
./obj_dir/Vemu -h
./obj_dir/Vemu --help

# Take screenshot at frame 245
./obj_dir/Vemu --screenshot 245
./obj_dir/Vemu -screenshot 245   # Legacy format (deprecated)

# Take screenshots at multiple frames
./obj_dir/Vemu --screenshot 100,200,300

# Stop simulation after frame 1000
./obj_dir/Vemu --stop-at-frame 1000

# Take screenshot and stop at same frame
./obj_dir/Vemu --screenshot 245 --stop-at-frame 245

# Run with selftest mode
./obj_dir/Vemu --selftest

# With debug output to file
./obj_dir/Vemu > debug.log 2>&1

# VCD waveform capture (for signal-level debugging)
# IMPORTANT: VCD files grow very fast. Only capture 2-3 frames maximum.
# Use --stop-at-frame no more than 3 frames after the dump start frame.
./obj_dir/Vemu --dump-vcd-after 400 --stop-at-frame 403  # Capture frames 400-403 to vsim.vcd

# use a disk image (HDD slot 7)
./Vemu --disk totalreplay.hdv
./Vemu --disk pd.hdv --screenshot 50 --stop-at-frame 100

# use floppy disk images
./Vemu --floppy game.nib                # 5.25" floppy (NIB format, 140K)
./Vemu --woz ArkanoidIIgs.woz           # WOZ format flux-level disk image
```

### Disk Image Options

| Option | Description | Formats |
|--------|-------------|---------|
| `--disk <file>` | HDD slot 7 unit 0 | HDV, PO, 2MG |
| `--disk2 <file>` | HDD slot 7 unit 1 | HDV, PO, 2MG |
| `--woz <file>` | WOZ flux-level disk image | WOZ 1.x/2.x (3.5"/5.25") |

**Note:** WOZ format provides flux-level accuracy for copy-protected disks. WOZ 3.5" disk images boot successfully in simulation.

### WOZ Floppy Disk Support

The WOZ format provides bit-level accuracy for floppy disk emulation, supporting copy-protected software that relies on precise flux timing.

**Architecture:**
- `rtl/flux_drive.v` - Physical drive emulation (motor, head position, flux transitions)
- `rtl/iwm_woz.v` - IWM controller with WOZ/flux interface
- `rtl/iwm_flux.v` - Flux decoding and IWM register reads
- `vsim/sim.v` - Track data BRAM and C++ integration

### Keyboard Input (--send-keys)
Send keyboard input at specific frames for automated testing:
```bash
# Basic key injection at frame 100
./obj_dir/Vemu --send-keys 100:hello

# Special escape sequences:
#   \n  = Enter/Return
#   \t  = Tab
#   \e  = Escape (ESC key)
#   \\  = Literal backslash
#   \xNN = Hex code (e.g., \x1b for ESC, \x08 for backspace)

# Examples:
./obj_dir/Vemu --send-keys 100:b\n          # Type 'b' then press Enter
./obj_dir/Vemu --send-keys 100:\e           # Press Escape key
./obj_dir/Vemu --send-keys 100:test\nyes\n  # Type 'test', Enter, 'yes', Enter

# Multiple key sequences at different frames
./obj_dir/Vemu --send-keys 100:a --send-keys 200:b\n
```

### Mouse Input (--send-mouse)
Send mouse movements and clicks at specific frames:
```bash
# Format: --send-mouse <frame>:<dx>,<dy>[,<btn>[,<dur>]]
#   dx, dy  = Movement deltas (-127 to 127)
#   btn     = Button state: 0=none, 1=left click (optional, default 0)
#   dur     = Duration in frames to hold (optional, default 1)

# Move mouse right 50 pixels at frame 100
./obj_dir/Vemu --send-mouse 100:50,0

# Move mouse down 30 pixels with left button held
./obj_dir/Vemu --send-mouse 100:0,30,1

# Click and hold for 5 frames
./obj_dir/Vemu --send-mouse 100:0,0,1,5

# Multiple mouse actions
./obj_dir/Vemu --send-mouse 100:100,0 --send-mouse 150:0,0,1,5
```

## Core Architecture

### System Integration (rtl/iigs.sv)
- **Top-level module** integrating all subsystems
- **Memory controller** handling fast RAM (banks 00-3F) and slow RAM (banks E0-E1)
- **I/O space mapping** ($C000-$CFFF) including video mode switches
- **Clock domains**: 14MHz master clock, pixel clock for video
- **Key signals**: AN3 (graphics mode control), NEWVIDEO[7:0], video mode switches

### Video Graphics Controller (rtl/vgc.v)
- **Dual-mode architecture**: SHRG (Super Hi-Res Graphics) vs Apple II compatibility modes
- **Apple II video modes**: Text 40/80, Lores 40/80, Hires 40/80, mixed modes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiSTer-devel/Apple-IIgs_MiSTer](https://github.com/MiSTer-devel/Apple-IIgs_MiSTer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
