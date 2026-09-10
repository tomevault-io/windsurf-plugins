---
trigger: always_on
description: Operating guide for AI coding agents (Claude Code) working in **Dionysos.jl**. Read this before
---

# CLAUDE.md

Operating guide for AI coding agents (Claude Code) working in **Dionysos.jl**. Read this before
making changes: it explains what the project is, how the code is organized, the one architectural
contract that matters most (the solver interface), the Julia conventions you must imitate, the exact
commands to run, and the repo-specific traps to avoid.

---

## 1. What Dionysos is & why it exists

Dionysos is a **framework for symbolic (abstraction-based) control**. It is the software of the ERC
project *Learning to Control* (L2C).

**The mission.** Today, controlling a complex system usually means a team of expert engineers, each
with deep knowledge of the plant, hand-crafting an ad hoc controller over months at significant cost.
Dionysos aims to change that paradigm into an automatic pipeline:

> **describe the system → select the problem (specification) → pick a solver → Dionysos automatically
> computes a controller with a certificate (formal guarantee).**

The goal is to make correct-by-construction control synthesis *accessible* — including to small
companies that have no dedicated controls/IT team — and to drastically cut controller-design time.

**Symbolic control 101.** The system is *abstracted* into a finite-state automaton (a **symbolic
model**) by discretizing its variables. Working on this finite object lets a computer synthesize
controllers systematically, even for complex specifications (e.g. LTL), using graph algorithms
(Dijkstra, A\*, fixed-point iterations). The price:

- **Curse of dimensionality** — the number of abstract states grows exponentially with the state
  dimension.
- **Non-determinism from discretization** — over-approximating a cell's dynamics introduces spurious
  transitions, which can make the abstract problem *infeasible* even when the concrete one is not.

The mitigation, and a core research direction of the toolbox, is **smart / lazy abstractions**: instead
of an a-priori uniform grid, co-design the abstraction and the controller and compute only the part of
the abstraction that is actually needed.

**Framing for an agent.** Dionysos is an **ecosystem, not one algorithm**. Its value is a *common
interface* that lets you swap the solver and then test / compare / benchmark algorithms on the same
control problem. Preserving that interface is the single most important thing when extending the code.

Background reading: [docs/src/manual/overview.md](docs/src/manual/overview.md) and
[docs/src/manual/abstraction-based-control.md](docs/src/manual/abstraction-based-control.md).

---

## 2. The framework in one paragraph

A control problem is a pair `(𝒮, Σ)`: a **system** `𝒮` (a
[`MathematicalSystems`](https://juliareach.github.io/MathematicalSystems.jl) or
[`HybridSystems`](https://github.com/blegat/HybridSystems.jl) object) plus a **specification** `Σ` (a
`Dionysos.Problem.ProblemType`). It is solved by a **solver** `𝒪` that is a
`MathOptInterface.AbstractOptimizer`, driven through the JuMP/MOI interface. This MOI contract is the
architectural keystone: **every algorithm is a swappable `Optimizer`**, so a task can be re-solved,
compared, and benchmarked by swapping the optimizer rather than rewriting the model. Abstraction-based
solvers turn the infinite-state system into a finite automaton via discretization, synthesize a
correct-by-construction controller on it, then *concretize* it back to the original system.

---

## 3. Repository map (root level)

| Path | What it is |
| :--- | :--- |
| [`src/`](src/) | The `Dionysos` package (all library code). |
| [`ext/`](ext/) | Package extensions — optional-dependency glue (Plots, Symbolics, Spot, CSV, RigidBodyDynamics). |
| [`test/`](test/) | Test suite; mirrors `src/` layout. Entry point [`test/runtests.jl`](test/runtests.jl). |
| [`docs/`](docs/) | Documenter.jl site + Literate.jl examples. Build script [`docs/make.jl`](docs/make.jl). |
| [`problems/`](problems/) | Reusable **benchmark problem library** (e.g. path planning, DC-DC, pendulum), one folder per problem. |
| [`examples/`](examples/) (root) | **Runnable example drivers** (user-facing), one folder per problem, mirroring `problems/` — **not** library code. |
| [`research/`](research/) (root) | Our **paper / experiment sims** (`CDC2024/`, `BisimulationQuotient/` = HSCC 2027, …), superseded work under `research/Deprecated/`. |
| [`bench/`](bench/) | Benchmarks (BenchmarkTools). |
| `control_server/`, `BipedRobot/`, `paper/`, `assets/` | Auxiliary app, robot demo, paper artifacts, images. |

> ⚠️ Top-level `examples/` and `research/` hold runnable driver scripts (they were a single
> `scripts/` folder before, and `utils/` before that) — **not** library code. Don't confuse them with
> the `src/utils/` module (the `Utils` library).

---

## 4. Core architecture — six library modules + the front-end

Top-level module [`src/Dionysos.jl`](src/Dionysos.jl) includes six library submodules **in
dependency order**, then the JuMP front-end on top of them:

```
Utils → System → Problem → Mapping → Symbolic → Optim → Wrapper
```

Reuse these **standard aliases** everywhere (they are established at the top of each module — match
them exactly):

```julia
const UT = Utils    # Dionysos.Utils
const ST = System   # Dionysos.System

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dionysos-dev/Dionysos.jl](https://github.com/dionysos-dev/Dionysos.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
