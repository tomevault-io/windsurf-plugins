---
trigger: always_on
description: This is a **production-ready Linux kernel driver** for the Yamaha ZG01 USB audio interface (VID: 0x0499, PID: 0x1513). The driver provides three independent audio channels (Game Output, Voice Output, Voice Input) as separate ALSA sound cards, with full-duplex 32-bit stereo audio at 48kHz. Distributed via DKMS package for automatic kernel module rebuild.
---

# Yamaha ZG01 Linux Kernel Driver - AI Coding Agent Instructions

## Project Overview

This is a **production-ready Linux kernel driver** for the Yamaha ZG01 USB audio interface (VID: 0x0499, PID: 0x1513). The driver provides three independent audio channels (Game Output, Voice Output, Voice Input) as separate ALSA sound cards, with full-duplex 32-bit stereo audio at 48kHz. Distributed via DKMS package for automatic kernel module rebuild.

## Critical Architecture Patterns

### Multi-Module Kernel Driver Structure
The driver splits functionality across **four separate kernel modules**:
- `zg01_usb.c`: USB device probe/disconnect, creates three separate sound cards (one per channel)
- `zg01_pcm.c`: PCM audio streaming (playback/capture), URB management, isochronous transfers
- `zg01_control.c`: ALSA control interface, device initialization magic sequence
- `zg01_usb_discovery.c`: USB hardware discovery and endpoint enumeration

**Critical**: All modules must be built from the **repository root** (not `src/`). The Makefile uses `obj-m := src/zg01_*.o` to reference source files in subdirectory while building from root. Building from `src/` causes kernel module relocation errors.

### Three Independent Channels = Three Sound Cards
Unlike standard USB audio drivers, this creates **three separate `struct zg01_dev`** instances via `zg01_probe()`:
- **Game channel** (Interface 1, EP 0x01 OUT): High-bandwidth playback, 240-byte packets, `card->id = "zg01game"`
- **Voice Out channel** (Interface 1, EP 0x01 OUT): Secondary playback, 240-byte packets, `card->id = "zg01voiceout"`
- **Voice In channel** (Interface 2, EP 0x81 IN): Capture, 108-byte packets (different format!), `card->id = "zg01voice"`

Global pointers (`game_dev`, `voice_in_dev`, `voice_out_dev`) track which cards exist. Each channel has independent URB arrays, PCM state, and cleanup flags.

### USB Packet Format - Critical Difference Between Channels
**Game/Voice Out** (240-byte packets): 6 frames × 40 bytes each (L+R+padding)
**Voice In** (108-byte packets): 8-byte header + (6 frames × 16 bytes) + 4-byte trailer
Each frame: L(4 bytes) + R(4 bytes) + 8 bytes padding

**Common Bug**: Using same packet parsing for both channels causes audio corruption. See [DEVELOPMENT_STATUS.md](../DEVELOPMENT_STATUS.md) section "Voice Channel Capture Format" for detailed packet structure.

### URB Lifecycle and Continuous Streaming
The driver uses **continuous isochronous streaming** with `MAX_URBS_PER_CHANNEL=16` (64ms buffering):
1. `TRIGGER_START`: Starts URB submission if not already streaming
2. URBs continuously resubmit themselves in callbacks, sending **silence when PCM not RUNNING**
3. `TRIGGER_STOP`: Calls `zg01_stop_streaming()` to actually stop URBs

**Critical Fix (Jan 2026)**: URBs must always resubmit until explicitly stopped. Previous bug: URBs stopped resubmitting when PCM state != RUNNING, causing USB stream death. Now they send silence when not RUNNING.

### PipeWire START/STOP Loop Prevention
PipeWire rapidly toggles TRIGGER_START/STOP during stream reconfiguration. Protection mechanisms:
- `cleanup_in_progress_{game,voice,voice_out}` flags prevent concurrent cleanup
- `last_trigger_jiffies` + `trigger_count` detect rapid loops (>3 triggers in 100ms)
- **TRIGGER_STOP must call `zg01_stop_streaming()`** - previous bug had STOP only mute URBs, causing state mismatch

## Development Workflow

### Build and Test (VM Recommended)
```bash
# Build all modules (MUST be from repo root)
make clean && make

# Load modules in dependency order
sudo insmod src/zg01_pcm.ko
sudo insmod src/zg01_control.ko
sudo insmod src/zg01_usb_discovery.ko
sudo insmod src/zg01_usb.ko

# Verify three sound cards created
cat /proc/asound/cards  # Should show zg01game, zg01voice, zg01voiceout

# Test playback
speaker-test -D hw:zg01game -c 2 -r 48000 -F S32_LE -t sine -f 440 -l 1

# Watch kernel logs
sudo dmesg -w | grep zg01
```

### DKMS Package Build
```bash
./scripts/build-deb.sh  # Creates ../snd-zg01-dkms_*.deb
sudo dpkg -i ../snd-zg01-dkms_*.deb
```

**DKMS Configuration** ([dkms.conf](../dkms.conf)): Builds 4 modules, installs udev rules from [src/90-zg01.rules](../src/90-zg01.rules) for automatic driver loading and device naming in PipeWire.

## Common Bugs and Pitfalls

### Multi-Channel Interference (Fixed Feb 7, 2026)
**Bug**: Opening Voice Out (Discord) kills Game channel playback
```c
// WRONG - Voice Out open() calls usb_set_interface(1,1) even if Game is streaming
if (!dev->voice_out_initialized) {
    usb_set_interface(dev->udev, 1, 1);  // Kills Game's Interface 1 URBs!
}

// CORRECT - Check if Game is streaming before ANY interface changes
struct zg01_dev **game_dev_ptr = (struct zg01_dev **)__symbol_get("game_dev");
bool game_streaming = (game_dev_ptr && *game_dev_ptr && (*game_dev_ptr)->active_urbs_game > 0);
if (game_dev_ptr) __symbol_put("game_dev");

if (game_streaming) {
    pr_info("Voice Out open - SKIPPING interface setup (Game streaming)");

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bsauvajon/snd-zg01](https://github.com/bsauvajon/snd-zg01) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
