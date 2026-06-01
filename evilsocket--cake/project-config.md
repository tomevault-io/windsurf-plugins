---
trigger: always_on
description: | Machine | Role | GPU | VRAM | OS | Work Dir | SSH |
---

# Cake Development Guide

## Cluster Machines

| Machine | Role | GPU | VRAM | OS | Work Dir | SSH |
|---------|------|-----|------|----|----------|-----|
| **blade.local** | Master (local) | RTX 3080 Laptop | 16 GB | Linux | `/home/evilsocket/Lab/cake` | N/A |
| **bahamut.local** | Worker | 2× TITAN X Pascal | 2×12 GB | Linux | `~/Lab/cake` | `ssh bahamut.local` |
| **stevie.local** | Worker | Apple M3 Pro | 36 GB unified | macOS | `~/Lab/cake` | `ssh stevie.local` |

## Build Commands

```bash
# blade.local (local, CUDA)
cargo build --release --features cuda

# bahamut.local (CUDA — MUST use cuda-12.4, driver only supports up to 12.4)
CUDA_HOME=/usr/local/cuda-12.4 LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64 cargo build --release --features cuda

# stevie.local (Metal)
cargo build --release --features metal

# Vulkan (Steam Deck, AMD GPUs, any Vulkan 1.3+ device)
cargo build --release --features vulkan
```

## Acceleration Features

| Feature | Platform | Backend | Best For | Notes |
|---------|----------|---------|----------|-------|
| `metal` | macOS (Apple Silicon) | GPU via MPS + custom MSL kernels | Primary inference on Mac | Fastest option on Apple Silicon (~42 tok/s on M3 Pro) |
| `cuda` | Linux (NVIDIA GPU) | GPU via cuBLAS/cuDNN | Primary inference on Linux | Requires CUDA toolkit matching driver version |
| `accelerate` | macOS | CPU via Apple Accelerate (AMX) | CPU-only F32 inference on Mac | 2.7x faster than pure-Rust for F32 matmul; no F16 support |
| `vulkan` | Any (Vulkan 1.3+) | GPU via Vulkan compute shaders | Steam Deck, AMD GPUs | Portable but less optimized than Metal/CUDA |
| (none) | Any | CPU via pure-Rust `gemm` | Portable CPU fallback | F16 weights stay F16, avoids bandwidth doubling |

**When to use which:**
- **Apple Silicon (stevie.local):** Use `--features metal`. Metal is 1.6x faster than CPU F16 (42 vs 26 tok/s). The `accelerate` feature doesn't help with Metal and doesn't support F16 matmul, so CPU F16 (default, no features) is actually faster than `accelerate` with F32 (26 vs 23 tok/s).
- **NVIDIA GPU (blade/bahamut):** Use `--features cuda`. Add `flash-attn` for flash attention support.
- **CPU-only with F32 models:** Use `--features accelerate` on macOS for 2.7x faster F32 matmul. On Linux, consider linking against MKL or OpenBLAS.
- **CPU-only with F16 models:** Use no features — pure-Rust `gemm` with F16 avoids the 2x memory bandwidth penalty of converting to F32.

## Interactive Chat

```bash
# Local mode (loads model, starts TUI chat — no server needed)
cake chat evilsocket/Qwen3-0.6B

# Remote mode (connects to a running API server)
cake chat --server http://localhost:8080
```

The TUI streams tokens in real time and detects `<think>` tags — showing a "thinking..." indicator with reasoning in gray, then the final response in white.

## Model Management

```bash
cake pull evilsocket/Qwen3-0.6B       # download from HuggingFace
cake list                              # show all cached models
cake rm evilsocket/Qwen3-0.6B         # delete a cached model (with confirmation)
cake rm Qwen3-0.6B                    # short name also works (suffix match)
```

`cake rm` shows the model name, path, and size before asking for confirmation. For HF cache models it removes the entire `models--org--name/` directory. Refuses to delete local (non-cache) models.

## Run Commands (Qwen3.5-0.8B cluster)

```bash
# Workers first (on each machine):
# bahamut:
LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64 ./target/release/cake run \
  evilsocket/Qwen3.5-0.8B --name bahamut \
  --topology topology-0.8B.yml --address 0.0.0.0:10128

# stevie:
./target/release/cake run \
  evilsocket/Qwen3.5-0.8B --name stevie \
  --topology topology-0.8B.yml --address 0.0.0.0:10128

# Master (blade, local):
./target/release/cake run evilsocket/Qwen3.5-0.8B \
  "Explain quantum computing in simple terms" \
  --topology topology-0.8B.yml
```

## Model: evilsocket/Qwen3.5-0.8B

- **Architecture**: Qwen3_5ForConditionalGeneration
- **Layers**: 24 (18 GatedDeltaNet linear attn + 6 full attn)
- **Hidden size**: 1024
- **Layer prefix**: `model.language_model.layers.{N}`
- **Location**: HuggingFace cache on all 3 machines (`~/.cache/huggingface/hub/models--Qwen--Qwen3.5-0.8B/`)
- **Size**: ~1.6 GB in F16

## Topology: `topology-0.8B.yml`

24 layers split evenly: bahamut 0-7, stevie 8-15, blade master keeps 16-23.

## Self Improving Loop

This is an iterative optimization process for maximizing inference speed:

### Process

1. **Instrument**: Add timing/profiling logs to hot code paths (forward pass, attention, MLP, network, serialization)
2. **Commit & push**: Commit changes, push to origin
3. **Deploy**: Pull on all 3 machines via SSH, rebuild with appropriate features (cuda/metal)
4. **Run experiment**: Start workers on bahamut and stevie, then master on blade with a test prompt
5. **Collect metrics**: Capture tok/s, per-layer timing, network latency from logs
6. **Analyze**: Identify the current bottleneck (slowest component)
7. **Optimize**: Make targeted code changes to address the bottleneck
8. **Verify**: Run tests (`cargo test --features cuda`), verify coherent output
9. **Compare**: Compare metrics with previous iteration
10. **Repeat**: Go to step 1 until no further gains are possible


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evilsocket/cake](https://github.com/evilsocket/cake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
