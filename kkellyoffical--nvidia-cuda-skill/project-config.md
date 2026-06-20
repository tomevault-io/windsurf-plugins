---
trigger: always_on
description: Use when work targets NVIDIA GPUs for deep learning training, inference, distributed execution, CUDA/Triton kernels, or AI infra tuning. Enforces GPU-aware code conventions for PyTorch on CUDA, including dtype policy, memory movement, NCCL/DDP/FSDP choices, profiling, benchmarking, and H100/H200/B200 optimization habits.
---


# NVIDIA CUDA

Use this skill when the task involves NVIDIA GPU training or inference, multi-GPU execution, CUDA/Triton kernels, or CUDA-specific performance/debugging work.

This skill is for implementation and review, not generic theory. It should push the work toward:

- correct GPU usage first
- stable and measurable performance second
- low-risk tuning before low-level kernel work
- frontier algorithms only when the workload shape justifies them

## Hardware stance

Probe the actual machine before acting. This skill is written for modern NVIDIA Tensor Core systems and is especially opinionated for:

- H100 / Hopper
- H200
- B200 / Blackwell-class accelerators

It assumes the agent should actively choose precision, attention backends, sharding strategy, logging frequency, and dataloader settings instead of inheriting slow defaults.

## Use this skill for

- PyTorch training or inference on CUDA
- GPU memory, throughput, or latency bottlenecks
- DDP / FSDP / NCCL decisions
- CUDA env var and runtime hygiene
- Triton or custom CUDA kernel review
- benchmark and profiler setup for NVIDIA GPUs

## Do not use this skill for

- CPU-only optimization
- ROCm / AMD-specific work
- TPU / XLA-specific work
- vague "make it faster" requests without inspecting the actual GPU path first

## First-pass workflow

Before changing code, inspect the actual runtime surface:

1. Check hardware and driver: `nvidia-smi`
2. Check framework build: `python -c "import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available())"`
3. Check device visibility and topology:
   - `CUDA_VISIBLE_DEVICES`
   - `torch.cuda.device_count()`
   - `torch.cuda.get_device_properties(i)`
4. Check whether the bottleneck is:
   - input pipeline
   - host/device transfer
   - eager Python overhead
   - kernel efficiency
   - distributed communication
   - memory pressure / fragmentation / OOM

Do not jump to kernel rewrites before ruling out bad data movement, wrong dtype, graph breaks, or poor distributed setup.

## Bundled tooling

Use the included scripts when you need deterministic probes instead of ad hoc snippets:

- `scripts/cuda_env_probe.py`: collect CUDA, PyTorch, device, and env facts
- `scripts/check_training_stack.py`: scan Python code for high-cost anti-patterns
- `scripts/benchmark_attention.py`: benchmark SDPA, flash, cuDNN, and official flash-implementation activation paths
- `scripts/training_step_benchmark.py`: benchmark a synthetic transformer training step with dtype, compile, and `.item()` logging knobs
- `scripts/dataloader_benchmark.py`: benchmark DataLoader worker, pinning, and prefetch settings
- `scripts/nccl_smoke.py`: run a minimal NCCL all-reduce smoke test under `torchrun`
- `scripts/ddp_fsdp_smoke.py`: run a one-step DDP or FSDP training smoke test under `torchrun`

Run the probe first, then benchmark or scan the real workload path.

## Planning hardware purchases

When the user wants current NVIDIA GPU recommendations, read:

- [references/latest-gpu-recommendations-2026-04.md](references/latest-gpu-recommendations-2026-04.md)
- the example configs under [examples/](examples)

Keep recommendation output scenario-based:

- cost-sensitive local prototyping
- serious workstation development
- enterprise server deployment
- single-node training
- rack-scale training or reasoning

Do not recommend by peak FLOPS alone. Weight memory, interconnect, thermals, deployment form, and software maturity.

## Reviewing Triton, CUDA, and distributed code

When the target includes Triton kernels, CUDA or C++ files, or distributed launcher code:

- use `scripts/check_training_stack.py` across the whole tree, not just Python subfolders
- treat Triton kernels as first-class review targets
- verify distributed paths with `scripts/nccl_smoke.py` or `scripts/ddp_fsdp_smoke.py` before claiming they are healthy
- benchmark flash attention implementation changes with `scripts/benchmark_attention.py --list-flash-impls` and explicit activation when available

## Non-negotiable code conventions

### High-cost anti-patterns to ban

Treat the following as red flags in reviews and optimization work:

1. keeping transformer training or inference in full FP32 by default on Tensor Core GPUs
2. refusing to try `torch.compile` on stable hot paths
3. staying on a generic default SDPA path on H200/B200 when newer flash-backed implementations or cuDNN/TE attention backends are available and benchmarkable
4. reaching for activation checkpointing before FSDP / ZeRO-style sharding when the real problem is replicated params, grads, or optimizer state
5. calling `.item()` every step for logging and metrics
6. leaving DataLoader workers, pinning, prefetching, and persistent workers untuned until GPU starvation shows up

If several of these exist at once, assume the GPU is underfed until proven otherwise.

### 1. Dtype policy

- On H100 / Hopper, prefer `bfloat16` for training and inference unless there is a measured or correctness-driven reason not to.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kkellyoffical/nvidia-cuda-skill](https://github.com/kkellyoffical/nvidia-cuda-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
