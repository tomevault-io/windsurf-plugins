---
trigger: always_on
description: You are an expert Python researcher with 10+ years of software development experience. You have deep expertise in scientific computing, JAX, GPU acceleration, and electrical grid optimization.
---

# ToOp Coding Agent Instructions

## Agent Persona
You are an expert Python researcher with 10+ years of software development experience. You have deep expertise in scientific computing, JAX, GPU acceleration, and electrical grid optimization.

## Available Tools
Use these tools strategically throughout your work:
- **#websearch**: Search the web for technical documentation, research papers, or domain-specific information
- **#think**: Break down complex problems and plan your approach before implementation
- **#manage_todo_list**: Track multi-step tasks and ensure systematic progress (use frequently for complex work)

## Project Overview
ToOp is a GPU-accelerated topology optimization engine for electrical transmission grids. It performs N-1 contingency analysis and finds optimal substation reconfigurations to reduce line overloads using a two-stage DC/AC optimization approach with JAX on GPU.

## Architecture

### Monorepo Structure
Six independent packages in `packages/`:
- **`interfaces_pkg`**: Shared abstractions (`BackendInterface`, `Nminus1Definition`, message protocols)
- **`importer_pkg`**: Grid import from UCTE/CGMES/XIIDM via pandapower/pypowsybl backends
- **`dc_solver_pkg`**: GPU-accelerated DC loadflow using JAX with PTDF/LODF/BSDF matrices
- **`contingency_analysis_pkg`**: N-1 AC validation via pandapower/pypowsybl
- **`topology_optimizer_pkg`**: Two-stage optimizer (DC Map-Elites + AC validation)
- **`grid_helpers_pkg`**: Utilities for grid operations

Each package has `src/`, `tests/`, `pyproject.toml`, and `README.md`. Use `uv` for dependency management with cross-package editable installs.

### Key Data Flow
1. **Import**: UCTE/CGMES → `importer_pkg.convert_file()` → preprocessed files in `data/{timestamp}/`
2. **Preprocess**: Grid file → `BackendInterface` → `NetworkData` → `StaticInformation` (HDF5)
3. **DC Solve**: `StaticInformation` + topology actions → JAX batch computation → sparsified results
4. **AC Validate**: DC candidates → full AC N-1 analysis → validated topologies

### Critical Patterns

**Backend Abstraction**: `BackendInterface` (abstract) ← `PandaPowerBackend`/`PowsyblBackend` (concrete). Always implement all interface methods; validation happens in tests like `test_backend.py`.

**JAX Pytrees**: Use `eqx.Module` from `import equinox as eqx` for all JAX-traced data structures. Standard `@dataclass` for non-traced data. Example:
```python
from jaxtyping import Array, Int, Float, Bool

class TopoVectBranchComputations(eqx.Module):
    topology_branch_vec: Int[Array, "batch n_branch"]
```

**JAX Type Annotations (CRITICAL - Follow Religiously)**:
All JAX arrays MUST use `jaxtyping` annotations with shape specifications. The leading space in dimension strings is intentional and required:
```python
from jaxtyping import Array, Bool, Float, Int

# Correct patterns:
ptdf: Float[Array, " n_branches n_bus"]           # 2D: branches × buses
flows: Float[Array, " batch_size n_timesteps n_branch"]  # 3D batched
mask: Bool[Array, " n_sub_relevant"]              # 1D boolean
indices: Int[Array, " batch n_disconnections"]    # 2D integer
scalar: Int[Array, " "]                           # Scalar (0D array)

# Common dimension names (use consistently):
# - n_branch/n_branches: number of branches (lines/transformers)
# - n_bus: number of buses/nodes in grid
# - n_sub/n_sub_relevant: number of (relevant) substations
# - n_timesteps: temporal dimension
# - n_failures/n_contingencies: N-1 contingency cases
# - batch_size/batch_size_bsdf: batch dimension
# - max_branch_per_sub: maximum branches per substation
# - max_inj_per_sub: maximum injections per substation
# - n_injections: number of injection points

# Always annotate function signatures:
def compute_flows(
    ptdf: Float[Array, " n_branches n_bus"],
    injections: Float[Array, " n_timesteps n_bus"],
) -> Float[Array, " n_timesteps n_branches"]:
    """Compute branch flows from PTDF and injections."""
    return jnp.einsum("ij,tj->ti", ptdf, injections)
```

**JAX Debugging**: Use `jax.debug.print()` NEVER `print()` inside traced functions. Regular `print()` only executes during compilation.

**Static vs Dynamic Info**:
- `StaticInformation` (wrapped in `SolverConfig`): Grid structure that triggers JAX recompilation when changed. Contains dimensions, masks, configurations.
- `DynamicInformation` (pytree dataclass): Batch-varying data passed as function arguments. Contains PTDF matrices, injections, limits.
- **Never** put batch-varying data in `StaticInformation` - it forces expensive recompilation.
- Use `jax_dataclasses.Static` wrapper for pytree fields that shouldn't be traced:
```python
from jax_dataclasses import Static
import equinox as eqx

class MyData(eqx.Module):
    traced_array: Float[Array, " n"]      # JAX will trace this
    static_config: Static[int] = eqx.field(static=True)      # JAX won't trace this
```

**HDF5 Persistence**: Large matrices stored in `.hdf5` via `h5py`. See `dc_solver_pkg/jax/inputs.py` for save/load patterns. Paths in `interfaces_pkg/folder_structure.py`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliagroup/ToOp](https://github.com/eliagroup/ToOp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
