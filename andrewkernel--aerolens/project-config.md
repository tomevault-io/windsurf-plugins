---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->

## Project

**AeroLens**

AeroLens is an open-source scientific computing project that recovers physically meaningful aerodynamic parameters and airfoil geometry from schlieren-style flow images. It composes geometry, flow simulation, differentiable rendering, and inverse optimization as independently reproducible Tesseracts while preserving gradients across every component boundary.

The primary competition category is differentiable graphics and rendering, with inverse design and shape optimization as a deliberate secondary category. The result must be a credible research artifact rather than a visual toy: gradients, recovered parameters, solver outputs, and external validation must all be inspectable.

**Core Value:** A reviewer can differentiate an image-space loss all the way back to aerodynamic design parameters, verify those gradients independently, and reproduce an optimization that recovers a physically correct solution.

### Constraints

- **Repository**: Public `andrewkernel/aerolens` with an Apache-2.0-compatible dependency and licensing story — required for open scientific reuse and prior competition eligibility.
- **Reproducibility**: A CPU reference path must run without proprietary services; acceleration may be optional — reviewers need a dependable baseline.
- **Scientific integrity**: Gradient checks, parameter-recovery error, and independent forward validation are release gates — attractive images cannot substitute for evidence.
- **Architecture**: At least two meaningful Tesseract boundaries must remain visible in the final pipeline — the project must demonstrate composition, not merely place a monolith in one container.
- **Scope**: Start with a small parameter space and deterministic inverse problem, then add fidelity only when a measured failure demands it — keeps scientific conclusions attributable.
- **Communication**: Every headline result must be regenerable by a documented command and backed by machine-readable metrics — prevents hand-curated demos.

<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->

## Technology Stack

## Recommendation

## Runtime

| Layer | Choice | Why |
|---|---|---|
| Language | Python 3.12 baseline; support 3.11-3.14 where dependencies permit | Current Tesseract Core requires Python >=3.10,<3.15; JAX-Fluids requires >=3.11. Python 3.12 is the least surprising CI/container target. |
| Array/AD | JAX, pinned by lock/constraints | Required by JAX-Fluids and composes naturally with Tesseract-JAX. Native Windows is CPU-only; Linux CI/container is the GPU path. |
| Compressible CFD | JAX-Fluids at commit `af2b7cf51c2f9c61e4ec732491e21a64511c57d8` initially | The live 2026-08-05 source contains a checkpointed differentiable `feed_forward` path, NACA/immersed-solid examples, WENO5-Z, HLLC, positivity limiting, and Cartesian level sets. Pinning a commit avoids an unversioned moving `main`. |
| Component runtime | `tesseract-core>=1.10,<2` | Provides schema validation, container build/serve/run, VJP/JVP/Jacobian endpoints, and gradient checking. |
| Composition | `tesseract-jax>=0.4,<0.5` | Makes a served Tesseract a JAX primitive and supports reverse-mode composition across container boundaries. Version 0.4 supports modern JAX and Python 3.14. |
| Optimization | Optax | Already a JAX-Fluids dependency; Adam provides a robust first optimizer without another optimization framework. |
| Independent validator | SU2 8.5.0 executable/container, invoked out of process | Mature Euler solver and public NACA0012 Mach 0.8/AoA 1.25 case. It must not share AeroLens's forward code or gradient implementation. |
| Figures | Matplotlib | Already a JAX-Fluids dependency and sufficient for publication figures, PNGs, and animations. |
| CLI/config | `argparse`, `json`, `dataclasses`, `pathlib` | Standard library is enough for deterministic experiment commands. |

## Packaging and quality

- Use one root `pyproject.toml` with `setuptools` and a `src/aerolens` package.
- Keep Tesseract component requirements beside each component; do not import Tesseract runtime into the mathematical core.
- Use `pytest` for unit, gradient, integration, and regression tests.
- Use Ruff as the single formatting/lint tool. Do not add a second formatter or linter.
- Use GitHub Actions on Ubuntu for the CPU-small suite and Docker build smoke tests. Add GPU jobs only if a runner is actually available.
- Store only small fixtures and headline result summaries in Git. Regenerate large fields and images through documented commands.

## Compatibility constraints

## Rejected alternatives

| Alternative | Reason not selected |
|---|---|
| Write a custom production CFD solver | High scientific and schedule risk; it would duplicate JAX-Fluids instead of demonstrating tool composition. |
| Use JAX-CFD | Official repository is no longer maintained and focuses on incompressible flow, while the target evidence depends on shocks. |
| Remesh with Gmsh in the optimization loop | Mesh topology changes make geometry derivatives brittle. A Cartesian level-set interface gives a smaller, more differentiable first system. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrewkernel/aerolens](https://github.com/andrewkernel/aerolens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
