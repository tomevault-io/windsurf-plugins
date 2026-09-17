---
trigger: always_on
description: FSR1 spatial + FSR2/3-style **temporal** upscaling for three.js `WebGPURenderer`, as hand-written **WGSL compute passes**. WebGPU-only, TypeScript, no TSL, no WebGL fallback. Extracted from the homefig monorepo into this standalone repo (`pmndrs/upscaler`).
---

# @pmndrs/upscaler — Claude Code Instructions & Handoff

FSR1 spatial + FSR2/3-style **temporal** upscaling for three.js `WebGPURenderer`, as hand-written **WGSL compute passes**. WebGPU-only, TypeScript, no TSL, no WebGL fallback. Extracted from the homefig monorepo into this standalone repo (`pmndrs/upscaler`).

---

## ⚠️ READ FIRST: current status

**The pipeline now runs correctly on a real GPU** (verified 2026-07-07 on Apple Metal-3
via headless Chrome + CDP). All four bench modes — native, bilinear, FSR1 spatial, FSR3
temporal — render correctly, and all four debug views (motion vectors, disocclusion,
depth, accumulation age) are validated. First-GPU-boot surfaced exactly two bugs, both
now fixed (see "landmines defused" #7–#8 below).

**Renamed to `@pmndrs/upscaler`** (2026-07-10, for the move to the Poimandres org):
every public `FSR3*`/`fsr3*` identifier dropped the AMD product mark — `Upscaler`,
`UpscalePass`, `UpscalerNode`, `upscale()`/`upscaleScene()`/`upscaleSpatial()`,
`QualityMode`/`DebugView`, `UpscalerConfig`/`DispatchInputs`/`RuntimeSettings`/`UpscalePath`.
**Attribution stays** — `LICENSE` keeps AMD's MIT notice, the README credits FidelityFX
FSR (nominative fair use), and internal WGSL port names (`FsrEasuF`, `FsrConstants`,
`FLAG_*`) are kept verbatim for provenance. Re-verified on GPU the same day (bench all
modes + examples 01/05/07/09) under the new names; the reactive-on-node + external-exposure
features shipped this session are GPU-clean too.

Still true: the unit tests are GPU-free (pure math + shader-string structure); the
fidelity/tuning of the temporal path (accumulation, disocclusion thresholds, motion
convention) is correct enough to render cleanly but **not tuned** — the "landmines still
live" section remains the guide for visual regressions.

**Parity program concluded (2026-07-21):** the three source-style FSR 3.1.5 candidate
graphs were GPU-verified and A/B-benchmarked against production — **+36% / +6.5% /
+76% GPU compute with no visual win**; none adopted. Consumer-facing rationale in
`PARITY.md` (root); evidence + decisions in `bench/docs/PARITY-DECISIONS.md` /
`PARITY-CANDIDATES.md`. **Post-parity
items 1–3 landed the same day** (see `bench/docs/NEXT-STEPS.md` for evidence):
(1) RCAS now sharpens in conditioned tonemap space, inverting once — **−34% RCAS,
−5.7% total** with capture-identical output; the old form is frozen as
`RCAS_PER_TAP_SHADER` under the `rcas-fsr315-limiter` bench identity. (2) Host
pre-exposure (`preExposureTexture`) is honored end-to-end — DeltaPreExposure history
correction + host-invariant auto-exposure metering, validated on the new **Q11**
scenario, byte-identical when absent. (3) The reconstruct pass uses AMD's
viewport/depth-scaled disocclusion (per-tap confidence voting) inside the fused
single pass. **Item 4 (the multi-scale shading-change detector) landed the same session**: the
3×3-neighborhood shading heuristic is replaced by `shadingChange.ts` — a fused
multi-scale block-mean detector (0.044 ms, 5× cheaper than the candidate's
two-pass form, measurably fewer false positives under motion; five GPU tuning
iterations documented in NEXT-STEPS). Nothing from the parity program remains
open. Candidate
A/B runs: `node scripts/run-benchmark.mjs --smoke --variant <A> --comparison <B>`
(see `--help`).

If you touch shaders/passes, re-verify on a real GPU. A dependency-free way to do it
headlessly (no Playwright): launch Chrome with `--headless=new --enable-unsafe-webgpu
--remote-debugging-port=N`, drive it over the DevTools Protocol (Node 22 has a native
`WebSocket`), collect `Log.entryAdded` (WGSL validation errors surface here) +
`Runtime.consoleAPICalled`, and `Page.captureScreenshot`. lil-gui dropdowns are real
`<select>` elements you can set + dispatch `change` on to flip modes/debug views.

```bash
npm install
npm run dev        # http://localhost:5199 — open in Chrome/Edge 113+ (real WebGPU)
```

When something breaks after an edit, expect failures in this order of likelihood:
1. **WGSL validation errors** at pipeline creation (binding types, storage formats, struct layout). The browser console prints exact line/column — these are quick.
2. **Bind-group / layout mismatches** — a pass's `createBindGroup` entry order must match its WGSL `@binding` order.
3. **Visual wrongness** even when it runs: black output, garbage history, smearing, wrong colors. Use the debug views (below) to localize before touching shader math.

Don't trust "it builds" as "it works." Drive the real bench.

Bench caveat (measured 2026-07-21): benchmark runs launched from a scratchpad
**git worktree** read ~3× slower absolute GPU times, uniformly across all passes
(the GPU never leaves its low power state — likely cold-vite frame delivery).
A/B comparisons *within* that environment are valid; never compare worktree
absolutes against repo-run records. Also: a `node_modules` **symlink** in a
worktree isn't matched by the root `.gitignore`'s `node_modules/` pattern
(trailing slash ≠ symlink) and crashes the benchmark's working-tree digest —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pmndrs/upscaler](https://github.com/pmndrs/upscaler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
