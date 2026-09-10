---
trigger: always_on
description: Platform-wide Sienna conventions (performance, type stability, formatter, environments, code style) live in `.claude/Sienna.md` — read it too. This file is repo-specific and does not restate them.
---

# PowerSimulationsDynamics.jl — Claude Guide

Platform-wide Sienna conventions (performance, type stability, formatter, environments, code style) live in `.claude/Sienna.md` — read it too. This file is repo-specific and does not restate them.

## Purpose & place in the stack

PowerSimulationsDynamics.jl (PSID) runs **time-domain dynamic simulations** of power systems. It formulates and solves the DAE/ODE system describing generator and inverter dynamic device models, using the SciML stack. This is **not** a JuMP optimization package — there is no optimization-container or constraint-building layer.

PSID consumes `PowerSystems.jl` (PSY) data: dynamic-injector device models (`DynamicGenerator`, `DynamicInverter`, their machines/AVRs/governors/PSS, inverter converters/filters/controls) and the static network. It uses `PowerFlows.jl` (PF) to get the steady-state operating point that seeds initialization, and `PowerNetworkMatrices.jl` (PNM) for network matrices. Verified deps in `Project.toml`: `PowerSystems` (compat 5), `PowerFlows` (^0.16), `PowerNetworkMatrices` (^0.20), `InfrastructureSystems` (3), plus the numerics stack `SciMLBase` (2), `NLsolve` (4), `ForwardDiff` (1), `SparseArrays`, `LinearAlgebra`, `DataStructures`, `FastClosures`, `TimerOutputs`. Note: `SciMLBase` is a dep, but the concrete solvers (`Sundials`, `OrdinaryDiffEq`, `DelayDiffEq`) are supplied by the caller / test env, not by PSID itself.

## Architecture & src/ layout

Include order is authoritative — see `src/PowerSimulationsDynamics.jl`. New constants/types must precede their uses.

- `src/base/` — the simulation machinery.
  - `definitions.jl`, `ports.jl`, `bus_categories.jl` — constants, port mappings, bus classification.
  - `device_wrapper.jl` — `DynamicWrapper{T<:PSY.DynamicInjection}`: wraps each PSY dynamic device with its `ix_range` / `ode_range` (slices into the global state vector), `global_index` (`ImmutableDict{Symbol,Int}` mapping state names → global indices), inner-component refs, and connection data. `branch_wrapper.jl` — analogous wrapper for dynamic branches.
  - `simulation_model.jl` — model-variation singleton types: `ResidualModel`, `MassMatrixModel` (both `<: SimulationModel`), and `NoDelays` / `HasDelays` (`<: DelayModel`).
  - `simulation_inputs.jl` — `SimulationInputs`: assembled state vector, mass matrix, wrappers, index ranges, `get_setpoints`. `system_model.jl` — `SystemModel{T<:SimulationModel, D<:DelayModel, C<:Cache}`, the callable passed to the solver.
  - `perturbations.jl` — perturbation/event types.
  - `mass_matrix.jl` — builds the singular DAE mass matrix. `jacobian.jl` — `JacobianFunctionWrapper` and `get_jacobian`, built with `ForwardDiff`. `caches.jl` — dual-number-aware caches for ForwardDiff.
  - `simulation.jl` — `Simulation` (a `mutable struct Simulation{T<:SimulationModel}`), `execute!`, `read_results`, `get_setpoints`.
  - `nlsolve_wrapper.jl`, `simulation_initialization.jl` — steady-state solve (see below). `small_signal.jl` — `small_signal_analysis`. `simulation_results.jl` — `SimulationResults`, `get_state_series`. `model_validation.jl`.
- `src/initialization/` — `init_device.jl` plus per-component routines under `generator_components/` (machine, shaft, avr, tg, pss) and `inverter_components/` (filter, DCside, converter, frequency_estimator, inner, outer). Each solves for that component's initial states given the power-flow operating point.
- `src/models/` — the device dynamic-model equations. Common: `branch.jl`, `device.jl`, `network_model.jl`, `dynline_model.jl`, `ref_transformations.jl`, `common_controls.jl`. `generator_models/` (machine, pss, avr, tg, shaft) and `inverter_models/` (DCside, filter, frequency_estimator, outer_control, inner_control, converter, output_current_limiter). `load_models.jl`, `source_models.jl`, `saturation_models.jl`, and `system.jl` which holds the top-level residual/MM callables.
- `src/post_processing/` — `get_*_series` accessors, branch-flow series, source/load/generator post-processing, `read_initial_conditions`, `show_states_initial_value`.
- `src/utils/` — `psy_utils.jl`, `pf_utils.jl`, `immutable_dicts.jl`, `print.jl`, `kwargs_check.jl`, `logging.jl`.

## Key public API / entry points

Verified exports in `src/PowerSimulationsDynamics.jl`:

- Build & run: `Simulation`, `Simulation!`, `execute!`, `SimulationResults`, `read_results`.
- Model selection (first positional arg to `Simulation`): `ResidualModel`, `MassMatrixModel`.
- Frequency reference: `ReferenceBus`, `ConstantFrequency`.
- Perturbations: `NetworkSwitch`, `ControlReferenceChange`, `BranchTrip`, `BranchImpedanceChange`, `SourceBusVoltageChange`, `GeneratorTrip`, `LoadTrip`, `LoadChange`, `PerturbState`. (`BusTrip` is defined but commented out of exports.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sienna-Platform/PowerSimulationsDynamics.jl](https://github.com/Sienna-Platform/PowerSimulationsDynamics.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
