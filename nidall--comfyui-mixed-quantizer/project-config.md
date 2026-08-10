---
trigger: always_on
description: `comfyui_wxa8_quantizer.py` is a standalone, single-file converter that turns
---

# AGENTS.md

## Project

`comfyui_wxa8_quantizer.py` is a standalone, single-file converter that turns
generative-model checkpoints into ComfyUI-native quantized checkpoints. It
does not import ComfyUI or comfy-kitchen at runtime.

The mixed mode (`--format mixed`) is a per-layer optimizer over the
ComfyUI-native formats `convrot_w4a4`, `asym_w4a8_int8`, and
`int8_tensorwise`, merged into main from the `experimental/mixed-precision`
branch. `--format w4a8` remains the default and is byte-identical to main
v1.3.0 (golden vectors and the 39/39 self-test suite prove it).

- Local path: `/home/nidall/projects/testdeepseek/quantizationscripts_w4a8_w3a8`
- Repo (public): `https://github.com/NidAll/comfyui-mixed-quantizer`
- Branch: `main` (the experimental branch was merged)
- Script version: `1.4.0-experimental` (`CONVERTER_VERSION`)
- Audit status: P0 items closed (hard quality/compression gates, BF16
  promotion candidate, runtime-output calibration metric, per-format runtime
  compatibility probe) and the follow-up implementation plan applied
  (W4A8 runtime-basis simulation, target-accurate W4A4, simulator
  equivalence suite, targeted global metric, unified ORIGINAL candidate,
  per-format capability matrix, certification levels, model-level quality
  harness, strict mixed smoke mode, upstream runtime-contract nightly sync).
  Krea2 policy fixed for the real Kroma v0.2 naming (blocks.N / txtfusion
  layerwise+refiner, commit e0141d7; krea2-real-dims self-test pins it).
  Remaining certification gaps are documented in the README limitations
  (real-model three-layout forward, LoRA/offload/low-VRAM, e2e generation,
  balanced-v1 threshold tuning from the benchmark matrix).

## Format facts (verified, do not guess)

### W4A8 (unchanged from main)

Per quantized layer: `{layer}.weight` int8 [N, K/2] (even column low nibble),
`{layer}.weight_s_rel` fp8_e4m3fn [N, K/16], `{layer}.weight_s_channel` fp32
[N], `{layer}.weight_codebook` fp32 [16]. Metadata: `format=asym_w4a8_int8`,
`group_size=16`, `convrot=true`, `convrot_groupsize=256`. K % 256 == 0 is
required (CUDA fused kernels are ConvRot-256-only).

### W4A4 (new, convrot_w4a4)

- `{layer}.weight` int8 [N, K/2], packed signed int4, low nibble = even column
  (matches comfy-kitchen `_pack_int4_row_major`)
- `{layer}.weight_scale` fp32 [N], rowwise absmax / 7, emission range [-7, 7]
  (matches eager `quantize_convrot_w4a4_weight`; golden-vector byte digests in
  `_test_w4a4_golden`)
- Metadata: `format=convrot_w4a4`, `convrot_groupsize` (largest power of 4 in
  {16, 64, 256} dividing K), `quant_group_size=64` (kernel contract), and
  `linear_dtype` (execution variant only: int4 or int8, default int8; never a
  quality fallback)
- Eligibility: K % 64 == 0 and K % cgs == 0. Boogu 3360 and OmniGen2 2520 are
  NOT eligible (3360 % 64 = 32, 2520 % 64 = 24). PixArt 1152, HunyuanDiT 1408,
  CogVideoX-2B 1920, SDXL 320/640, MiniMax fc2 1152 ARE eligible.
- Measured weight error ~0.142, about 2x the W4A8 codebook path (0.070).
  Dequant requires the signed nibble interpretation (`unpack_int4_signed`,
  NOT `unpack_w4` which is unsigned).

### INT8 (new, int8_tensorwise)

- `{layer}.weight` int8 [N, K], `{layer}.weight_scale` fp32 [N, 1]
  (rowwise absmax / 127, clamp min 1e-30; [N, 1] so eager and CUDA backends
  both broadcast; matches eager `quantize_int8_rowwise`, golden digests in
  `_test_int8_golden`)
- Metadata: `{"format": "int8_tensorwise"}` only. No ConvRot, no K constraint.
  This is the universal fallback tier and the fix for Boogu/OmniGen2.
- Measured weight error ~0.005. Comfy-Org ships the same rowwise scheme in
  its `*_int8_convrot` checkpoints (those add convrot=true; we do not).

## Mixed mode design

`MixedPlanner` (in the converter): per-layer candidate evaluation (quantize +
dequantize each eligible format; with calibration activations the error is
the RUNTIME OUTPUT error, an exact eager-path emulation: activation rotation,
dynamic rowwise activation quantization (int8, or int4 for the W4A4 int4
variant), and the scaled quantized GEMM), cheapest-acceptable selection under
the profile's per-layer gate, then a greedy promotion loop (best error
reduction per extra byte, original precision included as the final rescue)
until the PARAM-WEIGHTED global mean gate passes. Selection and promotion
mutate `TensorDecision.format` / `.convrot_groupsize` and must run BEFORE
`build_output_entries` (output shapes and offsets depend on per-layer
formats).

`--target-runtime` feeds `RuntimeCapabilities` (`runtime_capabilities_for`)
into eligibility: nvidia = all three formats accelerated (verified on the RTX
3050), amd = HIP/triton paths (not certified here), cpu = eager fallback for
all three. Capabilities are per-format `FormatRuntimeCapability`
(loadable/executable/accelerated/backend/reason), never a single boolean.
The W4A4 simulation is dispatch-aware via
`resolve_w4a4_execution_mode` (mirrors the comfy-kitchen CUDA dispatcher:
int8 -> A8 always; int4 -> native INT4 MMA on SM8x, uncertain on Turing
SM 7.5, INT8 fallback elsewhere; eager always A4). `W4A4ExecutionMode`
carries activation_bits/path/certain/reason. Uncertain modes evaluate BOTH
A4 and A8 and score with the WORST error (never optimistic). A runtime

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NidAll/comfyui-mixed-quantizer](https://github.com/NidAll/comfyui-mixed-quantizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
