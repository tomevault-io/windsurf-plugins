---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

reSIDuEngine is a C++ implementation of the MOS 6581/8580 SID (Sound Interface Device) chip used in the Commodore 64. The project provides accurate emulation of the SID's waveform synthesis, ADSR envelopes, filters, and combined waveform behavior.

## Build System

This project uses CMake (minimum version 3.10) with C++17.

### Building from Scratch

```bash
mkdir -p build
cd build
cmake ..
cmake --build .
```

Or from project root:
```bash
cmake --build build
```

### Clean Build

```bash
rm -rf build
mkdir build
cd build
cmake ..
cmake --build .
```

### Build Outputs

- `build/libresiduengine.a` - Static library
- `build/examples/reSIDuEngine_example1` - Example program (generates WAV file)
- `build/examples/simple_test` - Simple test program
- `build/examples/sid_player` - SID file player with SDL2 (if SDL2 found)
- `build/examples/sid_player_portaudio` - SID file player with PortAudio (if PortAudio found)
- `build/examples/sid_player_miniaudio` - SID file player with miniaudio (if miniaudio.h found)
- `build/examples/sidplayer2` - Enhanced single-SID player with miniaudio, IRQ injection, NMI digi, and PAL/NTSC detection (if miniaudio.h found)

### Running Examples

```bash
# Example 1: Generates reSIDuEngine_output.wav (C major chord demo)
./build/examples/reSIDuEngine_example1

# Simple test: Generates output.raw
./build/examples/simple_test

# Play raw audio (requires aplay)
aplay -f S16_LE -r 44100 -c 1 output.raw

# SID Player: Play .sid files (Commodore 64 music format)
./build/examples/sid_player song.sid [subtune] [chip_model] [seconds]
./build/examples/sid_player_portaudio song.sid 1 8580
./build/examples/sid_player_miniaudio song.sid

# Download SID files from HVSC (High Voltage SID Collection):
# https://www.hvsc.c64.org/
```

## Architecture Overview

The codebase uses a single-file implementation architecture where all components are contained in:
- `Source/reSIDuEngine.h` - Complete SID emulator class interface
- `Source/reSIDuEngine.cpp` - Complete implementation

The implementation is monolithic and self-contained. See SCAFFOLD.md for detailed architectural documentation.

### Core Components

The `SID` class contains:

1. **Three Voice Channels**: Each voice has independent oscillators with waveform generation (triangle, sawtooth, pulse, noise), ADSR envelope generators, and support for sync and ring modulation between voices.

2. **Waveform Generation**:
   - 24-bit phase accumulators for frequency accuracy
   - Band-limited synthesis for pulse and sawtooth to reduce aliasing
   - 23-bit LFSR for noise generation
   - Combined waveform support (when multiple waveform bits are set)

3. **ADSR Envelopes**: Accurate envelope timing with exponential decay curves and proper rate counters.

4. **Multi-Mode Filter**: Bi-quadratic state-variable filter supporting lowpass, bandpass, and highpass modes with per-voice routing and resonance control. Separate implementations for 6581 and 8580 filter characteristics.

5. **Clock and Sample Rate**: The SID runs at PAL (985248 Hz) or NTSC (1022727 Hz) CPU clock frequency and generates output at the configured sample rate (typically 44100 Hz).

### Voice Interconnections

- Voice 1 can sync/ring-modulate with Voice 3
- Voice 2 can sync/ring-modulate with Voice 1
- Voice 3 can sync/ring-modulate with Voice 2

This circular dependency is important for understanding voice interactions.

## Register Map

The SID uses memory-mapped I/O with registers at addresses 0xD400-0xD418:

**Per-Voice Registers** (7 bytes × 3 voices):
- `+0x00/0x07/0x0E`: FREQ_LO (frequency low byte)
- `+0x01/0x08/0x0F`: FREQ_HI (frequency high byte)
- `+0x02/0x09/0x10`: PW_LO (pulse width low byte)
- `+0x03/0x0A/0x11`: PW_HI (pulse width high 4 bits)
- `+0x04/0x0B/0x12`: CONTROL (waveform, gate, sync, ring, test)
- `+0x05/0x0C/0x13`: ATTACK_DECAY
- `+0x06/0x0D/0x14`: SUSTAIN_RELEASE

**Global Registers**:
- `0xD415`: FC_LO (filter cutoff low 3 bits)
- `0xD416`: FC_HI (filter cutoff high byte)
- `0xD417`: RES_FILT (resonance + voice filter routing)
- `0xD418`: MODE_VOL (filter modes + master volume)
- `0xD41B`: OSC3 (read-only: voice 3 oscillator output)
- `0xD41C`: ENV3 (read-only: voice 3 envelope output)

## Frequency Calculation

Frequency register value is calculated as:
```
freq_register = (note_frequency_Hz * 16777216) / clock_frequency_Hz
```

Where:
- PAL clock:  985248 Hz (`C64_PAL_CPUCLK` constant)
- NTSC clock: 1022727 Hz (`C64_NTSC_CPUCLK` constant)
- 16777216 = 2^24 (phase accumulator size)

## Key Implementation Details

### Combined Waveforms

When multiple waveform bits are enabled simultaneously, the SID produces combined waveforms through analog interaction of the chip's internal circuitry. The implementation models this using lookup tables that simulate how neighboring bits affect each other through open-drain drivers and FET switches.

### Anti-Aliasing

The implementation uses frequency-dependent techniques:
- **Pulse waves**: Edge transitions are elongated at high frequencies (trapezoidal)
- **Sawtooth waves**: Become asymmetric triangles at high frequencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gavindi/reSIDuEngine](https://github.com/gavindi/reSIDuEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
