---
trigger: always_on
description: TeleFuser is a high-performance framework for efficient multimodal generation model inference (image/video generation, video super-resolution).
---

# TeleFuser - Agent Guidelines

## Project Overview

TeleFuser is a high-performance framework for efficient multimodal generation model inference (image/video generation, video super-resolution). 

**Tech Stack:** Python 3.10-3.13, PyTorch 2.6+, CUDA 12.8+, FastAPI, Ray

**Supported Models:** WanVideo (Wan2.1/2.2), Qwen-Image, Z-Image, FlashVSR, HunyuanVideo, Flux2 Klein, LTX Video, LiveAct, LongCat-Video

## Commands

```bash
pip install -e ".[dev]"           # Development installation
pre-commit run --all-files        # Linting checks
pytest tests/                     # Run tests
bash scripts/run_ci_tests.sh      # Full CI suite
telefuser serve /path/to/pipeline.py --port 8000  # Start API server
```

## Troubleshooting

When multi-GPU inference hangs, zombie processes may remain. Clean them up with:

```bash
ps aux | grep -E 'spawn_main' | grep -v grep | awk '{print $2}' | xargs kill -9
```

## Architecture

```
telefuser/
├── core/             # Base abstractions: BasePipeline, BaseStage, configs
├── pipelines/        # Model-specific pipelines
│   ├── wan_video/    # Wan2.1/2.2: T2V, I2V, FL2V
│   ├── qwen_image/   # Qwen-Image: T2I, Edit
│   ├── z_image/      # Z-Image: T2I
│   ├── flashvsr/     # FlashVSR: VSR
│   ├── hunyuan_video_1_5/  # HunyuanVideo: T2V, I2V
│   ├── flux2_klein/  # Flux2 Klein: T2I
│   ├── ltx_video/    # LTX Video: I2V + Audio
│   ├── liveact/      # LiveAct: S2V (speech-to-video)
│   ├── longcat_video/ # LongCat-Video: T2V, I2V
│   └── common/       # Shared pipeline utilities
├── models/           # Model architectures: DiT, VAE, text encoders
├── ops/              # Custom operations: attention, FFN, normalization
├── kernel/           # Triton kernels: RMSNorm, rotary, quant, fused ops
│   └── triton/       # Pure Triton implementations
├── platforms/        # Hardware abstraction: CUDA, NPU, CPU
├── distributed/      # FSDP, TP, PP, SP, Ring/Ulysses attention
│   ├── ulysses_comm.py   # Ulysses All-to-All: ulysses_scatter_heads, ulysses_gather_heads
│   ├── ring.py            # Ring P2P communication for long sequences
│   ├── pp_comm.py         # Pipeline parallelism
│   ├── fsdp.py            # Fully Sharded Data Parallel
│   ├── tp_parallelize.py  # Tensor Parallelism
│   └── parallel_shard.py  # Parallel sharding utilities
├── schedulers/       # Diffusion schedulers
├── feature_cache/    # Feature caching: AdaTaylorCache
├── cache/            # General cache management
├── offload/          # CPU offload strategies
├── metrics/          # Metrics collection and monitoring
├── orchestrator/     # Pipeline orchestration
├── worker/           # Distributed worker management
├── entrypoints/      # CLI entry points
├── service/          # FastAPI service
└── client/           # Python SDK
```

### Layer Architecture Principles For Models

TeleFuser's model follows a strict layered architecture for operations:

```
┌─────────────────────────────────────────────────────────────┐
│                      models/                                 │
│  (DiT, VAE, text encoders - ONLY import from ops/)          │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                       ops/                                   │
│  (Compile-aware dispatch: native for compile, kernel for    │
│   eager mode. Base classes: CustomOp, CustomOpFunction)     │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   kernel/triton/                             │
│  (Pure Triton kernels, custom ops. NOT directly used by     │
│   models. May have torch.library.custom_op registration.)   │
└─────────────────────────────────────────────────────────────┘
```

**Key Rules:**

1. **models/** layer MUST only import from `telefuser.ops/`
   - ✅ `from telefuser.ops.normalization import RMSNorm, LayerNorm, modulate`
   - ✅ `from telefuser.ops.rotary import apply_rotary_emb`
   - ❌ `from telefuser.kernel.triton import apply_rotary_embedding`

2. **ops/** layer handles compile-aware dispatch:
   - `torch.compiler.is_compiling()` → PyTorch native implementation
   - Eager mode + CUDA → Optimized Triton kernel
   - Other platforms → PyTorch native fallback

3. **kernel/triton/** contains pure Triton code:
   - No `torch.compiler.is_compiling()` checks
   - May use `torch.library.custom_op` for torch.compile compatibility
   - Only used by ops/ layer, never directly by models/

## Code Style

- PEP8 with ruff (line length: 120)
- Comments and docstrings **must be in English**
- All public function parameters **must have type annotations** (return types optional)
- Use Python 3.10+ syntax: `str | None`, `list[int]`

## Documentation Links

| Topic | English | Chinese |
|-------|---------|---------|
| Adding New Example | [docs/en/adding_new_example.md](docs/en/adding_new_example.md) | [docs/zh/adding_new_example.md](docs/zh/adding_new_example.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tele-AI/TeleFuser](https://github.com/Tele-AI/TeleFuser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
