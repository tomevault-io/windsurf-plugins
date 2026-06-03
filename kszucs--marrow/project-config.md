---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Marrow is an implementation of Apache Arrow in Mojo. Apache Arrow is a cross-language development platform for in-memory data with a standardized columnar memory format. This implementation is in early/experimental stages as Mojo itself is under heavy development.

For information about the Mojo programming language and the standard library see https://github.com/modular/modular

## Build System & Commands

This project uses **pixi** as the package manager. Commands are scoped to environments:

| Environment | Purpose | Key command |
|-------------|---------|-------------|
| `dev`       | Tests + formatting (default for development) | `pixi run -e dev test` |
| `asan`      | AddressSanitizer test runs | `pixi run -e asan test_mojo_asan` |
| `bench`     | Benchmarks (polars, duckdb for comparison) | `pixi run -e bench bench` |
| `format`    | Formatting only (no test deps) | `pixi run -e format fmt` |
| `docs`      | Documentation generation | `pixi run -e docs docs` |
| `examples`  | Runnable examples | `pixi run -e examples datafusion_udf` |

```bash
# Run all tests
pixi run -e dev test

# Format code
pixi run -e dev fmt

# Build package
pixi run package
```

### Running Individual Tests

Always use `pytest` to run tests — never `mojo test` or `mojo run` directly.
The pytest harness handles build caching, test selection, output parsing, and
ASAN integration.

```bash
# single file
pixi run -e dev pytest marrow/tests/test_dtypes.mojo

# single test case
pixi run -e dev pytest marrow/tests/test_arrays.mojo::test_primitive_slice

# verbose (shows PASS/FAIL per test)
pixi run -e dev pytest -v marrow/kernels/tests/test_join.mojo
```

Useful options:

```bash
--benchmark              # include bench_*.mojo files; also enables -O3
--asan                   # AddressSanitizer (requires asan environment)
--gpu                    # include GPU tests (requires Metal/CUDA device)
--no-python              # skip Python binding tests
--competition            # print a side-by-side comparison table after benchmarks
```

The harness compiles runners to `.test_runners/test_runner_<hash>` (content-
hashed, stable across runs).  Re-running the same test selection skips
recompilation (~1 s vs ~5 s cold).

Tests run sequentially by default. Use `*_parallel` task variants (e.g.
`test_mojo_parallel`) to enable `--dist=loadfile` parallelism, which groups
all tests from the same `.mojo` file on the same worker so the compiled binary
is reused.  Benchmark tasks always pass `-n0` to disable parallelism for
accurate timing.

The Python shared library (`python/marrow.so`) is rebuilt automatically by
`conftest.py` before each test session — no manual `build_python` step needed.

### Writing Mojo Tests

Test files (`test_*.mojo`) use `TestSuite` from `marrow.testing`:

```mojo
from marrow.testing import TestSuite

def test_something() raises:
    assert_true(1 + 1 == 2)

def main():
    TestSuite.run[__functions_in_module()]()
```

`TestSuite.run` auto-discovers every `test_*` function in the module.

### Writing Mojo Benchmarks

Benchmark files (`bench_*.mojo`) use `BenchSuite` and `Benchmark` from
`marrow.testing`:

```mojo
from marrow.testing import BenchSuite, Benchmark, BenchMetric

def bench_my_kernel(mut b: Benchmark) raises:
    var data = _prepare_data(N)
    b.throughput(BenchMetric.elements, N)
    @always_inline
    @parameter
    def call():
        keep(my_kernel(data))
    b.iter[call]()
    keep(data)  # prevent ASAP destruction (see note below)

def main():
    BenchSuite.run[__functions_in_module()]()
```

**Important — `keep(data)` after `b.iter[call]()`**: Mojo's ASAP (As-Soon-As-Possible) destruction frees values as early as the compiler believes their last use has passed. When a `@parameter` closure captures a variable (e.g. `data`) and is passed to `b.iter[call]()`, ASAP may determine that `data` is no longer needed *after* the closure is registered but *before* it actually runs, causing a heap-use-after-free inside the iteration loop. Adding `keep(data)` after `b.iter[call]()` forces `data` to remain live through the entire benchmark. This applies to all non-trivial captured values: `StructArray`, `PrimitiveArray[T]`, `SwissHashTable`, `HashJoin`, etc.

For multiple sizes, define a shared helper and one thin wrapper per size:

```mojo
def _bench_kernel(mut b: Benchmark, n: Int) raises:
    ...

def bench_kernel_10k(mut b: Benchmark) raises: _bench_kernel(b, 10_000)
def bench_kernel_100k(mut b: Benchmark) raises: _bench_kernel(b, 100_000)
def bench_kernel_1m(mut b: Benchmark) raises: _bench_kernel(b, 1_000_000)
```

## Core Architecture

### Type-Erased Containers

Mojo lacks dynamic dispatch, so the codebase uses **type-erased containers** with **implicit conversions** to/from typed wrappers. Implicit conversions are cheap (O(1) ref-count bumps).

#### Arrays (`marrow/arrays.mojo`)

- **`Array`** - Trait that all typed arrays implement. Provides the common read-only interface: `type()`, `null_count()`, `is_valid()`, `as_any()`. Also extends `Sized`, `Writable`, `Equatable`, `Copyable`, `Movable`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kszucs/marrow](https://github.com/kszucs/marrow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
