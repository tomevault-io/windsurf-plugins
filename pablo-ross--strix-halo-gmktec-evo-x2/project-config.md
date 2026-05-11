---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains setup guides and configuration documentation for optimizing Ubuntu 24.04 on the GMKTEC EVO-X2 (AMD Ryzen AI Max+ 395 with Radeon 8060S) for LLM inference using llama.cpp with ROCm 7 RC and rocWMMA.

## Hardware Context

**Target System:**
- CPU: AMD RYZEN AI MAX+ 395 w/ Radeon 8060S (32 threads)
- GPU: AMD Radeon Graphics (gfx1151, RDNA 3.5, 40 CUs, Strix Halo)
- RAM: 124 GiB system memory
- GPU Memory: 1 GB VRAM (framebuffer) + 128 GB GTT (unified compute memory)

**Critical Understanding:** For this APU, GTT (Graphics Translation Table) is the primary compute memory pool for LLM inference, not VRAM. The system has ~120GB available for inference workloads.

## Key Setup Steps Reference

### System Configuration (from ROADMAP.md)

**Kernel Requirements:**
- Minimum: Linux 6.16.9 (critical for >15GB VRAM access)
- Current: 6.16.9-061609-generic

**Essential Kernel Parameters:**
```
amd_iommu=off amdgpu.gttsize=131072 ttm.pages_limit=31457280
```

**Critical Modprobe Configuration** (`/etc/modprobe.d/amdgpu_llm_optimized.conf`):
```bash
options amdgpu gttsize=122800
options ttm pages_limit=31457280
options ttm page_pool_size=31457280
```

**GPU Access (Ubuntu-specific):** Must create udev rules in `/etc/udev/rules.d/99-amd-kfd.rules`:
```bash
SUBSYSTEM=="kfd", GROUP="render", MODE="0666", OPTIONS+="last_rule"
SUBSYSTEM=="drm", KERNEL=="card[0-9]*", GROUP="render", MODE="0666", OPTIONS+="last_rule"
SUBSYSTEM=="drm", KERNEL=="renderD[0-9]*", GROUP="render", MODE="0666", OPTIONS+="last_rule"
```
The `renderD[0-9]*` rule is critical - without it, ROCm will fail with `HSA_STATUS_ERROR_OUT_OF_RESOURCES`.

### Container Environment

**Tool:** Distrobox (not toolbox - Ubuntu 24.04 doesn't include toolbox)
- Container: `docker.io/kyuz0/amd-strix-halo-toolboxes:rocm-7rc-rocwmma`
- Base OS: Fedora 44 (Rawhide)
- Package manager in container: `dnf` (not `apt`)

**Create container:**
```bash
distrobox create llama-rocm-7rc-rocwmma \
  --image docker.io/kyuz0/amd-strix-halo-toolboxes:rocm-7rc-rocwmma \
  --additional-flags "--device /dev/dri --device /dev/kfd --group-add video --group-add render --group-add sudo --security-opt seccomp=unconfined"
```

**Enter container:**
```bash
distrobox enter llama-rocm-7rc-rocwmma
```

### Building llama.cpp

**Location:** Inside the ROCm container at `~/llama.cpp`

**Build dependencies (install in container first):**
```bash
sudo dnf install -y cmake gcc-c++ git libcurl-devel python3-pip
```

**Build command:**
```bash
cmake -B build -S . \
  -DGGML_HIP=ON \
  -DAMDGPU_TARGETS="gfx1151" \
  -DGGML_HIP_ROCWMMA_FATTN=ON \
  -DGGML_HIP_MMQ_MFMA=ON

cmake --build build --config Release -j$(nproc)
```

**Binaries location:** `build/bin/`

### Running Inference

**Critical flags for this hardware:**
- `--no-mmap` (llama-cli) or `-mmp 0` (llama-bench): Required for GPU backends
- `-ngl 99` (or 999): Offload all layers to GPU

**CLI inference:**
```bash
./build/bin/llama-cli \
  -m ~/models/model.gguf \
  --no-mmap \
  -ngl 99 \
  -p "prompt" \
  -n 128
```

**Benchmark:**
```bash
./build/bin/llama-bench \
  -m ~/models/model.gguf \
  -mmp 0 \
  -ngl 99 \
  -p 512 \
  -n 128
```

**Server mode:**
```bash
./build/bin/llama-server \
  -m ~/models/model.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  -ngl 99 \
  --no-mmap \
  -c 4096
```

### Model Management

**Storage location:** `~/models` (outside containers to persist across updates)

**Downloading models:**
```bash
# Install HuggingFace CLI
pip install "huggingface-hub[cli]" hf-transfer

# Download (note: command is 'hf download' not 'huggingface-cli download')
export HF_HUB_ENABLE_HF_TRANSFER=1
hf download TheBloke/Llama-2-7B-GGUF llama-2-7b.Q4_K_M.gguf --local-dir ~/models
```

## Performance Characteristics

**Benchmark Results (from INITIAL_BENCHMARK.md):**
- Llama-2-7B Q4_K_M performance:
  - Prompt processing (512 tokens): 871.77 t/s
  - Text generation: 43.83 t/s
  - Optimal batch size: 512 tokens

**Memory Usage:**
- 7B Q4_K_M model: ~3.8 GB
- Context memory (4K): ~2 GB
- Total available: ~120 GB (can fit 70B+ models)

**Performance tuning:**
- Batch size: 512 (optimal for this hardware)
- Use `tuned-adm profile accelerator-performance`
- Verify GPU power state: check `/sys/class/drm/card1/device/power_dpm_force_performance_level`

## Common Issues and Solutions

### Ubuntu-Specific Issues

**`toolbox` not found:**
- Ubuntu 24.04 doesn't include `toolbox` in repos
- Use `distrobox` instead
- Replace all `toolbox` commands with `distrobox` commands

**`HSA_STATUS_ERROR_OUT_OF_RESOURCES` with rocminfo:**
- Missing renderD udev rule
- Fix: Add `renderD[0-9]*` rule to `/etc/udev/rules.d/99-amd-kfd.rules`
- Reload: `sudo udevadm control --reload-rules && sudo udevadm trigger`

**Container can't access GPU:**
- Check permissions: `ls -la /dev/kfd /dev/dri/`
- All should be `0666` (crw-rw-rw-)
- Verify user in groups: `video` and `render`

### Build Issues in Container

**`cmake: command not found`:**
- Container doesn't include build tools by default
- Install: `sudo dnf install -y cmake gcc-c++ git libcurl-devel`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pablo-ross/strix-halo-gmktec-evo-x2](https://github.com/pablo-ross/strix-halo-gmktec-evo-x2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
