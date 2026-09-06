---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this is

SimilaritySearch.jl is a Julia library for nearest-neighbor search. Its flagship index
is `SearchGraph`, an approximate, incrementally-built graph index; it also ships exact
baselines (`ExhaustiveSearch`, `ParallelExhaustiveSearch`), scalar quantization
(`ScalarQuant`), random/Hadamard projections and bit sketches (`Projections`), and
supporting utilities (k-NN result queues, batch parallelism, distance functions).

See `README.md`/`docs/src/index.md` for the research background and citations.

## Build / test / run

```sh
# from the repo root
julia --project=. -e 'using Pkg; Pkg.instantiate()'   # first-time setup
julia -t auto --project=. -e 'using Pkg; Pkg.test()'   # full suite -- required before commit/push
```

**Always pass `-t auto` (or `-tN`) when testing anything threading-related.** The default
session is single-threaded (`Threads.nthreads() == 1`), which silently takes every fast
serial path in `@BATCHES` and never exercises real parallelism, races, or
scheduler-specific behavior.

Individual test files live in `test/*.jl` and are `include`d from `test/runtests.jl`; to
run just one, `include` it directly after `using SimilaritySearch` in a REPL/script rather
than editing `runtests.jl`. `Aqua.jl` ambiguity/quality checks only run under
`VERSION == v"1.10"` (see the top of `runtests.jl`).

### Fast dev loop vs. the pre-commit/pre-push gate

Measured directly (this repo, 2026-08): a fresh `julia -t auto --project=. -e 'using Pkg;
Pkg.test()'` costs **~180s**, and **~90% of that is one-time JIT compilation** of
SearchGraph/InvertedFiles/SearchModels code paths, not test data size -- running the exact
same suite a *second* time inside the *same already-warm process* (no new process, so
compilation is already cached) drops to **~20s**. Shrinking `n`/iteration counts only
shaves a further ~20% off that already-warm 20s (`FAST_TESTS=true`, see below); it does
**nothing** for a cold process, because compilation swamps it. Concretely:

| invocation | cost |
|---|---|
| `julia -e '...; Pkg.test()'` (fresh process each time) | ~180s, `FAST_TESTS` included |
| same suite, 2nd `include` in an already-running session | ~20s |
| ...with `FAST_TESTS=true` on top | ~16s |

**The actual lever for a fast dev loop is a persistent process, not smaller data.** Keep one
Julia session open (e.g. with [`Revise.jl`](https://github.com/timholy/Revise.jl)) and
re-`include` a test file after each edit instead of spawning `julia -e ...`/`Pkg.test()`
per iteration:

```julia
using Revise, SimilaritySearch, Test
ENV["FAST_TESTS"] = "true"   # optional: shrinks the handful of tests whose cost actually
                             # scales with dataset size/iteration count (SearchGraph/
                             # InvertedFile construction, optimize_index! autotuning,
                             # SpatialAccessTree) -- worthwhile once warm, negligible cold.
includet("test/testsearchgraph.jl")   # Revise.includet, not include -- tracks edits
# ...edit source, then just re-run the line above; no new process, no re-compiling the world
```

`FAST_TESTS` reads once per session (`@isdefined(FAST_TESTS) || (const FAST_TESTS = ...)`
guard at the top of every test file that uses it) — set the `ENV` var (or export it before
launching Julia) *before* the first `include`/`Pkg.test()` call in that process; changing
`ENV["FAST_TESTS"]` mid-session has no effect on an already-`include`d file.

**Before commit/push, run the full gate** — plain `Pkg.test()` (no `FAST_TESTS`, fresh
process, full data, Aqua enabled on Julia 1.10) — since that's the only way to reliably
exercise a cold-compile path and the full-size code paths (`Pkg.test()` also always runs in
its own isolated sandboxed environment, unlike a warm dev session).

### Julia version matrix

CI (`.github/workflows/ci.yml`) only officially tests **Julia 1.12**. The package also
supports 1.10 and 1.11 (verified by hand repeatedly during development, not by CI) via
`@static if VERSION >= v"1.11"` gates, mainly in `src/parallel.jl` (native
`Threads.@threads :greedy` doesn't exist before 1.11). If `juliaup` has other versions
installed, cross-check with:

```sh
julia +1.11 -t auto --project=. -e 'using Pkg; Pkg.test()'
julia +1.12 -t auto --project=. -e 'using SimilaritySearch'   # at least a load smoke-test
```

## Architecture map (`src/`)

- `SimilaritySearch.jl` — top-level module; defines `AbstractSearchIndex`,
  `AbstractContext`, the `search`/`searchbatch`/`push_item!`/`append_items!`/`index!`
  generic-function interface, and `getminbatch` (see Parallelism below).
- `parallel.jl` — the `@BATCHES` macro (see below). **Read this file's docstrings before
  touching any parallel loop** — it documents real hygiene pitfalls, not just API.
  `include`d early in `SimilaritySearch.jl`, right after the module opens.
- `log.jl` — the two logging channels. A context holds `reporters` (receive `INFORM`/
  `@inform`, render progress for reading) and `observers` (receive `OBSERVE`, react to a
  structural `:add!` so something durable happens). `reporters=[]` silences a context
  completely; observers are untouched by that. Two rules that are easy to break: **never

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sadit/SimilaritySearch.jl](https://github.com/sadit/SimilaritySearch.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
