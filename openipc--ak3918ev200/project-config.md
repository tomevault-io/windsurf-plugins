---
trigger: always_on
description: **Project:** OpenIPC AK3918EV200 (Codex)
---

# AGENTS.md

**Project:** OpenIPC AK3918EV200 (Codex)  
**Goal:** Make OpenIPC-style userspace work reliably on Anyka **AK3918EV200** SoCs: bring up the ISP pipeline, drain the H.264 encoder, and persist streams to SD, with optional audio.  
**Author:** Abel Romero  
**License:** MIT

---

## 1) Overview

This document defines the **agents** (processes/threads/modules) and their responsibilities, inputs/outputs, and interfaces to the device nodes and SDK wrappers we use. It is written for contributors joining the Codex effort so they can understand *who does what* in our runtime.

### Key device nodes

- `/dev/video0` — V4L2 capture (must be opened to make the ISP happy).
- `/dev/isp_char` — ISP control plane (AK_ISP_* ioctls).
- `/dev/uio0` — H.264 encoder (IRQ + MMIO window; ring buffer of encoded frames).
- `/dev/ion` — DMA/DMABUF allocation (shared buffers).
- `/dev/akpcm_cdev0` — speaker (PCM out).
- `/dev/akpcm_cdev1` — microphone (PCM in).

### Filesystem layout (target camera)

- Persistent data: `/mnt/disc1`  
- Binaries, libs, configs: `/mnt/bin`  
- Use `/tmp` **only** for scratch/ephemeral data.

---

## 2) Agent map (who does what)

> Each agent is a self-contained unit. Some are threads inside the same process; others can be stand-alone binaries. The names below map to current source files where applicable.

### A. **ISP Config Agent**  
**Purpose:** Parse sensor `.conf` files and apply all ISP blocks in the correct order.

- **Inputs:** Sensor `.conf` blob (all vendors), pack & subpack structure.  
- **Outputs:** Series of `AK_ISP_*` ioctls to `/dev/isp_char`.  
- **Key rules:**
  - Subpack header is `{ uint16_t id; uint16_t size; }`.
  - The last subpack is **sensor load**; for `Ak_ISP_Sensor_Load_Conf` the SDK expects **size in 32-bit words of (payload + 4)** and a pointer to **payload** (without the subpack header).
  - Most other `AK_ISP_*` setters take **payload only** (no header).
- **Code:**  
  - `src/conf/parse_isp_conf.c` — walks packs/subpacks and invokes a callback.  
  - `src/isp/apply_isp_conf.c` — switch over subpack `id` → calls `ak_isp_*` wrappers.  
  - `include/akispsdk.h` — externs for all `AK_ISP_*`/`Ak_ISP_*`.

### B. **Device Bring-Up Agent**  
**Purpose:** Open devices in the right order and keep them alive.

- **Order (current best-known):**  
  1. Open `/dev/video0` (enables ISP data path).  
  2. Open `/dev/isp_char` (control).  
  3. Open `/dev/uio0` (encoder interrupts/MMIO).  
  4. Open `/dev/ion` (buffers).  
- **Also:** Set non-blocking where appropriate, install signal pipe for clean shutdowns.

### C. **ION/DMABUF Agent**  
**Purpose:** Allocate and track physically contiguous buffers for video pipeline and encoder.

- **Inputs:** Requested sizes from pipeline; alignment constraints.
- **Outputs:** DMABUF fds, exported to V4L2 or encoder as needed.
- **Notes:** Keep allocations small; devices are memory-constrained. Persist allocation metadata to `/mnt/disc1` if you need to re-use across restarts.

### D. **Encoder (UIO) Agent**  
**Purpose:** Drain encoded H.264 frames from the hardware ring through `/dev/uio0`.

- **UIO workflow (generic):**  
  1. `mmap()` the UIO region(s) to read encoder registers/ring descriptors.  
  2. `read(uio_fd, &irq_cnt, 4)` blocks until an IRQ (frame ready).  
  3. Parse head/tail and frame length fields; copy the encoded frame out.  
  4. `write(uio_fd, &ack, 4)` to re-enable IRQs (standard UIO pattern).  
- **SDK helpers:** `ak_uio_wait_irq()` now understands the vendor `uio_video_codec`
  ioctls (`AK_UIO_SYSREG_WRITE`, cache invalidation, alt IRQ waits). See
  `include/ak3918/video.h`, `src/lib/video/uio.c`, and `docs/sdk/ak3918_sdk_notes.md`
  before touching encoder registers.  
- **Outputs:** H.264 access units written to `/mnt/disc1/stream/` or a FIFO/UDP.  
- **Open items:** Exact offsets for head/tail/len registers are hardware-specific; we correlate *uio0 dump* and vendor *ipc* traces to finalize these (see §7).

### E. **V4L2 Agent**  
**Purpose:** Maintain the capture pipeline running (enqueue/dequeue loop) so the ISP/encoder keep producing.

- **Notes:** Even if we don’t save RAW, keep the QBUF/DQBUF loop alive at a modest depth. The vendor pipeline does this; starving queues can stall the encoder.

### F. **Audio Agents (optional)**  
**Mic Agent** (`/dev/akpcm_cdev1`) and **Speaker Agent** (`/dev/akpcm_cdev0`)

- **Purpose:** Provide synchronized PCM capture/playback when needed.  
- **Notes:** Both devices require an `ioctl()` configuration step **before** streaming. For development builds where we don’t want audio I/O, **comment out** the read/write loops after setup (we keep setup to validate the path). Keep the private ioctls documented in `docs/sdk/ak3918_sdk_notes.md#audio-codec-devakpcm_cdev0-devakpcm_cdev1` close by—codec mode/source clocks must be staged exactly in that order.

### G. **Trace/Hook Agent** (LD_PRELOAD)  
**Purpose:** Interpose `open/ioctl/mmap/read/write/...` to learn the vendor’s parameters and to dump structured payloads on selected ioctls.

- **Artifacts:** `ioctl_dump_map.txt` supports `(req, direction, dumps, struct_mode)`; for `0x40044969` (sensor load) we enable *struct mode* to follow the detected `struct blob_desc { uint32_t nwords; void* ptr; }`.

### H. **Supervisor Agent**  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenIPC/ak3918ev200](https://github.com/OpenIPC/ak3918ev200) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
