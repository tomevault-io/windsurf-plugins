---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What is FZGPUModules

FZGPUModules is a **CUDA library for building composable, high-throughput error-bounded
compression pipelines**. Rather than picking one monolithic compressor, you assemble a
directed acyclic graph (DAG) of independent GPU stages — predictors, quantizers, coders,
shufflers, transforms, and fused kernels — connected and executed entirely on the GPU with
stream-ordered memory management. It vendors/reimplements ideas from cuSZ, cuSZp/cuSZp2/cuSZp3,
cuSZ-Hi (LC), FZ-GPU, and PFPL as modular, mix-and-matchable stages.

## The pipeline model

Every pipeline is built with `fz::Pipeline` (`include/pipeline/compressor.h`):

```cpp
fz::Pipeline p(input_bytes, fz::MemoryStrategy::PREALLOCATE, /*pool_mult=*/4.0f);
p.setDims(nx, ny, nz);                 // before addStage, for dimension-aware stages
auto* stage = p.addStage<YourStage>();   // YourStage = a concrete stage class
p.connect(downstream, upstream, "codes");   // named port, not always "output"
p.finalize();                          // validates, sorts into levels, (pre-)allocates
p.compress(d_input, input_bytes, &d_comp, &comp_sz, stream);
p.decompress(d_comp, comp_sz, &d_decomp, &decomp_sz, stream);
```

Stages at the same DAG level run concurrently on separate CUDA streams. See
[docs/architecture.md](docs/architecture.md) for the full layer model (`Pipeline` →
`CompressionDAG` → `Stage`/`MemoryPool`) and [docs/api_reference.md](docs/api_reference.md)
for the complete lifecycle/enum reference.

## Stage categories (`modules/`)

| Category | Directory | What it does | Docs |
|---|---|---|---|
| Predictors | `modules/predictors/` | Model-based decorrelation (Lorenzo, tiled Lorenzo, diff/delta) | [docs/stages/predictors.md](docs/stages/predictors.md) |
| Quantizers | `modules/quantizers/` | Lossy discrete mapping (error-bounded quantizer) | [docs/stages/quantizers.md](docs/stages/quantizers.md) |
| Transforms | `modules/transforms/` | Invertible basis changes (zigzag, negabinary, log) | [docs/stages/transforms.md](docs/stages/transforms.md) |
| Coders | `modules/coders/` | Symbol-to-bitstream encoding (RLE, RZE, RRE, bitpack, adaptive bitpack, Huffman, ANS) | [docs/stages/coders.md](docs/stages/coders.md) |
| Shufflers | `modules/shufflers/` | Lossless data restructuring for better compressibility (bitshuffle) | [docs/stages/shufflers.md](docs/stages/shufflers.md) |
| Fused | `modules/fused/` | Combined multi-operation kernels (lorenzo_quant, GInterp, bitplane_rze) | [docs/stages/fused.md](docs/stages/fused.md) |
| Structural | `modules/structural/` | DAG plumbing, not a compression operation (merge N ports → 1) | [docs/stages/structural.md](docs/stages/structural.md) |

Full per-stage constraints, ports, and TOML keys: [docs/stages/index.md](docs/stages/index.md).
How to add a brand-new stage: [docs/how_to_add_a_stage.md](docs/how_to_add_a_stage.md).

## Build

```bash
cmake --preset release                          # default (no examples/tests/profiling)
cmake --preset release -DBUILD_EXAMPLES=ON      # with examples
cmake --preset release -DBUILD_TESTING=ON       # with tests
cmake --preset release -DBUILD_PROFILING=ON     # with profiling
cmake --build build/release -j$(nproc)
```

Test binaries go to `build/release/tests/`, example binaries to `build/release/bin/examples/`,
profiling binaries to `build/release/bin/profiling/`. Full option/preset list:
[docs/building.md](docs/building.md).

## Testing

```bash
ctest --preset default            # all tests, release build
ctest --preset stages             # stage unit tests only
ctest --preset pipeline           # pipeline integration tests only
ctest --preset asan               # full suite, ASan + UBSan     — see note below
ctest --preset compute-san        # full suite, compute-sanitizer — see note below
```

Both sanitizer modes require their runtime tools: Compute Sanitizer must be on `PATH`,
and the ASan run needs GCC's `libasan`. Prefer the checked-in wrapper, which manages the
sanitizer builds, `LD_PRELOAD`, and CUDA-compatible sanitizer options:

```bash
./scripts/run_sanitizers.sh                  # full sanitizer matrix
./scripts/run_sanitizers.sh --mode compute   # Compute Sanitizer only
./scripts/run_sanitizers.sh --mode asan      # ASan + UBSan only
```

`LD_PRELOAD` must not be exported globally — it would inject ASan into `nvcc` and
every other process. Run Compute Sanitizer on any change to device code: it catches
out-of-bounds device writes that land in pool slack, which a normal test run can miss.

`tests/` layout: `stages/` (per-stage unit tests), `pipeline/` (integration tests),
`golden/` (reference data), `helpers/`.

## Directory layout

```
include/            public headers (only these in user/example code)
modules/             stage implementations, see table above
src/pipeline/        Pipeline, DAG, MemoryPool internals
examples/            example programs (-DBUILD_EXAMPLES=ON)
profiling/           profiling programs (-DBUILD_PROFILING=ON)
tests/               test suite (-DBUILD_TESTING=ON)
docs/                doxygen-driven documentation site (mainpage, architecture, stages/, api_reference)
scripts/             helper scripts
```

## Key rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [szcompressor/FZGPUModules](https://github.com/szcompressor/FZGPUModules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
