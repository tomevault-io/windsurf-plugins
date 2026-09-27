---
trigger: always_on
description: **dotFly** is a native C#/.NET 11 inference engine for fly-connectome spiking models (MaleCNS v1.0,
---

# CLAUDE.md — dotFly Project Guide

## Project Identity

**dotFly** is a native C#/.NET 11 inference engine for fly-connectome spiking models (MaleCNS v1.0,
FlyWire v630/v783) — a library that Godot, console apps and experiments link in-process. Sibling of
the dotLLM project in conventions and spirit; read `PLAN.md` for the design.

- **License**: GPLv3 (code). Data: CC BY 4.0, downloaded separately, never vendored.
- **Target framework**: `net11.0` (RC/preview SDK pinned in `global.json`).
- **Reference model**: Shiu et al. 2024 LIF, `philshiu/Drosophila_brain_model@91bdd1e7` `model.py`. The
  equations and step order in `PLAN.md` §2 are the definition of "correct".

## Non-negotiables

1. **Honesty in wording.** Never describe the simulation as a digital fly, an upload, or a brain
   that "plays" a game. The graph is measured; equations, encoders and decoders are engineering
   choices and must be labelled *fixed*, *calibrated* or *trained* wherever they appear.
2. **Exact integer IDs.** Body/root IDs are `ulong`; never pass them through `double`/`float`.
3. **Determinism.** Same checkpoint + options + seed ⇒ identical spikes, independent of thread count
   on CPU. Any change that breaks this needs a test that proves it did not.
4. **Zero allocations on the step path.** State in `NativeBuffer<T>`; no LINQ, no closures, no
   `Task` per step. Scratch via `ArrayPool<T>.Shared`, returned promptly.
5. **Report neural time, simulated duration and wall time separately.** Never conflate them.

## Solution Structure

```
src/DotFly.Core      .dfb reader, tables, NeuronModel, SignPolicy, NativeBuffer, RNG, IBackend
src/DotFly.Data      Arrow/Feather readers → checkpoint builder (filters, sub-graphs, shuffles)
src/DotFly.Cpu       SIMD kernels, blocked-CSR delivery, ComputeThreadPool
src/DotFly.Engine    Simulation, InputPort/OutputPort, SimulationClock, RealtimeDriver, Recorder/Recording, adapters
samples/             SugarExperiment (the Shiu example), TrainReadout (adapters), Godot (2D), Godot3D (the room demo)
src/DotFly.Cli       Spectre.Console.Cli tool: info | build | inspect | run | bench | explore
src/DotFly.Adapters.Onnx / .MLNet   readout adapters over ONNX Runtime / ML.NET (never referenced by Engine)
tests/DotFly.Tests.Unit        xunit v3 (Microsoft.Testing.Platform runner — see global.json)
benchmarks/DotFly.Benchmarks   BenchmarkDotNet
tools/                         Python only for generating Brian2 golden fixtures offline
docs/                          sources of https://kkokosa.github.io/dotFly/ (DocFX; `docfx docs/docfx.json --serve` to preview)
```

## Code Style & Conventions

- File-scoped namespaces; `readonly record struct` for small value types; `Span<T>` in signatures.
- `[MethodImpl(MethodImplOptions.AggressiveInlining)]` on small hot-path methods; `[SkipLocalsInit]`
  on performance-critical methods.
- XML doc comments on all public APIs; `TreatWarningsAsErrors` is on in `src/`.
- Units: mV and ms inside `Core`/`Cpu`; user-facing helpers (`Hz()`, `Ms()`) live in `Engine`.
- Kernels are generic over `TFloat : IBinaryFloatingPointIeee754<TFloat>`; `double` is the reference,
  `float` the fast path. No `Half`.
- SIMD: `Vector512/256/128<T>` cross-platform APIs first, platform intrinsics only when measured
  faster; always a scalar fallback.
- Library projects are `IsAotCompatible`; the CLI opts out (Spectre.Console.Cli reflection).
- `InternalsVisibleTo("DotFly.Tests.Unit")` is the only allowed way for tests to reach internals.

## Engine threading contract

- `Simulation` methods run on one thread (the caller of `Run`/`Advance` or the `RealtimeDriver`
  thread). `InputPort.Write/Fill`, `OutputPort.Snapshot`, the causal switches and the read-only
  properties are safe from any thread; writes take effect at the next `Advance`.
- Use `Reset(seed)` for trials, not a new `Simulation` (constructing one rebuilds the blocked CSR).

## Hosting (Godot and other engines)

- A host that builds in Debug drags referenced projects into Debug: pin the dotFly references with
  `SetConfiguration="Configuration=Release"` (see `samples/DotFly.Sample.Godot`), or the kernel is 25× slower.
- The model has runaway attractors at gain 1.0 (optic lobe under one-sided LC4; antennal lobe /
  mushroom body under *any* ORN input). The 3D sample keeps gain 1.0 by silencing Kenyon cells +
  `lLN*` (`FlyBrain3D.StabilityControl`); `dotfly run --stim-seconds` shows whether a readout
  returns to baseline after the stimulus. Say so on screen whenever a control is applied.
- Until .NET 11 GA, hosts with their own runtime config need `DOTNET_ROLL_FORWARD_TO_PRERELEASE=1`.
- Never pin the caller thread inside a host (default off; `DOTFLY_PIN_CALLER=1` for CLI experiments).

## Hot-path rules learned in M2

- `[MethodImpl(MethodImplOptions.AggressiveOptimization)]` on every method that runs per step
  (kernel, batch loop, delivery, inputs): without it the tiered JIT leaves them at tier-0/OSR for
  most of a short run (3.5× slower). Verify with `DOTFLY_PROFILE=1 dotfly run …`.
- Per-step work must stay block-local: a block touches only its own `v/g/countdown/blocked bits`
  and spike lists published ≥ delay steps earlier. That is what allows one barrier per 18-step batch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kkokosa/dotFly](https://github.com/kkokosa/dotFly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
