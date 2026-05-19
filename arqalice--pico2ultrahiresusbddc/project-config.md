---
trigger: always_on
description: **Pico2UltraHiResUSBDDC** is a **USB Audio Class 1.0** compliant USB-DDC firmware targeting **RP2350 (Raspberry Pi Pico 2)**.
---

# AGENTS.md

## 1. Project overview

**Pico2UltraHiResUSBDDC** is a **USB Audio Class 1.0** compliant USB-DDC firmware targeting **RP2350 (Raspberry Pi Pico 2)**.
The device receives stereo PCM over USB isochronous OUT, performs real-time upsampling, and outputs **32-bit I²S** to an external DAC.

Primary goals:
- Stable UAC1.0 enumeration and isochronous audio ingestion
- Low-latency, glitch-free audio pipeline under hard real-time constraints
- High-quality upsampling using FIR/IIR (CMSIS-DSP)
- Deterministic I²S output via PIO + DMA

Non-goals (for this repo):
- Host-side apps/drivers
- UAC2.0 / DSD / DoP (not implemented here)

## 2. Quick start for agents

### 2.1 Build (CLI)

This project uses Pico SDK + CMake. The repository includes `pico-extras/`, `CMSIS/`, and `lufa/` sources.

Linux/macOS example:

```bash
git clone <repo>
cd Pico2UltraHiResUSBDDC
mkdir -p build
cd build
cmake .. \
  -DPICO_BOARD=pico2 \
  -DPICO_SDK_FETCH_FROM_GIT=ON \
  -DPICO_SDK_FETCH_FROM_GIT_TAG=2.0.0
cmake --build . -j
```

Windows (PowerShell) example:

```powershell
mkdir build
cd build
cmake .. -DPICO_BOARD=pico2 -DPICO_SDK_FETCH_FROM_GIT=ON -DPICO_SDK_FETCH_FROM_GIT_TAG=2.0.0
cmake --build .
```

Build outputs are under `build/src/`:
- `Pico2UltraHiResUSBDDC.uf2` (flashable)
- `Pico2UltraHiResUSBDDC.elf` (debug)
- `Pico2UltraHiResUSBDDC.bin`

Notes:
- If you already have Pico SDK installed, prefer `-DPICO_SDK_PATH=<path>` instead of fetching from Git.
- The top-level `CMakeLists.txt` adds `-O3` and Cortex-M33 FP options. Avoid changing these unless you understand the timing impact.

### 2.2 Flash

1. Hold **BOOTSEL** on Pico 2 while connecting USB.
2. Copy `build/src/Pico2UltraHiResUSBDDC.uf2` to the mounted drive.
3. The device should enumerate as a standard USB audio device.

## 3. Repository layout

Top-level:
- `src/` : firmware sources (Core0/Core1, USB, upsampling, I2S/PIO/DMA)
- `CMSIS/` : CMSIS-Core and CMSIS-DSP (vendored)
- `pico-extras/` : Pico extras (vendored)
- `lufa/` : LUFA headers used for USB Audio class descriptor structures
- `make_digitalFilter/` : Python scripts to design filter coefficients

Important `src/` files:
- `main.c` : Core0 main; init clocks/IO, starts USB, launches Core1, timer ISR
- `usb_device_control.c/.h` : USB descriptors and UAC1 control/streaming endpoint handling
- `upsampling.c/.h` : real-time upsampling processing (Core0 + Core1 stages)
- `upsampling_coef.c` : filter coefficient tables (generated)
- `transmit_to_dac.c/.h` : PIO I2S init and DMA TX pipeline (Core1)
- `i2s.pio` + `i2s_pio_interface.c/.h` : PIO program and helpers
- `ringbuffer.c/.h` : lock-aware ring buffers between subsystems/cores
- `ess_specific.c/.h` + `nonblocking_i2c.c/.h` : optional ESS DAC register control over I2C
- `debug_with_gpio.c/.h` : logic-analyzer friendly debug output
- `common.h` : user configuration macros and global constants

## 4. Runtime architecture (high-level)

### 4.1 Dataflow

```
USB Iso OUT (PCM) -> endpoint handler -> ringbuffer (L/R)
  -> Core0 upsampling stage(s) -> ringbuffer (to Core1)
  -> Core1 upsampling stage -> interleaved int32 LR samples
  -> DMA queue -> PIO TX FIFO -> I²S pins -> external DAC
```

### 4.2 Core split

- Core0 (main CPU):
  - USB enumeration, UAC1 control requests, isochronous OUT packet parsing
  - Maintains input ring buffers (`buffer_ep_Lch`, `buffer_ep_Rch`)
  - Executes timer-triggered upsampling work and produces Core1 input buffers
  - Power-mode switching (Hi/Lo power), clock changes, volume/mute state updates

- Core1:
  - Initializes I²S (PIO) and DMA engine
  - Periodically calls `dma_tx_start()` which:
    - Pulls upsampled audio from Core0 ring buffers
    - Runs the final upsampling stage (`upsampling_process_core1`)
    - Packs into a DMA TX ring and keeps the PIO FIFO fed

### 4.3 Timing / real-time constraints

- The audio pipeline is hard real-time.
- Time-critical routines are marked `__not_in_flash_func(...)` and must remain fast.
- Avoid any of the following in time-critical paths (USB packet handler, timer callback, DMA IRQ):
  - Dynamic allocation (`malloc/free`)
  - Blocking I/O (UART printf flood, I2C blocking calls)
  - Long critical sections (interrupts disabled for long)
  - Excessive floating-point overhead without profiling

## 5. Configuration knobs (common.h)

Most user-facing configuration is done via macros in `src/common.h` under “User Configurable”.

Common edits:
- Pin assignment:
  - `I2S_DATA_PIN`, `I2S_SIDESET_BASE` (BCLK/LRCK are sideset)
  - I2C pins: `I2C_SDA`, `I2C_SCL`
  - Optional pins: `DAC_ENABLE_PIN`, `POWER_MODE_SWITCH_PIN`, `DCDC_MODE_PIN`
- Upsampling behavior:
  - `BYPASS_CORE1_UPSAMPLING`, `CORE0_UPSAMPLING_192K`, `ENABLE_1536KHZ_OUTPUT`
  - `DEFAULT_GAIN_RATIO` (important to avoid clipping)
- Power mode:
  - `ALWAYS_HIGH_POWER`, `ALWAYS_LOW_POWER`, `V_CORE_HI`, `V_CORE_LO`

When changing pinouts, ensure the PCB wiring matches and that no pin conflicts exist (PIO, I2C, LED, etc.).

## 6. USB audio implementation notes

- USB stack: Pico SDK `pico/usb_device`.
- USB Audio class descriptor types/structs: `lufa/AudioClassCommon.h`.
- Audio streaming endpoints are defined in `src/usb_device_control.c`.

Important invariants:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArqAlice/Pico2UltraHiResUSBDDC](https://github.com/ArqAlice/Pico2UltraHiResUSBDDC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
