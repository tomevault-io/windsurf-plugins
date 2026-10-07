---
trigger: always_on
description: Reimplement the inference pass of NVIDIA's DLSS 5 Neural Rendering (DLSS-NR) so it
---

# DLSS 5 Neural Rendering on Intel Xe2 (Lunar Lake)

## What this project is

Reimplement the inference pass of NVIDIA's DLSS 5 Neural Rendering (DLSS-NR) so it
runs on an Intel Xe2 integrated GPU.

As of 2026-09-07 no public port exists for **any** Intel GPU. The only non-NVIDIA
implementation is `danielblnc/DLSS-NR-on-AMD` for RDNA4, which works because RDNA4
has native FP8 E4M3 matrix hardware. That approach does not transfer here — see
"No FP8" below.

**Success = one frame goes in, a neurally-rendered frame comes out, and it matches
a CPU reference.** Playable framerates are explicitly *not* a goal. Do not propose
optimisations that trade correctness for speed until Phase 3 is done.

> **DONE 2026-09-10, further than the line above set out.** A frame of **Dead or Alive
> 5** goes out of a running game, through the recovered 71-block graph on this iGPU, and
> back into the game's own swapchain — hair separating into strands, eyelashes
> resolving, skin picking up texture that is not in the input. 1280x720 in **~0.49 s**
> of graph time, correlation with the CPU reference **0.981311**, and every optimisation
> since has kept the output bit-identical.
>
> `notes/phase34-doa5.md` for the game, `phase7-first-render.md` for the first frame,
> `phase8-xmx-graph.md` for the XMX path.
>
> **Read `notes/HANDOFF.md` first**; it overrides this file, and large parts of what follows
> are the record of how the answers were reached rather than the answers.
> `notes/INDEX.md` maps every phase note to the question it settles.

---

## Hardware (already probed — do not re-probe, trust these values)

- Acer Aspire A14-52M, Intel Lunar Lake, Arc 140V-class iGPU (Xe2 / Battlemage-family)
- Arch Linux, Mesa ANV Vulkan driver, `intel-compute-runtime` for OpenCL
- **15 GiB total RAM, shared with the iGPU.** 7.6 GiB swap. There is no dedicated VRAM.
- OpenCL stack alive: `Platform #0: Intel(R) OpenCL Graphics / Device #0: Intel(R) Arc(TM) Graphics`

Vulkan capabilities, confirmed via `vulkaninfo`:

```
VK_KHR_cooperative_matrix                : extension revision 2
cooperativeMatrix                        = true
cooperativeMatrixRobustBufferAccess      = false
cooperativeMatrixSupportedStages         = SHADER_STAGE_COMPUTE_BIT   (compute only)
shaderBFloat16CooperativeMatrix          = true                        <-- primary path
VK_NV_cooperative_matrix2                : extension revision 1        (Mesa impl)
```

XMX matrix engines support: TF32, FP16, BF16, INT8, INT4, INT2.
**They do not support FP8.** Xe3 / Panther Lake does not add it either.

**Verified 2026-09-07** (`src/probe/coopmat_probe.c`, full table in
`notes/hw-coopmat.md`): Mesa exposes exactly **6** cooperative matrix configs, all
scope = subgroup, all **M=8 N=16** (K=16 float, K=32 int8) — fp16 and bf16, each
with either same-type or **fp32** accumulate, plus sint8/uint8 → int32.
`subgroupSize` = 32. TF32 and INT4/INT2 are *not* reachable from Vulkan even though
the DPAS hardware has TF32 (OpenCL advertises
`cl_intel_subgroup_matrix_multiply_accumulate_tf32`). `VK_NV_cooperative_matrix2` is
advertised but reports **zero** flexible-dimension configs.
The **primary path is config 1, `fp16 × fp16 → fp32`** — the shipped weights are FP16
(see below), so nothing is converted. The bf16 config is a fallback, not the plan.

---

## The weight format — what is live, and what was wrong twice

This section has been wrong **three** times, so it is worth stating carefully what survived.

> **Wrong the third time, corrected 2026-09-16** (`notes/phase61-the-parameter-count.md`).
> The model has **145 755 123 parameters**, not 73 841 889. The large matrices — 143.0 M of
> them — are stored as **FP8 E4M3, one byte each**; the small tensors are FP16 and
> `attn_scale` is FP32. That fits the 147.7 MB section to 0.5 %, where dense FP16 would need
> 291.5 MB. **73 841 889 is the section's bytes divided by two** — the dense-FP16 misreading
> this section already warned against, surviving in the count after it had been corrected
> in the encoding. The early "~148 M FP8" figure it dismissed was essentially right.
> Also withdrawn with it: GQA 4:1 (`qkv` is `(C, 3C)`, full MHA) and "27 % subnormals" —
> the real weights hold 7.

**Live, and load-bearing:**

- **There is no FP8 anywhere in our *compute* path.** The XMX units have no FP8, so the
  E4M3 matrices are decoded to FP16 at extraction and computed with from there. That decode
  *is* a dequantisation step — the weights are not "used as they arrive".
- **The primary path is FP16, not BF16.** Config 1 of the cooperative-matrix table is
  `fp16 × fp16 → fp32`, and the logical weights are FP16, so they reach the matrix units
  with **zero conversion**. **Do not convert to BF16** — it would throw away 3 of 10
  mantissa bits to reach a format the hardware likes no better, and phase 22 measured
  exactly that.
- **145 755 123 parameters.** On disk they take 147 MB because most are one byte; decoded to
  FP16 for compute they take about 292 MB.

**Superseded, and the trap to avoid:** the container does *not* hold plain dense FP16.
It holds **packed backend payloads** — permuted into `mma` fragment order, partly E4M3 —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Uzbekunknown/dlss-nr-on-intel](https://github.com/Uzbekunknown/dlss-nr-on-intel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
