---
trigger: always_on
description: Platform-wide Sienna conventions (performance, type stability, formatter, environments, code style) live in `.claude/Sienna.md` — read it too. This file is repo-specific and does not restate them.
---

# PowerSystems.jl — Claude Guide

Platform-wide Sienna conventions (performance, type stability, formatter, environments, code style) live in `.claude/Sienna.md` — read it too. This file is repo-specific and does not restate them.

## Purpose & place in the stack

PowerSystems.jl is the **central data-model package** of Sienna. It defines the `System`
container and the full hierarchy of power-system component types (generators, loads,
branches, storage, services, dynamic models), their operational cost structures, time
series, and unit-system handling. It is not a solver — it is the typed data layer that
nearly every other Sienna package builds on.

- **Built on** InfrastructureSystems.jl (IS): `System.data` is an `IS.SystemData`; the
  component container, time-series store, serialization, validation, and struct-generation
  machinery all come from IS. PSY imports IS as `const IS` (`import InfrastructureSystems as IS`).
- **Consumed by** essentially the whole stack: PowerSimulations.jl, PowerFlows.jl,
  PowerNetworkMatrices.jl, PowerSimulationsDynamics.jl, PowerSystemCaseBuilder.jl (PSCB),
  the parsers, and investment/portfolio packages. Any change to a public type, accessor, or
  serialization format ripples downstream — weigh that before changing them.

Current clone: `version = 5.11.1`, `InfrastructureSystems = "^3.6"`, `julia = "^1.10"`.

## src/ layout

Main module: `src/PowerSystems.jl` — all `export`s (~560), all IS imports/re-exports, and
the `include` order live here. **Respect include order**: a type/const must be defined in a
file included before any file that references it. Auto-generated structs are pulled in last
via `include("models/generated/includes.jl")`.

- `base.jl` — the `System` type and its core methods (`add_component!`, `get_component(s)`,
  `remove_component!`, `set_units_base_system!`, etc.).
- `definitions.jl` — core enums and constants.
- `component_selector.jl` / `component_selector_interface.jl`, `get_components_interface.jl` —
  component selection/query surface.
- `subsystems.jl`, `contingencies.jl`, `outages.jl`, `emissions_data.jl`,
  `impedance_correction.jl`, `plant_attribute.jl`, `data_format_conversions.jl`.
- `deprecated.jl` — deprecation shims (see policy note below).
- `models/` — component model logic, hand-written behavior layered on top of generated
  structs: `components.jl` (base + units/conversion hooks), `branches.jl`, `generation.jl`,
  `storage.jl`, `loads.jl`, `topological_elements.jl`, `reserves.jl`, `services.jl`,
  `static_models.jl`, `injection.jl`, `static_injection_subsystem.jl`, `HybridSystem.jl`,
  dynamic models (`dynamic_generator.jl`, `dynamic_inverter.jl`, `dynamic_branch.jl`,
  `dynamic_loads.jl`, `dynamic_machines.jl`, the `*Exponential.jl`/`*Quadratic.jl` machine
  files, `OuterControl.jl`), `cost_function_timeseries.jl`, `serialization.jl`,
  `supplemental_constructors.jl`, `supplemental_accessors.jl`, `supplemental_setters.jl`.
  - `models/cost_functions/` — operational cost types (ThermalGenerationCost, StorageCost,
    MarketBidCost, CostCurve/FuelCurve value curves, etc.).
  - `models/generated/` — **AUTO-GENERATED structs. Do not edit by hand.** (see below)
- `parsers/` — data import: `power_system_table_data.jl` (CSV/table, incl. RTS-GMLC),
  `power_models_data.jl` (PowerModels/MATPOWER), `psse_dynamic_data.jl`, with `pm_io/`
  (matpower, psse, pti) and `im_io/`.
- `utils/` — `generate_struct_files.jl` (thin wrapper over IS struct generation),
  `conversion.jl`, `logging.jl`, `print.jl`/`print_pt_v2.jl`/`print_pt_v3.jl` (display),
  and `IO/` data-validation checks.
- `descriptors/` — `power_system_structs.json` (struct definitions, source of truth for
  generated code) and `power_system_inputs.json` (parser input specs).

## Auto-generated structs — never edit by hand

The ~141 files in `src/models/generated/` (one per component type) are generated from
`src/descriptors/power_system_structs.json`. Each has its own typed fields, docstring,
constructor(s), and accessors. **Do not edit anything in `src/models/generated/`** — your
changes will be overwritten and CI will diff against the descriptor.

To change a struct: edit `src/descriptors/power_system_structs.json`, then regenerate:

```sh
julia --project=test -e 'using PowerSystems; PowerSystems.generate_struct_files(PowerSystems.read_struct_functions(...))'
```

In practice the supported path is via IS's generator. The descriptor is the source of
truth; the generation itself is `IS.generate_struct_files` wrapped by
`src/utils/generate_struct_files.jl`. `test/test_generate_structs.jl` exercises it — run
that after touching the descriptor and check the `models/generated/` diff is intentional.
Hand-written behavior (validation, custom show, conversions, extra constructors) belongs in
the `src/models/*.jl` files, never in the generated file.

## Core abstractions (quick map)

- `System` — top-level container; holds components + time series in `IS.SystemData`,
  `frequency`, `bus_numbers`, `runchecks`, `units_settings`.
- Component hierarchy: `Topology` (`ACBus`/`DCBus`, `Arc`, `Area`, `LoadZone`), `Device`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sienna-Platform/PowerSystems.jl](https://github.com/Sienna-Platform/PowerSystems.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
