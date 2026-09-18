---
trigger: always_on
description: CUDA inference and serving for NVFP4 and mixed-precision models on NVIDIA DGX
---

# Eider

CUDA inference and serving for NVFP4 and mixed-precision models on NVIDIA DGX
Spark / GB10 (`sm_121`). The workspace contains the `eider-api` server, the
multi-model `infer` runtime, and the `nvfp4` CUDA kernel crate. Supported model
families currently include Qwen3.5/3.6 MoE, dense Qwen3.8, Step-3.7,
Laguna-S-2.1, Gemma 4, Nemotron 3, and DeepSeek V4 Flash.
Muse Glimmer 30B is supported through the Inferact ModelOpt NVFP4 checkpoint's
text path; its image and video towers are not served.

## Build / run

```sh
cargo build --workspace
cargo build --release -p eider-api --bin eider-serve
cargo test --workspace

scripts/run-eider
cargo run --release -p eider-api --bin eider-serve -- \
    gemma-4-26b-a4b-nvfp4 --offline
```

Catalogue starts resolve pinned Hugging Face revisions and keep snapshots
immutable. Derived model artifacts belong below the Eider XDG cache, not in a
snapshot or the repository. Use `--model-dir` only for local development
checkpoints. Catalogue IDs select deployments; API requests use the served
model names reported by `eider model list` or `/v1/models`.

The server exposes `/v1/responses` and `/v1/chat/completions`; both translate
into the same `ChatRequest`, inference actor, scheduler, prefix cache, and model
runtime. Keep cancellation, tool history, sampling, usage, and finish-reason
semantics aligned across both adapters. Pi launchers default to Responses; set
`PI_EIDER_PROVIDER=eider-chat` to exercise Chat Completions.

The Qwen3.6 fast MoE path is enabled by default: indexed CUTLASS W4A4 gate/up
for single-row decode, grouped W4A4 for larger batches, SM12x down, segmented
decode graph capture, and the shared radix prompt-prefix cache. Compatibility
flags from older experiments are not required for normal runs.

## Microbenchmarks

```sh
cargo bench -p nvfp4 --bench sm121_w4a16_routed_gate_up
cargo bench -p nvfp4 --bench qwen36_routed_moe_decode
cargo bench -p infer --bench qwen36_prefill
cargo bench -p infer --bench step37_prefill
cargo bench -p infer --bench laguna_prefill
cargo bench -p nvfp4 --bench gemma4_prefill_attention
cargo bench -p nvfp4 --bench nemotron3_dense_linear
```

The benches use my [`micromeasure`](https://github.com/rdaum/micromeasure) crate
and should validate correctness before timing.
Keep shape-specific kernel changes backed by a focused benchmark and compare
the result against the existing reference path.

## CUTLASS / CUDA

`scripts/setup-cutlass-sm12x.sh` configures CUTLASS for `sm_121` under the
repo-local, git-ignored `.deps/` directory:

```sh
scripts/setup-cutlass-sm12x.sh
source .deps/cutlass-sm12x.env
```

The build defaults are CUDA 13.0, `.deps/cutlass`, and
`.deps/cutlass-build-sm121`; those paths are used automatically when present.
`scripts/probe-cutlass-sm12x.sh` checks CUTLASS SM12x compile support.

## TLDR SM121

For CUDA and CUTLASS work on GB10, keep these
[SM12x NVFP4 notes](https://research.colfax-intl.com/cutlass-tutorial-nvfp4-blockscaled-gemm-on-nvidia-rtx-pro-blackwell-gpus-sm12x/)
in view:

- SM121 is Blackwell, but it is not the SM100/B200 programming model. Its tensor
  cores use synchronous, warp-collective `mma.sync` with operand and accumulator
  fragments in registers; it has no `tcgen05` or TMEM. SM100 GEMM kernels are
  incompatible. SM8x mainloop, scheduling, and pipelining ideas are the more
  useful starting point, with SM12x additions such as TMA.
- Build SM121 architecture-specific MMA translation units with
  `compute_121a,code=sm_121a`, as `crates/nvfp4/build.rs` does. Do not silently
  compile them under a conservative generic target. Conversely, do not assume
  an SM120a/f-only feature such as warpgroup register reallocation exists on
  GB10; confirm support with the installed CUDA/CUTLASS toolchain and a device
  probe.
- The hardware NVFP4 atom is fixed:
  `mma.sync.aligned.kind::mxf4nvf4.block_scale.scale_vec::4X.m16n8k64.row.col.f32.e2m1.e2m1.f32.ue4m3`.
  It consumes E2M1 A `16x64`, E2M1 B `64x8`, UE4M3 SFA `16x4`, UE4M3 SFB
  `4x8`, and FP32 accumulators. The `m16n8k64` shape, `4X` scale vector, and
  UE4M3 scale type are not tuning choices.
- NVFP4 means packed E2M1 values with one UE4M3 scale per 16 values along K.
  E2M1 has only `0`, `±0.5`, `±1`, `±1.5`, `±2`, `±3`, `±4`, and `±6`; it has
  no NaN or infinity encoding, and UE4M3 scales are nonnegative.
  Logically, A `[M,K]` has SFA `[M,K/16]` and B `[K,N]` has SFB `[K/16,N]`.
  One MMA K atom therefore consumes four scales. An unguarded MMA mainloop still
  needs K in 64-element atoms; handle or pad tails rather than treating
  divisibility by 16 as sufficient.
- Keep logical, checkpoint, cuBLASLt, CUTLASS, shared-memory, and native MMA
  scale layouts distinct. The hardware fixes the register fragment layout, not
  the global- or shared-memory layout. ModelOpt row-major scales are not
  cuBLASLt `VEC16_UE4M3` scales and neither can be reinterpreted as a native MMA
  layout. Convert once while preparing/caching weights, not in the decode hot
  path.
- Scale register ownership is quad-based. With CUTLASS's selector convention,
  lanes 0 and 1 of each four-lane quad supply SFA while lane 0 supplies SFB.
  CUTLASS layouts consequently replicate SFA registers 2x and SFB registers 4x;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rdaum/eider](https://github.com/rdaum/eider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
