---
trigger: always_on
description: Read these files before changing code:
---

# Agent instructions

## Start here

Read these files before changing code:

1. `README.md`
2. `docs/architecture.md`
3. `docs/roadmap.md`
4. `docs/compatibility.md`
5. `docs/plans/2026-08-29-mlx-omarchy-ane-compatibility-plan.md`
6. `docs/plans/2026-09-12-coreml-parakeet-ane-plan.md` (Core ML/Parakeet
   plan; package inspection, reference capture, and compiler preparation
   exist in source. Linux encoder execution remains unqualified.)

Run `scripts/prepare-mlx.sh` before inspecting the pinned MLX source.
Inspect the affected backend under `.work/mlx`.
Keep project-owned files under `overlay/` and upstream edits under `patches/`.

Use the CUDA backend for the complete GPU boundary.
Use `mlx/backend/no_cpu/` and `mlx/backend/no_gpu/` for failure behavior.

## Fixed product contract

- Keep the public MLX Python import as `mlx`.
- Keep `mx.gpu` as the public accelerator device.
- Use Honeykrisp Vulkan for complete tensor execution.
- Treat ANE as an internal graph-region accelerator.
- Let CPU code schedule, copy, tokenize, and serve requests.
- Never run a tensor primitive on CPU in a release build.
- Return an exact compatibility error when Vulkan and ANE cannot run a primitive.
- Do not emulate Metal on Linux.
- Qualify M1 Omarchy before other Apple Silicon models.

Do not weaken these rules to make a model run.
Add the missing accelerator implementation or keep the test failing.

## Repository boundaries

- This repository owns the Omarchy backend, ANE partitioner, patches, packaging, tests, and releases.
- Upstream MLX source and history must not enter this repository.
- `joshuaswarren/ane-linux-experiments` owns hardware probes, format research, and unstable fixtures.
- `joshuaswarren/omarchy-ane` owns the ANE DRM driver and `libane` ABI
  (upstream lineage and backport flow: `docs/forks.md`).

Do not copy experimental driver code here.
Prove a driver change in the experiment repository, then send the smallest stable ABI change to `joshuaswarren/omarchy-ane`.

## Upstream MLX sync

Pin the upstream release, commit, archive URL, and SHA-256 in `mlx.lock`.
Fetch upstream source only through `scripts/prepare-mlx.sh`.
Add new project files under `overlay/`.
Keep modifications to existing MLX files in a small patch series.
Rerun runtime, primitive, and semantic gates after every baseline change.
Keep Omarchy work behind `MLX_BUILD_OMARCHY`.
Do not rename public MLX APIs to fit backend code.
Resolve an upstream change at the shared MLX contract before adding an Omarchy-only path.

This repository is independent from Apple.
Do not open an upstream MLX pull request or push to an Apple remote from an agent session.

## Implementation order

Work in the dependency order in `docs/roadmap.md`.
The Vulkan baseline does not wait for ANE.
Owner direction, 2026-09-12: the Core ML/Parakeet ANE lane
(`docs/plans/2026-09-12-coreml-parakeet-ane-plan.md`) runs concurrently
with GPU performance-parity work; neither lane waits on the other's
numbers. The correctness gates do not move: the ANE exporter still
waits for the MIL compiler proof, general MLX lowering still waits for
the one-operation MIL proof and the stable `libane` ABI, and the
exporter still does not wait for dma-buf support.

Implement primitives in model-driven order, but keep each primitive general.
Do not add a model-name branch inside a core tensor kernel.
Reuse backend-neutral MLX code before adding Omarchy code.
Run the U2 matmul and attention speed spike before broad primitive work.
Stop and revise the backend design if the spike misses its fixed go or no-go gate.
Partition ANE regions before `mx.compile` fusion.
Represent each selected region as an opaque `AneRegion` primitive.
Send the rest of the graph through Vulkan fusion.

## Hardware safety

The owner authorized both the 13-inch M1 and 16-inch M1 Max Omarchy
laptops for installation, hardware testing, and necessary reboots
(2026-09-12). Consult the private fleet inventory for SSH targets.
Use an independent `/tmp/m1-gpu.lock` on each physical host. Exclude a
serving host from LiteLLM and confirm it is quiescent before benchmarking.
Native performance comparisons must match the physical machine; never
use the base M1 as the M1 Max denominator.

**Hardware status (2026-09-20).** m1-test-host (13-inch M1) runs provisioned
Omarchy; its Linux ANE qualification ladder has Step 5 closed — full-ASR
104/104 across two bit-exact repeats on the fork driver and a 10/10
perf battery (encoder AC median 5217.4 ms vs 259.9 ms macOS same-encoder
divisor, cross-OS indicative, no parity claim;
[`ane-linux-experiments` 3babdb5](https://github.com/joshuaswarren/ane-linux-experiments/commit/3babdb5)
/
[119b954](https://github.com/joshuaswarren/ane-linux-experiments/commit/119b954)).
Hybrid partition and full-encoder coverage are pending. Pre-provisioning
M1 ANE / Parakeet numbers in this tree are historical dated evidence from
prior m1-test-host Linux boots. t6001-test-host (T6001) Linux ANE is live. t6021-test-host (T6021)
Linux reads kernel 7.1.13-3-1-ARCH stable, ANE_UNBOUND, no
`/dev/accel/accel0`. T6021 GPU is qualified
([`ane-linux-experiments/receipts/2026-09-23-m2-gpu-qwen38/README.md`](https://github.com/joshuaswarren/ane-linux-experiments/blob/main/receipts/2026-09-23-m2-gpu-qwen38/README.md));

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshuaswarren/omarchy-mlx](https://github.com/joshuaswarren/omarchy-mlx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
