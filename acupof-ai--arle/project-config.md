---
trigger: always_on
description: Assisting **ckl**. Project caveats and hard gates only — generic Rust / CUDA /
---

# ARLE — Agent Contract

Assisting **ckl**. Project caveats and hard gates only — generic Rust / CUDA /
Metal / git knowledge is intentionally absent, and so is anything you can read
off the file tree. Match the surrounding code's idiom, naming, and comment
density rather than a style rulebook.

**Load on demand, not upfront:**

| When | Read |
|------|------|
| Evidence bar, decomposition, distilled lessons | [`docs/agent-method.md`](docs/agent-method.md) |
| Any bench or trace | [`docs/bench-and-trace-spec.md`](docs/bench-and-trace-spec.md) |
| Where code lives / execution paths | [`docs/codebase-map.md`](docs/codebase-map.md), [`docs/architecture.md`](docs/architecture.md) |
| Editing `crates/{autograd,cuda-kernels,mlx-sys}/` | that crate's `AGENTS.md` |
| Backend / model / quant support level | [`docs/support-matrix.md`](docs/support-matrix.md) |
| Env vars, SM tier policy | [`docs/environment.md`](docs/environment.md) |
| Session start | [`docs/index.md`](docs/index.md) (PARA index) |

`AGENTS.md` is canonical; `CLAUDE.md` is a symlink to it.

---

## Project shape

`ARLE` is a Rust-native, device-neutral inference runtime with an integrated
local agent and **On-Policy Distillation (OPD)** workflows. No PyTorch, no Python
on the hot path.

Two backends plug into one seam (`infer_seam::{BackendExecutor, KvPool}`, two
host-only traits): CUDA continuous batching (`cudarc` + vendored FlashMLA /
DeepGEMM / DeepEP + TileLang AOT + native CUDA C) and Metal (`crates/mlx-sys`
C++ bridge, packed varlen decode). One `infer_core::Engine<E, K>` drives both —
**the seam is the engine's cost contract (submit/poll + `StepLimits` +
explicit capability accessors): a new backend implements the core loop and
declares its capabilities, and a family whose decode is not submit-poll
shaped forks the loop (precedent: `diffusion_executor.rs`) rather than
bending the trait.**

Non-obvious ownership:
- **`infer-*` owns serving/runtime truth.** The monolithic `infer/` crate was
  deleted 2026-06-04 (`e81b98fb`, ~167k LOC) — any doc or command referencing
  `infer/` or `-p infer` is stale.
- `infer-api` (`LoadedInferenceEngine`) is the single programmatic entry point;
  `arle` is the CLI entry point.
- **`train` is OPD-only** (no second product line). Scratch pretrain / SFT /
  GRPO / multi-turn RL were deleted in the 2026-05-18 pivot (pretrain unwinnable
  at a 322× gap; the rest duplicate vLLM+verl / TRL / axolotl). OPD is the one
  axis where ARLE's runtime authority differentiates.
- CUDA kernels: adopt-official-first (`vendor/`), hand-rolled at
  `crates/cuda-kernels/csrc/` only for the genuine gap.

**Metal canonical model — globally unified:
`mlx-community/Qwen3.6-35B-A3B-4bit`** (MoE, ~19 GB, HF-cached) — the default for
every Metal serve, `scripts/bench_*.sh`, smoke, and Metal wins/errors; detects
MoE regressions a dense model cannot. Unit-test opt-out:
`INFER_TEST_MODEL_PATH=models/Qwen3.5-0.8B-MLX-4bit` (document why). CUDA benches
keep their own defaults.
- **Auto-wired-limit** (always-on): the Metal executor pins weights via
  `mlx::set_wired_limit` at construction (`infer-metal/src/wired_limit.rs`) —
  c=1 p99 86→15 ms on Qwen3.6.
- **`MLX_MAX_OPS_PER_BUFFER` / `MLX_MAX_MB_PER_BUFFER` are not defaults** — a
  Qwen3.5-dense tune, wash-or-loss on Qwen3.6 MoE. Per-workload matched-A/B only.

---

## Hard gates

**Backend isolation (CRITICAL).** Never `cfg`-leak backend types into
cross-backend modules — everything above the seam (`infer-core` / `-server` /
`-api`) stays device-neutral; backend types live only in `infer-cuda` /
`infer-metal`. CUDA stubs on other targets: `todo!("GPU required: ...")`. Mac
pre-push lint without nvcc (CI Lint mirror; `CUDARC_CUDA_VERSION` skips the
toolkit probe):
`CUDARC_CUDA_VERSION=12080 cargo clippy -p infer-api --release --no-default-features --features cuda,no-cuda,nccl,deepep --lib -- -D warnings`.
Run it before pushing any `infer-cuda` / `cuda-kernels` / cuda-gated `cli`
edit; `metal,no-cuda` and plain `cargo check` never see those lints.

**Every runtime change produces a bench entry.** A dated entry under
`docs/experience/wins/` (or `errors/` on regression) — no entry, not shipped. In
scope: `crates/infer-*/src/`, `crates/cuda-kernels/csrc/`, `crates/mlx-sys/src/`,
`src/`, `scripts/bench_*` param changes, feature-flag default flips, hot-path dep
bumps. Exempt: docs / agent files / memory / dev-only tooling — say so in the
commit body. Can't run locally (CUDA on a Mac) → stub `pending-remote` and cite
the remote ticket; no silent skips. Minimum, params, and the A/B contract live in
the bench spec.

**GPU kernel work** ships a measured before/after — `ncu` (CUDA) or Xcode Metal
capture / MLX instruments (Metal).

**Fast path only.** If it only works on the eager/un-captured path,
it isn't done — no per-step readback/sync in the hot loop; capture at sync
points, rebuild transient state on restore.

**Correctness parity = the correct-inference gate** (byte-identity is not
required, due to MoE non-determinism): `scripts/needle_gate.py` + `scripts/lever_gate.sh`, needle
ladder ×3 same-config vs the baseline envelope. Default flips additionally need a
wall-clock perf license.

**No half-states.** Finish a refactor unit or revert it; never leave parallel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acupof-ai/arle](https://github.com/acupof-ai/arle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
