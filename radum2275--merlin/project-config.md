---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What Merlin is

Merlin is a standalone C++ solver for probabilistic inference over graphical models (Bayesian networks, Markov random fields). It supports the tasks PR (partition function / probability of evidence), MAR (posterior marginals), MAP, MMAP (marginal MAP), and EM (maximum-likelihood parameter learning for Bayes nets only). Inference algorithms include Loopy Belief Propagation (LBP), Iterative Join-Graph Propagation (IJGP), Join-Graph Linear Programming (JGLP), Weighted Mini-Bucket (WMB), Bucket-Tree Elimination (BTE), Clique-Tree Elimination (CTE), and Gibbs sampling. WMB/IJGP/JGLP are parameterized by an **i-bound** that trades accuracy for cost (i-bound ≥ treewidth gives exact inference).

Requires **boost** (linked against `boost_program_options`, and `boost_thread` via CMake). Built with CMake at C++17.

## Build & run

**CLI binary (CMake — the only build for the solver):**
```
mkdir build && cd build
cmake ..
cmake --build .        # produces build/merlin
```

**Python bindings** (`src/merlin_pybind.cpp`, opt-in CMake target): configure with `-DMERLIN_BUILD_PYTHON=ON`; pybind11 is fetched via `FetchContent` (see `python/CMakeLists.txt`), so no preinstalled env is needed. Builds `build/python/merlin.*.so`, importable as `import merlin` (add `build/python` to `PYTHONPATH`). The module exposes the `Merlin` class plus `Task`/`Algorithm`/`InputFormat`/`OutputFormat` IntEnums. `run()` writes the solution to `<output_file>.<TASK>` (there is no in-memory output string), so the Python workflow reads that file back — see the README example.

**Docs** (Doxygen, opt-in CMake target): configure with `-DMERLIN_BUILD_DOCS=ON` then `cmake --build build --target docs` (requires Doxygen installed). `merlin.doxyfile` is the config; it scans `include`, `src`, and `README.md` (the mainpage) and writes HTML to `docs/html/`. Generated output under `docs/` is gitignored. Can also be run directly with `doxygen merlin.doxyfile`.

**Tests** (GoogleTest, in `tests/`). The root CMake build enables them by default (`MERLIN_BUILD_TESTS=ON`) and fetches GoogleTest via `FetchContent` on first configure (needs network once). To build and run:
```
cmake -S . -B build && cmake --build build -j
ctest --test-dir build --output-on-failure          # all tests (~0.4s)
./build/tests/merlin_unit_tests --gtest_filter=Factor.*   # one suite directly
```
`tests/unit/` covers the header value classes (`variable`, `variable_set`, `factor`, `graph`, indexing helpers, `my_set`, etc.) and `graphical_model` UAI parsing; `tests/integration/` runs `Merlin::run()` end-to-end and checks the MAR/PR result against the committed `cancer.uai.MAR` golden and EM against a pinned snapshot in `tests/fixtures/`. The test targets recompile the solver `src/*.cpp` (minus `main.cpp`/`merlin_pybind.cpp`) directly. Configure with `-DMERLIN_BUILD_TESTS=OFF` for a plain binary build with no GoogleTest dependency. There is no linter.

Two latent library quirks are documented (not worked around) by the tests: the `MapType&` overload of `ind2sub` (`variable_set.h`) is declared `void` but `return`s a value (rejected by strict compilers), and `MER_ENUM`'s string constructor only prefix-matches the first value and values right after a comma — later values are stored with a leading space (see `tests/unit/test_enum.cpp`).

You can also verify changes by running the binary against instances in `examples/`, e.g.:
```
./build/merlin --input-file examples/pedigree1.uai --evidence-file examples/pedigree1.evid \
         --task MAR --algorithm wmb --ibound 4 --iterations 10 --output-format json
```
Output is written to `<input>.<TASK>.out` (or a `--output-file`); the EM task writes a new model to `<input>.EM`. See README.md for the full CLI flag list and the UAI file formats (input/evidence/virtual-evidence/query/dataset/output) — these formats are the authoritative interface contract and the README documents them in detail.

## Architecture

**Two entry points, one engine.** The CLI (`src/main.cpp` → `src/program_options.cpp`) and the Python module (`src/merlin_pybind.cpp`) both drive the same `Merlin` facade class (`include/merlin.h`, `src/merlin.cpp`). `main.cpp` parses options into a struct and calls a long sequence of `eng.set_*(...)` setters followed by `eng.run()`; the pybind module exposes those same setters/getters as Python properties. **When you add a configurable parameter, it must be threaded through all of: the `Merlin` member + setter/getter, `program_options` (CLI), and `merlin_pybind.cpp` (Python), to stay consistent across both front ends.**

**Dual I/O modes.** `Merlin` reads models/evidence/query/dataset either from files or from in-memory strings, selected by `set_use_files(bool)`. Every input has both a `*_file` and a `*_string` variant (e.g. `read_model(const char*)` vs `read_model(std::string)`). The string path exists primarily for the Python/embedded use case.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radum2275/merlin](https://github.com/radum2275/merlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
