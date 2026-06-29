---
trigger: always_on
description: This repository is a MATLAB-based spectrum sensing simulation system. Its job is
---

# Codex Rules For ChangShuoRadioData

## 1. Mission

This repository is a MATLAB-based spectrum sensing simulation system. Its job is
to generate physically and communicationally consistent synthetic data for
multi-scene radio monitoring, spectrum occupancy analysis, and downstream model
training or evaluation.

Every code change must preserve one core property:

- The generated signal, the scene state, and the annotation must describe the
  same underlying event.

Correctness is more important than speed, convenience, or local elegance.

## 2. Project Shape

Treat the main pipeline as:

1. `SimulationRunner`
2. `ChangShuo`
3. `ScenarioFactory`
4. physical environment simulation
5. communication behavior simulation
6. waveform generation and transmitter processing
7. channel propagation
8. receiver processing
9. annotation export

Do not change a downstream block without checking the contracts from the
upstream block that feeds it and the downstream block that consumes it.

## 3. Scope Discipline

- Read only the code and configuration needed for the task.
- Do not bulk-read unrelated data folders, generated outputs, or map assets.
- Do not rewrite raw datasets, OSM assets, or generated samples unless the task
  explicitly asks for it.
- Keep edits tightly scoped. Avoid opportunistic refactors in unrelated modules.

## 4. Non-Negotiable Simulation Invariants

### 4.1 Time-frequency consistency

- Occupancy annotations must match the actual emitted signal in time.
- Center frequency, bandwidth, sample rate, frame duration, and modulation
  parameters must stay numerically consistent across the pipeline.
- Never silently reinterpret units. If a field changes unit, rename it or add a
  clear conversion step.

### 4.2 Spatial consistency

- Never mix geographic coordinates and Cartesian coordinates silently.
- Latitude and longitude are not meters. Velocity in meters per second must not
  be combined with degree deltas without explicit conversion.
- Antenna height, distance, relative velocity, and map geometry must be handled
  in a physically meaningful way.

### 4.3 Power and channel consistency

- Do not introduce or remove attenuation, gain, or noise terms without tracing
  their effect on received power and annotation fields.
- Channel model selection must be explicit. Do not rely on field order or other
  implicit selection behavior.
- Different Tx-Rx links must not accidentally share mutable channel state unless
  that sharing is physically intended and clearly documented.

### 4.4 Annotation consistency

- If the simulator falls back from one physical model to another, the fallback
  must be visible in metadata or annotations.
- Never claim a physically richer model was used when the code actually used a
  simpler fallback.
- If a value is estimated rather than directly simulated, label it accordingly.

## 5. Map And Environment Rules

### 5.1 OSM handling

- Empty or geometry-free OSM files are valid inputs if the scenario is still
  physically meaningful.
- For empty OSM map geometry, prefer an explicit flat-terrain policy over
  throwing or silently skipping the scenario.
- Building detection must recognize both `building` and `building:part`.
- Inspect only the selected OSM file for geometry checks. Do not scan unrelated
  map files.

### 5.2 Ray tracing behavior

- Ray tracing changes must be tested in both building-present and
  building-absent scenarios.
- For geographic ray tracing, map configuration, terrain material, and site
  construction must remain mutually consistent.
- Any fallback for `NoValidPaths` or unsupported material settings must be
  explicit, logged, and reflected in channel metadata.

### 5.3 Batch safety

- Avoid GUI-dependent code paths in automated or batch tests when a non-visual
  API is available.
- Any viewer or external resource created by a block must have a cleanup path.

## 6. MATLAB Engineering Rules

- Respect System object lifecycle: setup, step, reset, release.
- Do not hide critical state in globals, persistent variables, or workspace side
  effects unless the module already relies on that pattern and the behavior is
  tested.
- Prefer explicit structs, typed metadata fields, and validation over stringly
  typed ad hoc plumbing.
- Validate configuration early. Invalid map ratios, empty type lists, and
  impossible channel settings should fail fast with actionable messages.
- Use local helper functions when they reduce ambiguity, not as cosmetic
  indirection.

## 7. Randomness And Reproducibility

- Any change affecting randomness must preserve deterministic replay when the
  same seed and scenario config are used.
- Do not add hidden RNG draws in helper functions without checking whether they
  perturb frame-level reproducibility.
- If a new stochastic branch is introduced, document which seed stream controls
  it.

## 8. Testing Standard

For every non-trivial change, run targeted tests plus at least one pipeline
regression touching the affected path.

Minimum expectations:

- Config validation tests for new config branches.
- A targeted test for the modified component.
- A regression test for the main simulation path when shared behavior changes.
- For map or channel changes, cover both the intended path and the fallback
  path.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Singingkettle/ChangShuoRadioData](https://github.com/Singingkettle/ChangShuoRadioData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
