---
trigger: always_on
description: Platform-wide Sienna conventions (performance, type stability, formatter, environments, code style) live in `.claude/Sienna.md` — read it too. This file is repo-specific and does not restate them.
---

# PowerSimulations.jl — Claude Guide

Platform-wide Sienna conventions (performance, type stability, formatter, environments, code style) live in `.claude/Sienna.md` — read it too. This file is repo-specific and does not restate them.

Power-system optimization and simulation framework. Builds and solves large-scale optimization (JuMP) problems for operations modeling across multiple time scales (planning, day-ahead, real-time). Package version `0.36.x`; Julia compat `^1.10`.

## Where it sits in the Sienna stack

- **Upstream deps:** `InfrastructureSystems` (IS — container, time series, serialization, `@assert_op`), `PowerSystems` (PSY — `System`, component/device/service types), `PowerNetworkMatrices` (PNM — PTDF/LODF/`VirtualPTDF`/`VirtualMODF`, network reduction), `PowerFlows` (PF — power-flow-in-the-loop), `PowerModels` (PM — AC/DC network formulation abstract types), `JuMP`/`MathOptInterface` (the optimizer interface), `HDF5` (results store).
- **Downstream:** PowerAnalytics / PowerGraphics / PowerSimulationsDynamics consume PSI results and `System` copies. Changes to results storage and serialization have downstream blast radius.
- `InfrastructureOptimizationModels` and `PowerOperationsModels` are NOT current dependencies (not in `Project.toml`) — do not assume coupling to them.

### Version pairing (non-obvious, keep coupled)
- PF and PNM are version-coupled: do not mix incompatible majors. Current compat: `PowerFlows ^0.21.1`, `PowerNetworkMatrices ^0.24`, `PowerSystems ^5.11`, `PowerModels ^0.21.5`, `JuMP ^1.28`, `InfrastructureSystems ^3.5`. When bumping PF, bump PNM in lockstep and re-run the full suite.

## Core Architecture

### Operation Models
Central abstraction `OperationModel`, two concrete types:
- **`DecisionModel{M <: DecisionProblem}`** — optimization over a horizon (e.g. 24h UC, 1h ED). Holds a `ProblemTemplate`, an `OptimizationContainer` (JuMP wrapper), and a PSY `System`.
- **`EmulationModel{M <: EmulationProblem}`** — single-time-step real-time emulation (AGC, reserve deployment).

Built-in problem types: `GenericOpProblem`, `UnitCommitmentProblem`, `EconomicDispatchProblem`, `AGCReserveDeployment`.

### ProblemTemplate
Defines network representation + device/service/event formulations:
```julia
template = ProblemTemplate(NetworkModel(CopperPlatePowerModel))
set_device_model!(template, ThermalStandard, ThermalBasicUnitCommitment)
set_service_model!(template, VariableReserve{ReserveUp}, RangeReserve)
```

### Device, Service, Network, Event Models
- **`DeviceModel{D <: PSY.Device, B <: AbstractDeviceFormulation}`** — binds a device type to a formulation; carries feedforwards, time-series mappings, attributes.
- **`ServiceModel{D <: PSY.Service, B <: AbstractServiceFormulation}`** — same pattern for ancillary services.
- **`NetworkModel{T <: PM.AbstractPowerModel}`** — power-flow formulation: `CopperPlatePowerModel`, `PTDFPowerModel`/`AbstractPTDFModel`, `AreaBalancePowerModel`, `AreaPTDFPowerModel`, full AC/DC from PowerModels.
- **`EventModel`** (`set_event_model!`) — outage/contingency events; see `src/core/event_model.jl`, `event_keys.jl`, and the `contingency_model/` directory.

### Formulation Hierarchy (formulation type controls what is built)
- **Thermal:** `ThermalBasicUnitCommitment`, `ThermalStandardUnitCommitment`, `ThermalCompactUnitCommitment`, `ThermalBasicDispatch`, etc. (UC = binary on/off + min up/down; dispatch = continuous only)
- **Renewable:** `RenewableFullDispatch`, `RenewableConstantPowerFactor`
- **Load:** `StaticPowerLoad`, `PowerLoadInterruption`, `PowerLoadDispatch`
- **Storage:** `BookKeeping`, `BatteryAncillaryServices`
- **Branches:** `StaticBranch`, `StaticBranchBounds`, `StaticBranchUnbounded`, `HVDCTwoTerminalDispatch`, `SecurityConstrainedStaticBranch`

### OptimizationContainer
Wraps the JuMP model; holds Variables, Constraints, Parameters (time series + feedforward), Expressions (shared nodal/area balance), and the objective, all in typed containers keyed by `OptimizationContainerKey`.

## Simulation Architecture

- **`Simulation`** orchestrates multi-model runs across time.
- **`SimulationModels`** — vector of `DecisionModel`s + optional `EmulationModel`.
- **`SimulationSequence`** — execution order, feedforward connections, initial-condition chronologies.
- **`SimulationState`** — `current_time`, `last_decision_model`, `decision_states::DatasetContainer`, `system_states::DatasetContainer`.

### Feedforwards
`UpperBoundFeedforward`, `LowerBoundFeedforward`, `SemiContinuousFeedforward`, `FixValueFeedforward` — parameterize a downstream model with upstream results (source model, source variable, affected target component/variable).

### Initial Conditions
`DevicePower`, `DeviceStatus`, `InitialTimeDurationOn/Off`, `InitialEnergyLevel`, `AreaControlError`. Chronologies: `InterProblemChronology` (other model's results) / `IntraProblemChronology` (same model's previous solve).

### Execution loop
read state → update feedforward params → update initial conditions → `JuMP.optimize!` → write results to `SimulationState` + store (HDF5 or in-memory) → advance.

## src/ layout

```
src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sienna-Platform/PowerSimulations.jl](https://github.com/Sienna-Platform/PowerSimulations.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
