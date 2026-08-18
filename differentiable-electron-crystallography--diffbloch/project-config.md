---
trigger: always_on
description: Provides a value to the forward model?  -> component        (engine/components.py)
---

# AGENTS.md for diffBloch

Guidance for coding agents working on diffBloch. It records the architecture, invariants, and principles that constrain every change, plus the way to derive the right implementation shape for a new feature. Read it before touching `src/diffBloch/`. This document stands alone: do not depend on files outside the repository as implementation authority.

When something is unclear, ask before changing architecture. When running unattended, choose the smallest reasonable interpretation, proceed, and state the assumption in your summary.

## What diffBloch is

diffBloch refines crystal structures against rotating-stage 3D electron-diffraction data: continuous-rotation 3DED, a sequence of diffraction frames collected as the crystal is tilted/rocked through reciprocal space around a goniometer axis, reduced upstream by PETS2 into `.cif_pets` experimental data. Because the diffraction is dynamical (multiple-scattering), diffBloch uses a differentiable Bloch-wave simulation rather than a kinematical approximation.

The object of value is small: a differentiable map from a handful of structural parameters (atom positions, ADPs, occupancies, structure factors) to simulated intensities and a single scalar R-loss. Gradients of that loss update the selected trainable parameters. Everything else in the package prepares inputs for that map, runs the optimizer around it, checkpoints expensive setup, or reports progress — around the numerical core, never inside it.

`infer` means simulate and score a settled `Plan` without updating parameters. `refine` means run optimizer steps that update selected trainable parameters.

## Architecture: functional core, imperative shell

The design is a pure functional core wrapped in a thin imperative shell. The core is deterministic mathematics; the shell does I/O, optimization, device placement, checkpointing, and logging. Keep that boundary sharp: it is what makes runs inspectable, reproducible at the artifact boundary, and attributable.

Preserve this system shape:

```text
.cif + .cif_pets + experiment.yaml
  -> typed IO / config boundary
  -> Plan-building preprocess shell
  -> deterministic Bloch-wave core
  -> refinement engine / objective
  -> quarantined optimizer / app / logging shell
```

Three quick smell-tests for any change:

- A change that makes a lower layer know about a higher layer is suspect.
- A change that hides a result-changing input outside config/provenance is suspect.
- A change that mutates caller-owned scientific values in place is suspect.

### Layers and dependency direction

Dependencies point one way. A layer may import from layers below it, never from orchestration above.

| Layer | Role | Hard rule |
|---|---|---|
| `io` | Parse CIF/PETS into validated typed records. | Parser details stop at typed records; numerical code never consumes raw parser objects. |
| `params` | Raw refinable parameters and crystallographic constraints. | `RefinableParams -> constrain -> PhysicalState`; no optimizer state here. |
| `specs` | Frozen value-types for preprocess/scientific knobs. | Defaults and validation live here. |
| `core` | Deterministic crystallographic and Bloch-wave kernels. | Tensor/value in, tensor/value out; no parser, optimizer, vendor SDK, filesystem, or app imports. |
| `engine` | Combine `Plan` + parameters, simulate, score, refine. | Forward/objective code stays pure; optimizer mutation is quarantined in the refinement loop. |
| `preprocess` | Build and improve the immutable `Plan`. | Public shape is `Plan -> Plan`; steps return new values, never mutations. |
| `config` | Validate experiment settings and lock identity. | Pydantic boundary, `extra="forbid"`; no Hydra/`DictConfig` reaches the core. |
| `observability` | Typed event values and the logger protocol. | Emit event data; backend I/O lives at the app/logger boundary. |
| `app` | CLI and default orchestration around reusable APIs. | Compose public APIs; do not hide science-only behaviour here. |

`io`, `params`, `specs` sit below `core`; `core -> engine -> preprocess`; `config` and `app` wire the lower layers at the top. The engine never imports `preprocess`.

### "Core" means the functional core

`core` is the `src/diffBloch/core/` layer — pure tensor-in/tensor-out kernels from structural parameters to simulated intensities and the R-loss — not "whatever is important." Keep this wording precise in prose.

### What is pure and what has effects

- **Pure:** every kernel in `core/*`, `params.constrain`, all `specs` value-types, the objective/simulation in `engine/forward.py`, and every `Plan -> Plan` step in `preprocess/steps/`.
- **The one quarantined stateful corner:** the `torch.optim` loop in `engine/refine.py`. It clones selected trainable fields into fresh leaves and never mutates the caller's parameters. `core/` never imports `torch.optim`.
- **Effects/I/O:** `io` (file reads), `config` (YAML/lock/manifest, git/version stamps), `preprocess/serialize.py` (`Plan` checkpoint read/write), and `app` (CLI, orchestration, logger backends).

### Invariants of the Bloch-wave core

These make the numerical core trustworthy. Treat a change that violates one as a defect even if the forward value looks right.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Differentiable-Electron-Crystallography/diffBloch](https://github.com/Differentiable-Electron-Crystallography/diffBloch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
