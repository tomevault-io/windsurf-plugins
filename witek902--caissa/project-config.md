---
trigger: always_on
description: Caissa is a strong UCI-compatible chess engine written in C++20 with a custom NNUE-style neural network evaluator. It targets rated play at 3600+ ELO and supports standard chess, FRC (Chess960), and DFRC.
---

# Caissa Chess Engine – Copilot Instructions

Caissa is a strong UCI-compatible chess engine written in C++20 with a custom NNUE-style neural network evaluator. It targets rated play at 3600+ ELO and supports standard chess, FRC (Chess960), and DFRC.

## Build & Test

### Linux (CMake – recommended)
```bash
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..   # or Final / Debug
make -j$(nproc)
```

Build types:
- `Final` – production: no asserts, maximum optimizations
- `Release` – development: asserts on, optimizations on
- `Debug` – development: asserts on, optimizations off

Architecture (default is `x64-bmi2` on x86-64):
```bash
cmake -DTARGET_ARCH=x64-avx512 -DCMAKE_BUILD_TYPE=Final ..
# Other options: x64-bmi2, x64-avx2, x64-sse4-popcnt, x64-legacy, aarch64-neon, aarch64
```

### Linux (Makefile – quick, builds AVX2/BMI2)
```bash
cd src && make -j$(nproc)
```

### Windows
Run `GenerateVisualStudioSolution.bat`, then open `build_<arch>/caissa.sln` in Visual Studio 2022.

### Running tests
```bash
# Unit tests (from build directory)
bin/utils unittest

# Smoke test / benchmark
bin/caissa "bench" "quit"

# Performance tests
bin/utils perftest [paths...]
```

### Neural network file
Neural network files (`.pnn`) are **not stored in this repository**. They are hosted in [Witek902/Caissa-Nets](https://github.com/Witek902/Caissa-Nets) as GitHub Release assets.

- **CMake**: automatically downloads the net to `data/neuralNets/` at configure time and copies it to the build output directory. No manual steps needed.
- **Makefile**: automatically downloads the net to `data/neuralNets/` via `curl` before compilation (required for INCBIN embedding). No manual steps needed.
- **`data/neuralNets/*.pnn` is in `.gitignore`** — never commit net files to this repo.

To update to a new net version, change `DEFAULT_NEURAL_NET_FILE_NAME` / `NET_VERSION` in `CMakeLists.txt`, `DEFAULT_EVALFILE` in `src/makefile`, and `c_DefaultEvalFile` in `src/backend/Evaluate.cpp`.

---

## Architecture

The project has three CMake targets:

| Target | Type | Location | Purpose |
|---|---|---|---|
| `backend` | static library | `src/backend/` | Engine core: search, eval, move gen, position |
| `caissa` (frontend) | executable | `src/frontend/` | UCI protocol wrapper |
| `utils` | executable | `src/utils/` | Trainer, self-play, unit tests, perf tests |

`utils` depends on `backend`; `frontend` also depends on `backend`.

### backend – key files
- `Search.cpp/.hpp` – negamax with alpha-beta, PVS, LMR, null-move pruning, singular extensions, correction history
- `Position.cpp/.hpp` – board state; `SidePosition` holds per-color bitboards + piece array
- `MoveGen.hpp`, `MoveList.hpp` – move generation; max 280 moves per position (`MaxAllowedMoves`)
- `PackedNeuralNetwork.cpp/.hpp` – runtime NNUE inference (manually SIMD-vectorized)
- `NeuralNetworkEvaluator.cpp/.hpp` – incremental accumulator updates; `AccumulatorCache`
- `TranspositionTable.cpp/.hpp` – shared TT with large-page support
- `Evaluate.cpp/.hpp` – static eval entry point; piece values, WLD model
- `MoveOrderer.cpp/.hpp`, `MovePicker.cpp/.hpp` – move ordering
- `TimeManager.cpp/.hpp` – time management
- `Endgame.cpp/.hpp` – special endgame routines
- `Tablebase.cpp/.hpp` – Syzygy (enabled by default) and Gaviota (opt-in) probing
- `Tuning.cpp/.hpp` – `DEFINE_PARAM` macro for exposing search params to UCI (requires `ENABLE_TUNING` build flag)

### Neural network (runtime)
Architecture: `(32×768 → 1024) × 2 → 1` (dual-perspective, one accumulator per king side, 32 king buckets, 768 = 12 piece types × 64 squares). The last layer has 8 variants selected by piece count. Network files use the `.pnn` extension.

### utils – subcommands (invoked as `bin/utils <command>`)
`unittest`, `perftest`, `selfplay`, `prepareTrainingData`, `plainTextToTrainingData`, `dumpGames`, `testNetwork`, `trainNetwork`, `trainCudaNetwork` (CUDA only), `validateEndgame`, `generateEndgamePositions`, `analyzeGames`

CUDA trainer is optional: auto-detected by CMake; compiled only when CUDA Toolkit is found (`USE_CUDA` define).

---

## Key Conventions

### Macros and inlining
- `INLINE` / `NO_INLINE` / `INLINE_LAMBDA` – cross-compiler inlining (`__forceinline` on MSVC, `__attribute__((always_inline))` on GCC/Clang)
- `ASSERT(x)` – fires in Debug/Release, compiled out in Final
- `VERIFY(x)` – same condition but always executes the expression; use where side-effects must survive Final builds
- `UNUSED(x)` – suppress unused-variable warnings

### Build configuration defines
- `CONFIGURATION_DEBUG` / `CONFIGURATION_RELEASE` / `CONFIGURATION_FINAL` – set by CMake per build type
- SIMD capability defines: `USE_SSE`, `USE_SSE2`, `USE_SSE4`, `USE_POPCNT`, `USE_AVX`, `USE_AVX2`, `USE_BMI2`, `USE_AVX512`, `USE_ARM_NEON`
- Architecture: `ARCHITECTURE_X64` or `ARCHITECTURE_AARCH64`
- Always guard SIMD code with the appropriate `#ifdef USE_*` / `#ifdef NN_USE_*` blocks

### Types and constants (defined in `Common.hpp` / `Score.hpp`)
- `ScoreType` = `int16_t`; scores are in centipawns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Witek902/Caissa](https://github.com/Witek902/Caissa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
