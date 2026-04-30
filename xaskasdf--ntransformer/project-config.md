---
trigger: always_on
description: High-efficiency C++/CUDA LLM inference engine. Goal: run Llama 70B at Q8-equivalent quality on a single RTX 3090 (24GB VRAM) by combining 6 memory-optimization techniques.
---

# NTransformer - Project Guide

## What Is This
High-efficiency C++/CUDA LLM inference engine. Goal: run Llama 70B at Q8-equivalent quality on a single RTX 3090 (24GB VRAM) by combining 6 memory-optimization techniques.

## Current State
**Phase 2 (SLEP) - COMPLETE. 3-tier adaptive caching — 33x speedup on 70B.**
- Phase 1 fully working: Llama 3.1 8B Q8_0 at 48.9 tok/s decode (resident), 0.9 tok/s (streaming)
- Phase 2 SLEP streaming: pipelined layer streaming via PCIe with worker thread
- **3-tier adaptive caching**: VRAM resident + pinned RAM + NVMe/mmap (auto-sized from hardware)
- 70B Q6_K: 29 VRAM + 51 RAM → **0.2 tok/s** (33x over mmap baseline), 23 GB VRAM
- 8B Q8_0: auto-promotes all 32 layers to VRAM → 48.8 tok/s (equivalent to resident)
- Q6_K quantization support: 70B Llama running on single RTX 3090
- `--streaming` CLI flag enables tiered mode (auto-selects best tier per layer)
- All unit tests passing (7/7 tensor, 6/6 kernel)
- **gpu-nvme-direct integrated and verified** — NVMe reads at 3,315 MB/s sustained (95% PCIe 4.0 x4)
- Worker thread pipeline: fallback path when NVMe unavailable or mmap pinning succeeds
- **Ported from Windows/MSVC/CUDA 12.4 to Linux/gcc-14/CUDA 13.1 (C++20 unified)**
- Setup/restore scripts: `scripts/setup_nvme.sh`, `scripts/restore_nvme.sh`

## Development Setup
- **Platform:** Linux (Ubuntu, kernel 6.17+)
- **Compiler:** gcc-14 / g++-14 (gcc-15 is incompatible with CUDA 13.1)
- **CUDA:** Toolkit 13.1, C++20 for both host and device code
- **GPU:** RTX 3090 24GB, Compute 8.6
- **Build requirements:** CMake 3.24+, CUDA Toolkit 13.1, C++20, gcc-14
- **No external dependencies** beyond CUDA Toolkit (no PyTorch, no cuBLAS)
- **Test models:** Configure paths via `-m` flag

### Build Commands
```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_C_COMPILER=gcc-14 \
  -DCMAKE_CXX_COMPILER=g++-14 \
  -DCMAKE_CUDA_COMPILER=/usr/local/cuda-13.1/bin/nvcc
cmake --build . --config Release -j
# Tests
./Release/test_tensor
./Release/test_gemm
# Run
./Release/ntransformer -m /path/to/model.gguf -p "Hello" -n 128
./Release/ntransformer -m /path/to/model.gguf --chat
./Release/ntransformer -m /path/to/model.gguf --benchmark -n 64
# Phase 2: Streaming mode (streams layers from CPU via PCIe)
./Release/ntransformer -m /path/to/model.gguf -p "Hello" -n 128 --streaming
```

### Build with gpu-nvme-direct (NVMe backend for streaming)
```bash
cmake .. -DCMAKE_BUILD_TYPE=Release -DUSE_GPUNVME=ON \
  -DCMAKE_C_COMPILER=gcc-14 \
  -DCMAKE_CXX_COMPILER=g++-14 \
  -DCMAKE_CUDA_COMPILER=/usr/local/cuda-13.1/bin/nvcc
cmake --build . --config Release -j

# Setup NVMe for VFIO (once after reboot)
sudo ./scripts/setup_nvme.sh 0000:01:00.0

# Write GGUF to NVMe raw device (once per model)
sudo ./scripts/restore_nvme.sh 0000:01:00.0   # bind to kernel driver
sudo dd if=/path/to/model.gguf of=/dev/nvme0n1 bs=1M oflag=direct status=progress
sudo ./scripts/setup_nvme.sh 0000:01:00.0      # rebind to VFIO

# Run with NVMe backend
sudo GPUNVME_PCI_BDF=0000:01:00.0 GPUNVME_GGUF_LBA=0 \
  ./build/ntransformer -m /path/to/model.gguf -p "Hello" -n 128 --streaming

# Restore NVMe to kernel driver when done
sudo ./scripts/restore_nvme.sh 0000:01:00.0
```

## Architecture Overview

### Namespace & Conventions
- Everything in `namespace nt`, CUDA kernels in `namespace nt::cuda`
- CUDA launchers: `launch_*()` (e.g., `launch_rmsnorm`, `launch_gemv`)
- C-linkage device ops: `nt_cuda_*()` (for .cpp files compiled without nvcc)
- Error macros: `NT_CHECK(cond, msg)`, `NT_CUDA_CHECK(err)` — both call `abort()` on failure

### Directory Layout
```
src/
├── core/           # Fundamental: types, tensor, allocator, GPU device
├── cuda/           # CUDA kernels (.cu files) + kernels.h declarations
├── memory/         # Phase 2: SLEP layer streaming (streamer.h/cu)
├── model/          # Transformer components: config, loader, norm, attention, ffn, transformer
├── inference/      # High-level: tokenizer, sampler, engine
├── utils/          # Timer, logger, profiler
├── main.cpp        # CLI entry point
include/            # Public C API (ntransformer.h)
tests/              # Unit tests
scripts/            # Python utilities (Phase 3+)
```

### Data Flow (Forward Pass)
```
tokens[seq_len] (CPU)
  → embed_tokens(): CPU lookup + H2D copy → hidden_state[seq_len, hidden_size] (GPU)
  → for each layer:
      norm → attention(+RoPE, KV cache, GQA) → residual add
      norm → FFN(SwiGLU: gate↦SiLU, up, down) → residual add
  → output_norm (last token only)
  → LM head GEMV → logits[vocab_size] (GPU)
  → D2H copy → sampling (CPU) → next token
```

### Streaming Pipeline (Phase 2)
```
Worker thread:  [memcpy L0→stg0][memcpy L1→stg1][memcpy L2→stg0]...
H2D DMA:        [              ][stg0→gpu0     ][stg1→gpu1     ]...
GPU Compute:    [              ][              ][layer 0       ]...
```
Three-stage pipeline: worker memcpy (CPU cores), async H2D (PCIe DMA), GPU compute (SMs).

**With gpu-nvme-direct (eliminates CPU from data path):**
```
GPU MMIO:       [doorbell L0  ][doorbell L1  ][doorbell L2  ]...
NVMe DMA:       [DMA L0→stg0  ][DMA L1→stg1  ][DMA L2→stg0 ]...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xaskasdf/ntransformer](https://github.com/xaskasdf/ntransformer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
