---
trigger: always_on
description: AI framework for Reinforcement Learning, Automated Planning, and Scheduling. Developed by Airbus AI Research.
---

# Scikit-Decide

AI framework for Reinforcement Learning, Automated Planning, and Scheduling. Developed by Airbus AI Research.
Repository: https://github.com/airbus/scikit-decide | Docs: https://airbus.github.io/scikit-decide/

## Quick Reference

```bash
# Build & install (editable, all extras)
uv sync --extra=all -v

# Run Python with the project
uv run python <script.py>

# Run without rebuilding C++ (when only Python changed)
uv run --no-sync python <script.py>

# Tests
uv run pytest tests                          # all tests
uv run pytest tests/path/to/test_file.py -v  # single file
uv run pytest --nbmake notebooks -v          # notebook tests

# Lint (auto-fixes files)
pre-commit run --all-files

# Docs (local dev server)
uv run yarn docs:dev
```

## Architecture

### Core Design: Mixin-Based Capability Composition

Domains and solvers are built by composing orthogonal **builder mixins** across independent dimensions. Each dimension is a single-inheritance chain from most general to most specific.

**Domain dimensions** (each is a mixin chain in `src/skdecide/builders/domain/`):

| Dimension | Chain (general → specific) | Type variable |
|---|---|---|
| Agent | `MultiAgent` → `SingleAgent` | `T_agent` |
| Concurrency | `Parallel` → `Sequential` | `T_concurrency` |
| Dynamics | `Environment` → `Simulation` → `UncertainTransitions` → `EnumerableTransitions` → `DeterministicTransitions` | — |
| Events | `Events` → `Actions` → `UnrestrictedActions` | — |
| Memory | `History` → `FiniteHistory` → `Markovian` → `Memoryless` | `T_memory` |
| Observability | `PartiallyObservable` → `TransformedObservable` → `FullyObservable` | — |
| Value | `Rewards` → `PositiveCosts` | — |
| Initialization | `Initializable` → `UncertainInitialized` → `DeterministicInitialized` | — |
| Goals | `Goals` (opt-in) | — |
| Constraints | `Constrained` (opt-in) | — |
| Renderability | `Renderable` (opt-in) | — |

**Domain presets** (common combinations in `src/skdecide/domains.py`):
- `Domain` — most general (MultiAgent, Parallel, Environment, History, PartiallyObservable, Rewards)
- `RLDomain` — RL (SingleAgent, Sequential, Environment, Actions, Markovian, TransformedObservable, Rewards)
- `MDPDomain` — MDP (adds EnumerableTransitions, FullyObservable, DeterministicInitialized)
- `GoalMDPDomain` — Goal MDP (adds Goals, PositiveCosts)
- `DeterministicPlanningDomain` — classical planning (DeterministicTransitions, Goals, PositiveCosts)
- `POMDPDomain` / `GoalPOMDPDomain` — partially observable variants
- `StatelessSimulatorDomain` — external simulation wrapper

**Solver capability mixins** (in `src/skdecide/builders/solver/`):
- Solvability: `FromInitialState`, `FromAnyState`
- Policy: `Policies` → `UncertainPolicies` → `DeterministicPolicies`
- Assessment: `Utilities`, `QValues`
- Restoration: `Restorable`
- Parallelization: `ParallelSolver`

### Method Naming Convention (Critical Pattern)

Three-tier method visibility used throughout the codebase:

```
domain.get_X()        # Public API — autocast wrapper, user calls this
domain._get_X()       # LRU-cached middle layer — calls _get_X_()
domain._get_X_()      # Implementation point — override this in subclasses
                      # Trailing _ means "result is constant, safe to cache"
```

For domain dynamics:
```
domain.step(action)          # Public — manages memory, observation, autocast
domain._state_step(action)   # Override point — implement transition logic here
domain.reset()               # Public — calls _state_reset()
domain._state_reset()        # Override point — implement initialization here
```

### Autocast System

The `@autocastable` decorator + `autocast_all()` function enable automatic type conversion between domains and solvers. When a solver declares `T_domain`, the framework:
1. Casts the domain up to the solver's abstraction level (domain → solver)
2. Casts solver outputs down to the domain's concrete types (solver → domain)

Type parameters reference `D.T_state`, `D.T_observation`, `D.T_event`, `D.T_value`, `D.T_predicate`, `D.T_info`.

### Solver-Domain Compatibility

Every solver declares a `T_domain` class attribute specifying required domain capabilities. The framework introspects this via MRO to extract builder requirements. `Solver.check_domain(domain)` validates that a domain satisfies all requirements.

Solvers receive a `domain_factory: Callable[[], Domain]` (not a domain instance) to support creating multiple copies for parallel solving.

### Plugin System (Entry Points)

Domains and solvers register via Python entry points in `pyproject.toml`:
```toml
[project.entry-points."skdecide.domains"]
MyDomain = "skdecide.hub.domain.my_domain:MyDomain [domains]"

[project.entry-points."skdecide.solvers"]
MySolver = "skdecide.hub.solver.my_solver:MySolver [solvers]"
```

Discovered at runtime via `skdecide.utils.get_registered_domains()` / `get_registered_solvers()` and `load_registered_domain(name)` / `load_registered_solver(name)`.

## Source Layout

```
src/skdecide/
├── core.py                    # Type system (D, Space, Distribution, autocast)
├── domains.py                 # Domain presets (RLDomain, MDPDomain, etc.)
├── solvers.py                 # Solver base class + domain factory wrapping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airbus/scikit-decide](https://github.com/airbus/scikit-decide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
