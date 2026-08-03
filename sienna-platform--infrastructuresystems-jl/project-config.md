---
trigger: always_on
description: Platform-wide Sienna conventions (performance, type stability, formatter, environments, code style) live in `.claude/Sienna.md` — read it too. This file is repo-specific and does not restate them.
---

# InfrastructureSystems.jl — Claude Guide

Platform-wide Sienna conventions (performance, type stability, formatter, environments, code style) live in `.claude/Sienna.md` — read it too. This file is repo-specific and does not restate them.

## Purpose & foundational role

IS is the **foundational utility library of the Sienna stack** — almost every other Sienna package depends on it and imports its core machinery rather than reimplementing it. It provides the shared, performance-critical building blocks for infrastructure data models: component containers, the time-series subsystem, serialization, system-data management, the `@assert_op` macro, struct auto-generation, logging/recorder utilities, and the abstract optimization-container types. It is *domain-agnostic* (no power-systems concepts live here — those are in PowerSystems.jl).

Consumed by: PowerSystems.jl, PowerSimulations.jl, PowerSimulationsDynamics.jl, PowerNetworkMatrices.jl, PowerFlows.jl, PowerSystemCaseBuilder.jl, and the IOM/POM packages. **Changes here ripple platform-wide** — assume any signature/behavior change can break downstream packages, and weigh that before changing public surface.

- `version` is `3.6.0`. **Do not bump it** during dev work (even breaking-change work). A local version ahead of the registry breaks cross-package `Pkg.develop`/test resolution for the whole stack; release versions are set at publish time. If a bump reappears in the working tree, revert it. Same rule applies to other Sienna packages worked on alongside IS.
- Julia compat: `julia = "^1.10"` (Project.toml). The README's "1.6 or higher" line is stale prose — trust Project.toml.
- Default branch is **`main`**, not `master`. The harness may report `master` at session start — that is wrong here; `git ls-remote origin` shows only `main`. Use `main` for all PRs/diffs/base refs.

## Architecture & `src/` layout

Top-level `src/` files (flat, included from `src/InfrastructureSystems.jl` — respect include order when adding types/constants):

- `InfrastructureSystems.jl` — main module, includes, and the (small) export list.
- **Component model:** `component.jl`, `components.jl`, `component_container.jl`, `component_uuids.jl`, `containers.jl`, `internal.jl` — abstract types `InfrastructureSystemsComponent`, `InfrastructureSystemsType`, `InfrastructureSystemsContainer` and the storage backing `SystemData`.
- **System data:** `system_data.jl` — `SystemData`, the central container tying components, time series, supplemental attributes, and subsystems together. Also `subsystems.jl`, `supplemental_attribute_*.jl`, `geographic_supplemental_attribute.jl`, `validation.jl`.
- **Time series:** the largest subsystem. `abstract_time_series.jl`, `static_time_series.jl`, `forecasts.jl`, `deterministic*.jl`, `probabilistic.jl`, `scenarios.jl`, `single_time_series.jl`, `time_series_interface.jl` (public API), `time_series_manager.jl`, `time_series_metadata_store.jl` (SQLite-backed), `time_series_storage.jl` + `hdf5_time_series_storage.jl` / `in_memory_time_series_storage.jl`, `time_series_cache.jl`, `time_series_parser.jl`, `time_series_formats.jl`, `time_series_parameters.jl`, `time_series_structs.jl`, `time_series_utils.jl`.
- **Component selection:** `component_selector.jl` — `ComponentSelector` (lazy, named, partitioned subsets of components; `make_selector`, `get_groups`, `rebuild_selector`).
- **Cost / function curves:** `value_curve.jl`, `production_variable_cost_curve.jl`, `cost_aliases.jl`, and `function_data/` (`function_data.jl`, `convexity_checks.jl`, `make_convex.jl`) — `FunctionData`, `ValueCurve`, `ProductionVariableCostCurve` (`CostCurve`/`FuelCurve`), and curve aliases.
- `serialization.jl` — JSON3/StructTypes serialization infrastructure used stack-wide.
- `units.jl` — only time-period conversion helpers here; there is **no** `UnitSystem` enum in IS (that lives in PowerSystems).
- `common.jl`, `definitions.jl`, `iterators.jl`, `random_seed.jl`, `results.jl`, `deprecated.jl`.

Subdirectories:

- `generated/` — **auto-generated struct files. DO NOT EDIT.** Currently the time-series metadata structs (`DeterministicMetadata`, `ProbabilisticMetadata`, `ScenariosMetadata`, `SingleTimeSeriesMetadata`) plus `includes.jl`.
- `descriptors/structs.json` — JSON descriptor that drives struct generation.
- `utils/` — `assert_op.jl` (the `@assert_op` macro), `generate_structs.jl` / `generate_struct_files.jl` (Mustache-based generator), `logging.jl`, `recorder_events.jl`, `timers.jl`, `print*.jl` (PrettyTables display), `sqlite.jl`, `flatten_iterator_wrapper.jl`, `lazy_dict_from_iterator.jl`, `stdout_redirector.jl`, `test.jl`.
- `Optimization/` — submodule `InfrastructureSystems.Optimization` (`module Optimization`). Abstract optimization-container plumbing shared by PowerSimulations and the IOM/POM stack: `optimization_container_keys.jl`, `optimization_container_types.jl`, `optimization_container_metadata.jl`, `abstract_model_store*.jl`, `model_internal.jl`, `optimization_problem_results*.jl`, `optimizer_stats.jl`, `enums.jl`. It defines the *abstractions*; concrete solver logic lives downstream.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sienna-Platform/InfrastructureSystems.jl](https://github.com/Sienna-Platform/InfrastructureSystems.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
