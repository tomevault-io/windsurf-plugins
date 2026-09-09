---
trigger: always_on
description: This file provides guidance to coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working in this repository.

## Project Overview

Turing.jl is the user-facing entry point for the [TuringLang](https://github.com/TuringLang) probabilistic programming ecosystem. It is largely a translation layer between DynamicPPL models — which work with named, structured parameters — and inference algorithms that expect flat, vectorised samples (e.g. HMC/NUTS operate on `AbstractVector{<:Real}`). DynamicPPL's `LogDensityFunction` handles most of this translation; Turing provides the sampler wrappers that set it up and manage state across iterations.

Model definition lives in [DynamicPPL.jl](https://github.com/TuringLang/DynamicPPL.jl), parameter transformations in [Bijectors.jl](https://github.com/TuringLang/Bijectors.jl), and sampling interfaces in [AbstractMCMC.jl](https://github.com/TuringLang/AbstractMCMC.jl). Turing re-exports their APIs and provides concrete sampler implementations that wire everything together.

## Building and Testing

Code formatting uses [JuliaFormatter.jl](https://github.com/domluna/JuliaFormatter.jl) v1 (not v2) with the **Blue style** (configured in `.JuliaFormatter.toml`). CI enforces formatting on all PRs. JuliaFormatter must be installed in the **global** Julia environment, not the project environment — do not use `--project`. See the [formatting guide](https://turinglang.org/docs/contributing/code-formatting/) for setup details.

```bash
julia -e 'using JuliaFormatter; format(".")'
```

Tests use `SelectiveTests.jl` (in `test/test_utils/`) to filter by path. CI splits the suite into four shards: `mcmc/gibbs.jl`, `mcmc/Inference.jl`, `ad.jl`, and everything else. To run a subset locally:

```bash
julia --project -e 'using Pkg; Pkg.test(; test_args=["mcmc/hmc.jl"])'
```

Use `--skip` to exclude files:

```bash
julia --project -e 'using Pkg; Pkg.test(; test_args=["--skip", "mcmc/gibbs.jl", "ad.jl"])'
```

CI matrix: Julia stable + min, Ubuntu/Windows/macOS, 1 and 2 threads.

`test/test_utils/sampler.jl` provides generic test helpers (`test_rng_respected`, `test_sampler_analytical`, `test_chain_logp_metadata`) that should work for any sampler. Beyond these, sampler-specific tests are needed to capture the properties you care about — there is no standardised test template yet.

## Architecture

### What lives here vs elsewhere

Most complexity is in DynamicPPL. Turing.jl contains:

  - **Sampler implementations** (`src/mcmc/`): HMC/NUTS/HMCDA (wrapping AdvancedHMC), MH (wrapping AdvancedMH), particle samplers SMC/PG/CSMC (implemented natively in `particle_mcmc.jl` on top of Libtask coroutines), ESS (wrapping EllipticalSliceSampling), Emcee, and Gibbs.
  - **External sampler interface** (`src/mcmc/external_sampler.jl`): The `externalsampler()` wrapper lets any `AbstractMCMC.AbstractSampler` that implements `step` for `LogDensityModel` work with Turing models. This is the easier path for new samplers — it only requires a dependency on AbstractMCMC and the LogDensityProblems.jl interface, with no Turing internals. The tradeoff is less power: you can only interact with the model as a black-box log-density function, just like using `LogDensityFunction` directly.
  - **Variational inference** (`src/variational/`): Wraps AdvancedVI algorithms.
  - **Mode estimation** (`src/optimisation/`): MAP and MLE via Optimization.jl.
  - **Custom distributions** (`src/stdlib/`): `Flat`, `FlatPos`, `BinomialLogit`, `OrderedLogistic`, `LogPoisson`, and Dirichlet/Chinese Restaurant processes.

For how the model and inference machinery works under the hood, see the [DynamicPPL docs](https://turinglang.org/DynamicPPL.jl/stable/) and the [developer guides](https://turinglang.org/docs/developers/).

### Gibbs sampler

The Gibbs sampler (`src/mcmc/gibbs.jl`) is the most complex piece in Turing.jl. It threads a `VarNamedTuple` of raw values for all variables through the sweep. To step a component, it `condition`s the model on the values of every variable that component does not sample (`conditioned_values` picks them out), runs the component sampler, and merges the values it returns into the next component's conditioning set. The threaded `VarNamedTuple` is never mutated in place.

Gibbs owns the assignment of variables to component samplers. Repeating that assignment
inside a component risks updating or scoring the wrong variables; see
`VarNamedTuple` for parameter collections below.
`Gibbs(@varname(x) => MH(cov_matrix))` applies `cov_matrix` to the complete linked vector of
the conditioned `x` block.

To plug a sampler into Gibbs, implement:

  - `gibbs_get_parameter_values(state)` — return a `VarNamedTuple` of the values of the variables this sampler is responsible for, leaving out `:=` quantities. The old name `gibbs_get_raw_values` still works, with a deprecation warning.
  - `gibbs_update_state!!(sampler, state, model, global_vals)` — update the sampler's state to reflect new conditioned values. For samplers that use `LogDensityFunction`, the helper `gibbs_recompute_ldf_and_params` handles the common case.
  - Optionally, `supports_gibbs(sampler)` — return `false` to disallow use in Gibbs (the default is `true`). The old name `isgibbscomponent` still works, with a deprecation warning.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TuringLang/Turing.jl](https://github.com/TuringLang/Turing.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
