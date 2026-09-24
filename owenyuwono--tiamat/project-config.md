---
trigger: always_on
description: Real-time fluid simulation with GPU solvers and raymarched water rendering. SPH is production-ready; FLIP and Euler are disabled in UI (coming soon).
---

# fluid3

Real-time fluid simulation with GPU solvers and raymarched water rendering. SPH is production-ready; FLIP and Euler are disabled in UI (coming soon).

## Stack

- Three.js (v0.184) for camera/controls math only (WebGPU path) or full rendering (CPU fallback)
- WebGPU compute shaders for simulation + WebGPU render pipeline for raymarching
- CPU fallback: Three.js WebGL2 rendering + CPU SPH (`src/sph/`)
- TypeScript, Vite
- GLSL shaders imported via `?raw`, WGSL shaders via `?raw`

## Architecture

### GPU Pipeline (`src/gpu/`) — zero CPU readback

Single WebGPU device handles both compute and render. One command encoder per frame.

**Three solvers, switchable at runtime:**
- **`GPUCompute.ts`** (SPH) — Lagrangian particle solver. Spatial hash grid (`MAX_PER_CELL=16`), pairwise kernel density/force computation, Tait pressure, XSPH smoothing, mirror boundary forces
- **`FLIPCompute.ts`** (FLIP) — Hybrid PIC/FLIP solver. Particles → staggered MAC grid (P2G), pressure projection via Jacobi iteration, grid → particles (G2P). Uses 6 staggered face velocity buffers for proper incompressibility
- **`EulerCompute.ts`** (Euler) — Pure grid Navier-Stokes solver. Semi-Lagrangian advection (RK2) on staggered MAC grid, 80-iteration Jacobi pressure projection, velocity extrapolation into air cells (Bridson §6.3), upwind density advection with 4 substeps. No particles during simulation — fixed O(n³) cost independent of fluid volume. Uses `copyBufferToBuffer` for velocity advection ping-pong, bind group ping-pong for density and pressure Jacobi

All three implement the same interface: `encodeStep()`, `uploadInitialPositions()`, `resetVelocities()`, `getDensityFieldBuffer()`, `getParamsBuffer()`, `updateSimConfig()`

**`WebGPURenderer.ts`** — render pipeline orchestrator
- Creates `rg32float` 3D texture (100^3, STORAGE_BINDING | TEXTURE_BINDING) shared between compute and render
- `bufferToTexture.wgsl` compute pass converts u32 density buffer → 3D texture with 3x3x3 box filter (workgroup 4,4,4)
- `waterRaymarch.wgsl` renders fullscreen triangle with WGSL raymarching (400 iterations, step 0.025)
- `floor.wgsl` renders textured sand floor slab matching container footprint
- Wireframe pipeline exists but is not drawn (removed from render pass)
- Water bind group includes sand texture + repeat sampler (bindings 3-4) for refraction
- `rebindComputeBuffers()` swaps density/params buffers when switching solvers without recreating pipelines/textures
- `loadFloorTexture()` recreates both floor AND water bind groups when sand texture changes

**SPH frame pipeline:**
```
1. clearGrid → insertParticles → [substeps × (density → forces → integrate)]
2. clearDensityField → splatDensity (atomic u32 buffer)
3. bufferToTexture compute (u32 → rg32float 3D texture)
4. render pass: clear bg → draw floor → draw water (fullscreen tri, alpha blend)
```

**FLIP frame pipeline (disabled — see known issues):**
```
1. encoder.clearBuffer × 15 (zero all grid buffers via DMA, no shader needed)
2. flipP2G → flipNormalizeA (U+V faces) → flipNormalizeB (W faces + fluid marker)
3. [substeps × (flipDivergence → 40× jacobi → flipProject → flipProjectStaggered)]
4. flipG2P → clearDensityField → flipSplatDensity
5. bufferToTexture → render pass (same as SPH)
```

**Euler frame pipeline:**
```
1. eulerClearGrid (zero pressure/divergence)
2. eulerAdvectVelocity (semi-Lagrangian RK2 → temp buffers)
3. copyBufferToBuffer (temp → main velocity)
4. eulerApplyForces (gravity on v-faces)
5. eulerDivergence → [80 × eulerJacobi] → eulerProject
6. eulerExtrapolateInit (classify face validity from density markers)
7. eulerExtrapolateSweep × 3 (extend velocity into air, ping-pong valid buffers)
8. eulerAdvectDensity × 4 (upwind finite-difference, ping-pong density)
9. eulerWriteDensity (f32 marker → atomic u32 densityField)
10. bufferToTexture → render pass (same as SPH/FLIP)
```

**Params struct** (128 bytes, 32 fields) shared across all shaders — WGSL struct layout must match TypeScript array indices exactly. All three solvers write the same layout so the renderer doesn't care which solver is active.

### UI (`src/ui/`)
- `AlgorithmPicker.ts` — standalone SPH/FLIP/Euler toggle with technical descriptions
- `ControlPanel.ts` — particle count slider, physics sliders, render sliders, pause/reset, light toggle
- `SimConfig.ts` — all runtime-tunable parameters, `Algorithm` type (`'sph' | 'flip' | 'euler'`)
- `StatsPanel.ts` — FPS, GPU timing (when `timestamp-query` available), substep count

### CPU Simulation fallback (`src/sph/`)
- `simulation.ts` — SoA particle data (Float32Array), prefix-sum spatial hash, same physics as GPU SPH
- `constants.ts` — all physics tuning parameters (shared by both GPU and CPU paths)
- Used when WebGPU is unavailable (`GPUCompute.create()` returns null)
- Falls back to Three.js WebGLRenderer with `WaterRenderer.ts` (Data3DTexture + GLSL raymarching)

### Rendering fallback (`src/rendering/`)
- `WaterRenderer.ts` — Three.js ShaderMaterial with BackSide box rendering, `transparent: true`, `depthWrite: false`
- `DensityField.ts` — CPU splatting into Float32Array backing a Data3DTexture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owenyuwono/tiamat](https://github.com/owenyuwono/tiamat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
