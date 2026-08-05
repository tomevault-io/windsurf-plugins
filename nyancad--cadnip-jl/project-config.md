---
trigger: always_on
description: **Use Julia 1.12.** Better compilation performance for long functions (like VA
---

# Claude Development Notes

## Environment

**Use Julia 1.12.** Better compilation performance for long functions (like VA
models), and it builds a full c6288 (212k-variable, PSP103-heavy) circuit with
no trouble.

- On a local machine Julia is usually pre-installed via juliaup — just use `julia`.
- In a fresh cloud/remote session Julia is typically not pre-installed. Install it:
  ```bash
  curl -fsSL https://install.julialang.org | sh -s -- -y
  . ~/.bashrc
  ~/.juliaup/bin/juliaup add 1.12 && ~/.juliaup/bin/juliaup default 1.12
  ```
  then run Julia via `~/.juliaup/bin/julia` (full path).
- For a fresh local setup: `juliaup add 1.12 && juliaup default 1.12`.

**Heavy VA model precompilation** (PSP103VA with 200+ params, BSIM4) can be slow
or memory-hungry. To skip the compile workloads, create
`test/LocalPreferences.toml`:

```toml
[PSPModels]
precompile_workload = false

[VADistillerModels]
precompile_workload = false
```

This file is gitignored. The packages will still work but with slower first-call latency.
See [PrecompileTools docs](https://julialang.github.io/PrecompileTools.jl/stable/) for details.

### CI Environment

- CI uses Julia 1.11 (what Manifest.toml is locked to)
- Don't add compatibility hacks for older Julia versions

## Development Guidelines

### Code Modification Philosophy

- **ALWAYS update existing code** - refactor and modify in place
- **NEVER add compatibility layers** - no deprecation wrappers, no duplicate APIs
- **NEVER create parallel implementations** - one clean API, not old + new
- We are at early stage development where breaking changes are expected
- If you need to change behavior, change it directly - don't preserve the old way

### Measure before you claim

Run it before asserting it. Claims about how this codebase behaves — what a
lens returns, which branch is reachable, whether a builder can be observed —
are cheap to check in a REPL and expensive to get wrong, because a plausible
wrong one gets written into a design doc and believed later.

Two traps that have already produced wrong claims here:

- **A synthetic probe is not real usage.** Calling `getproperty(observer, :vin)`
  by hand "showed" a phantom-child bug in `ParamObserver`; no builder does that,
  and the real call path was correct. The fix derived from the probe broke the
  `.param x1` / `X1` collision case.
- **Reading the code is not running the code.** `ParamLens.getproperty` looks
  like it returns a `ValLens` for a leaf. It cannot — canonicalization moves
  leaves under `params` first — so that branch was dead for years.

When a measurement contradicts the reasoning, the measurement wins, and the
prose that ships should be written from the measurement.

### MNA Backend Migration

- **DO NOT maintain backward compatibility with DAECompiler**
- Update existing APIs to use the new MNA backend directly
- When modifying sweep/simulation code, replace DAECompiler patterns with MNA equivalents
- Do not create duplicate types (e.g., `MNACircuitSweep`) - modify existing types instead

### Key MNA Components

- `MNACircuit`: Parameterized circuit simulation wrapper
- `MNAContext`: Circuit builder context for stamping (structure discovery)
- `DirectStampContext`: Zero-allocation context for fast restamping during solve
- `alter()`: Create new simulation with modified parameters
- `dc!()` / `tran!()`: DC and transient analysis
- `CircuitSweep`: Parameter sweep over MNA circuits

See `doc/` for design documents. Check `git log --oneline -20 --name-only` for recently changed files relevant to current work.

## CI and Testing

### Workflow

1. **Sanity check** - run the specific test file for what you changed
2. **Commit and push** - CI runs `test-core` + `test-integration` in parallel
3. **Run full tests locally** - `all` tests + benchmarks while CI runs

### Commands

```bash
# Specific test file (sanity check)
~/.juliaup/bin/julia --project=test test/mna/core.jl

# All tests (core + integration)
~/.juliaup/bin/julia --project=test test/runtests.jl all

# Benchmarks
~/.juliaup/bin/julia --project=. benchmarks/vacask/run_benchmarks.jl

# Parser tests
~/.juliaup/bin/julia --project=NyanSpectreNetlistParser.jl -e 'using Pkg; Pkg.test()'
~/.juliaup/bin/julia --project=NyanVerilogAParser.jl -e 'using Pkg; Pkg.test()'
```

### Test Files

| File | What it tests |
|------|---------------|
| `test/mna/core.jl` | MNA stamping, matrix assembly, DC/AC |
| `test/mna/va.jl` | VA contribution stamping |
| `test/basic.jl` | SPICE codegen, simple circuits |
| `test/transients.jl` | PWL/SIN sources |
| `test/sweep.jl` | Parameter sweeps |
| `test/mna/vadistiller.jl` | VADistiller models |
| `test/mna/vadistiller_integration.jl` | Large VA models (BSIM4) |
| `test/mna/audio_integration.jl` | BJT circuits |

### Test Style: prefer netlists + the high-level API

**Default to SPICE/Spectre netlists driven through the high-level API for any
test that asserts on *circuit behavior* (a DC operating point, a transient
trajectory, convergence, model-card parameter handling, an AC response).**
Reserve hand-written `stamp!` / `MNAContext` / `get_node!` builders for unit
tests that specifically exercise *low-level stamping mechanics* — matrix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NyanCAD/Cadnip.jl](https://github.com/NyanCAD/Cadnip.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
