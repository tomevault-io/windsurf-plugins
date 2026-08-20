---
trigger: always_on
description: Browser CAM for pen plotters. Three machine families: G-code plotters (Prusa printers + a
---

# Kurvengefahr — pen-plotter CAM

Browser CAM for pen plotters. Three machine families: G-code plotters (Prusa printers + a
spring-loaded pen-holder toolhead, or any G-code machine), AxiDraw-style EBB machines (streamed
live over Web Serial), and GRBL 1.1 plotters (G-code download or live Web Serial). Client-only
React/TS SPA. Inputs: handwriting (the original MVP), text, vector shapes/paths, SVG and DXF
import, raster stylization, generative primitives, and Logo programs — all reduced to the same
`Stroke[]` IR.

This file holds only what's useful in *every* context: the architecture to keep in mind, style
preferences, and non-obvious design choices. Area-specific invariants live in that area's module
headers (they're deliberately rich — read them before changing a subsystem). Project memories hold
only workflow & communication preferences; anything all contributors' agents should know lives
here.

## Architecture

**All "fancy" geometry/toolpath compute lives in Rust** (`crate/`, compiled to WASM; `npm run
build:wasm` after any change). TS owns the app shell, UI, view-state, and boundary marshalling —
nothing more. New geometry computation goes in Rust unless genuinely view-coupled (per-frame
render loops, DOM viewport math). No TF.js / no JS ML — the handwriting model is a pure-Rust
forward pass. Geometry tolerances/tessellation resolution are centralized in `crate/src/tess.rs`.

**The IR is the waist**: `Stroke` = one pen-down polyline in mm + metadata (`src/core/types.ts` ↔
`crate/src/geom.rs`). Everything that makes marks produces `Stroke[]`; everything that makes
motion consumes it. A new input type is just a new registry `generate()` — nothing downstream
changes. The unifying abstraction is the IR, not a shared trait.

**The editor targets an *abstract* plotter.** The IR and everything upstream (elements, effects,
optimize, canvas) speak machine-neutral terms: mm, a `pen` index, `pressure` 0..1. Realizing them
as physical motion (Z heights, servo angles, feeds, G-code) is the machine profile + emit/plan
layer's job *alone* — machine concepts must not leak into core copy, IR fields, or generators.
`MachineProfile` is a discriminated union on `kind`, so the compiler enumerates every consumer
when a branch changes.

**Stroke metadata encodes the cheap-invalidation contract:**

- `pen` — stamped at concatenation from `DocElement.pen`, NOT a geometry param, so a pen change
  never regenerates. Natively multi-colour generators (Logo `setpen`, containers) register
  `multiPen` to opt out of stamping. The pen list order **is** the plot order; the optimizer keeps
  each pen's strokes contiguous with an operator pause between groups.
- `pressure` (per-point 0..1) — the element's single pressure is a *gain* applied at concatenation,
  never an overwrite, so variable-pressure generators compose with the knob for free. Rendered as
  line weight on screen; realized per machine (e.g. interpolated pen-down Z) at emit.
- `reversible` — the optimizer may flip the stroke. `group` nonzero = one locked, ordered,
  contiguous chain (handwriting's reading order); 0 = free singleton in the optimization bag.

**Pipeline** (`src/core/pipeline`): generate (Rust, memoized on a stable hash of geometry-affecting
params) → effect (Rust, local mm, memoized) → place (local→page affine) → clip (to the reachable
region) → optimize (per-pen, chain-aware) → emit G-code *or* plan an EBB segment tape, by machine
kind. `effectedLocal` is the single local-geometry accessor everything rendering/plotting goes
through, so the canvas shows exactly what plots. Invalidation taxonomy: params → regenerate;
effects/transform/pen/pressure → re-place; machine settings → re-emit only.

**WASM boundary**: flat CSR typed-array buffers (`src/core/wasm/serde.ts` ↔ `geom.rs`), one decode
path; call `.free()` after reading a returned struct. Main-thread WASM initializes before first
render, so geometry calls are synchronous in app code. The exceptions are the three worker-backed
generators — handwriting (manual regenerate; the model run is slow), raster and Logo (live,
debounced) — each with its own WASM instance behind one shared message protocol/controller
(`core/generation.ts`). Rich param unions cross the boundary as one JSON string (serde struct =
schema) instead of positional args.

**Determinism is a memoization contract**: generation is assumed pure per (params, seed) — the
registry caches on exactly that. Seeded randomness only (re-roll = new seed); never wall-clock
anything in a generator (Logo's runaway limits are deterministic budgets, not timers).

**Coordinate spaces** (plotter bugs live in the seams): element-local mm → page mm (top-left, +Y
down — the editor is *always* page space) → machine mm (Y-flip iff bottom-left origin) → G-code
(minus the pen→nozzle offset). The reachable area is `bed ∩ (bed + offset)`; everything outside is
clipped away, not just greyed out.

**Stack & state**: React 18 + Vite; Zustand stores; Konva with the layer scaled so 1 unit = 1 mm.
The `document` store is authoritative; the canvas is a view (Transformer edits are read back on
transform-end). Persisted state goes through loaders that **never throw** (`Outcome` +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tibordp/kurvengefahr](https://github.com/tibordp/kurvengefahr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
