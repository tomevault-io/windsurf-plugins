---
trigger: always_on
description: Read this before writing kernels, tests, or chasing bugs. It is **how to work
---

# diffgemma — how to work on this codebase

Read this before writing kernels, tests, or chasing bugs. It is **how to work
here without repeating the mistakes that have already cost days.**

The project: a Rust + Metal inference engine for DiffusionGemma (Gemma-4
26B-A4B MoE, discrete block diffusion) on Apple Silicon.

Doc map — three documents, each with one job:
- **ARCHITECTURE.md** — the design and the implemented generation contract,
  including the *Negative Knowledge* section (disproven approaches and the
  physics that blocked them).
- **PLAN.md** — open work only.
- **AGENTS.md** (this file) — working discipline.
- Everything else lives in **git history** (thorough commit messages are the
  changelog) and the agent memory. Do NOT add changelog-style banners,
  dated status sections, or "history of this fix" narratives to code or
  docs — describe the present; commit messages record the past.

Authoritative numeric behavior: the CPU reference (per-kernel `cpu.rs`
oracles under `src/shaders/`, shared ops in `src/shaders/cpu/`, plus
`sample.rs`) and the weight manifest (`model.dgq.json`).

---

## 1. The one thing to internalize

**Every serious bug in this project has lived in a fused or accelerated GPU
path that a slower reference path got right.** MPS-Q4 producing uniform
logits, the SC GEMM transpose, the softmax grid collapse, the MoE
route-garbage from a last-expert `n_tok` bug, the fast-prefill encoder
running a denoise-only norm — all the same shape: the optimized path computes
a different function than the reference, parity is green because the
optimized path has no golden, and the symptom only appears downstream
(layer 2+, entropy collapse, pad output, >2.5k-token collapse) far from the
cause.

The corollary that governs everything below: **an untested path is where the
next bug is.** Speed without a per-path correctness gate is how bugs ship.
So the strategy is not "go fast"; it is "make divergence impossible to
introduce silently, then go fast."

---

## 2. Diagnostic discipline (how to chase a bug)

When output is wrong, follow this order. Do not skip to optimization or to
rewriting a kernel you can't explain.

1. **Localize before you theorize.** Find the *smallest* unit that diverges
   from the reference. A 30-layer entropy collapse is not a bug location;
   it's a symptom. Bisect: which layer, which kernel, which stage. The MoE
   hunt took ten turns because we theorized about kernel math for rounds
   before dumping the one value that localized it in a single read.

2. **Same input, same weights, two paths.** Run the suspect GPU path and the
   CPU reference on *byte-identical* input, compare cosine, and read the
   magnitude as a diagnostic: >0.999 = that stage is fine; ~0.5 =
   correlated-but-wrong (a swap, a scale, partial corruption); ~0 = reading
   the wrong data entirely.

3. **Dump what the kernel actually reads, not what you think it reads.** A
   CPU transliteration of the *source* can match the reference while the
   *GPU execution* diverges — it can't reproduce threadgroup semantics, arena
   bindings, or route resolution. Write the kernel's real inputs to a scratch
   buffer and read them back.

4. **Impossible numbers mean wrong N or wrong normalization.** Entropy >
   ln(N), Z=0, cos > 1, values at ~1e38 are never "the model is just bad" —
   they are indexing, normalization, or overflow bugs with a specific cause.

5. **Two paths failing differently is a gift.** The *difference* localizes the
   bug to path-specific code. One exploding (inf) and one inert (zero) meant
   two bugs in the same conceptual spot, and finding one explained the other.

6. **A contradiction is a second bug, not an anomaly.** If a fix changes an
   intermediate but not the output, you are measuring two different code
   paths (probe vs production) or reading a stale buffer. Resolve it.

7. **Don't let a workaround end the investigation.** Falling back to a slow
   reference path unblocks convergence but leaves the latent bug in every
   other kernel sharing the flawed pattern.

8. **Reproduce a gap across inputs before calling it a bug, and change one
   variable at a time.** In the denoise loop a sub-1e-4 per-step difference
   can flip an accept decision and cascade into a different but equally valid
   trajectory, so a single-prompt delta can be chaos rather than a defect —
   check it is systematic before chasing it. When it is real, bisect to the
   axis (sampler / forward precision / a specific quantized tensor) before
   rebuilding; "rebuild with everything different" tells you nothing.

9. **Measure value/type ranges before calling a precision experiment failed**
   (`DGQ_TRACE_RANGES`, value-cos), and check ARCHITECTURE.md's Negative
   Knowledge + agent memory for the lever family BEFORE planning perf work —
   task descriptions carry stale premises.

---

## 3. Measure before optimizing — always

- **Get a clean measurement first.** Compile out probes/readbacks before
  timing anything. A readback is a GPU pipeline stall; instrumentation can
  dominate a step.
- **Attribute the time before reducing it.** Per-dispatch timing on one clean
  step. Do not guess which stage dominates; the guesses have been wrong.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmastrac/diffgemma](https://github.com/mmastrac/diffgemma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
