---
trigger: always_on
description: NumPy reimplementation in Rust with PyO3 bindings.
---

# Rumpy Development Guide

NumPy reimplementation in Rust with PyO3 bindings.

## Build & Test

```bash
# First time setup
uv venv && source .venv/bin/activate
uv pip install pytest numpy hypothesis

# Build (required after Rust changes)
PYO3_USE_ABI3_FORWARD_COMPATIBILITY=1 uv tool run maturin develop

# Test
pytest tests/ -v
```

## Testing Philosophy

See `designs/testing.md` for full details. Key points:

- **NumPy is truth**: Every test compares rumpy against numpy
- **Parametrize over operations**: Group by domain, test all variants
- **Tiered dtypes**: Core dtypes for wrappers, full coverage for custom code
- **Use fixtures**: `conftest.py` has dtype/shape constants and fixtures

```python
from conftest import FLOAT_DTYPES, NUMERIC_DTYPES, CORE_SHAPES

# Pattern: parametrize over operations
POSITIVE_UFUNCS = ["sqrt", "log", "log10"]

class TestPositiveUfuncs:
    @pytest.mark.parametrize("ufunc", POSITIVE_UFUNCS)
    @pytest.mark.parametrize("dtype", FLOAT_DTYPES)
    def test_dtypes(self, ufunc, dtype):
        n = np.array([1, 4, 9], dtype=dtype)
        r = rp.asarray(n)
        assert_eq(getattr(rp, ufunc)(r), getattr(np, ufunc)(n))
```

**Helpers** (`tests/helpers.py`):
- `assert_eq(r, n)` - compare rumpy vs numpy arrays
- `make_numpy(shape, dtype)` - generate test array with non-trivial values
- `make_pair(shape, dtype)` - returns (rumpy, numpy) pair

## File Structure

```
src/
├── lib.rs                    # Crate root, PyO3 module entry point
├── array/                    # Core array types
│   ├── mod.rs                # RumpyArray struct, constructors, shape ops
│   ├── buffer.rs             # ArrayBuffer (Arc-wrapped memory)
│   ├── flags.rs              # ArrayFlags (contiguity, writeable)
│   ├── iter.rs               # StridedIter, AxisOffsetIter
│   └── dtype/                # DType system
│       ├── mod.rs            # DType wrapper, DTypeOps trait, promotion
│       ├── macros.rs         # impl_float_dtype!, impl_signed_int_dtype!
│       ├── float32.rs        # Float32 implementation
│       ├── float64.rs        # Float64 implementation
│       ├── int*.rs           # Integer type implementations
│       ├── bool.rs           # Boolean implementation
│       └── complex*.rs       # Complex number implementations
│
├── ops/                      # Operations (Rust implementation)
│   ├── mod.rs                # Core binary/compare ops, re-exports
│   ├── kernels/              # Pure operation definitions (kernel/dispatch system)
│   │   ├── mod.rs            # Traits: BinaryKernel, UnaryKernel, ReduceKernel, CompareKernel
│   │   ├── arithmetic.rs     # Add, Sub, Mul, Div, Sum, Prod, Max, Min, NanSum, etc.
│   │   ├── bitwise.rs        # And, Or, Xor, LeftShift, RightShift, Not
│   │   ├── comparison.rs     # Gt, Lt, Ge, Le, Eq, Ne
│   │   └── math.rs           # Sqrt, Exp, Log, Sin, Cos, etc.
│   ├── loops/                # Layout strategies (SIMD optimizations live here)
│   │   ├── mod.rs            # Re-exports
│   │   ├── contiguous.rs     # Slice-based loops (SIMD-friendly, 8-accumulator reductions)
│   │   └── strided.rs        # Pointer arithmetic loops for non-contiguous
│   ├── dispatch.rs           # Type resolution + layout detection → kernel + loop
│   ├── ufunc.rs              # Public API: map_unary_op, map_binary_op, reduce_axis_op
│   ├── registry.rs           # Legacy loops (Bool reductions only)
│   ├── array_methods/        # RumpyArray method implementations by category
│   │   ├── unary.rs          # sqrt, exp, log, sin, cos, real, imag, nan_to_num
│   │   ├── reductions.rs     # sum, mean, var, std, argmax + NaN variants
│   │   ├── sorting.rs        # sort, argsort, partition, unique, lexsort
│   │   ├── cumulative.rs     # diff, cumsum, cumprod
│   │   └── logical.rs        # all, any, count_nonzero
│   ├── comparison.rs         # logical_and/or/xor/not, equal, isclose
│   ├── bitwise.rs            # bitwise_and/or/xor/not, shifts
│   ├── statistics.rs         # histogram, cov, corrcoef, median
│   ├── numerical.rs          # gradient, trapezoid, interp, correlate
│   ├── poly.rs               # polyfit, polyval, polyder, polyint, roots
│   ├── set_ops.rs            # isin, intersect1d, union1d, setdiff1d
│   ├── indexing.rs           # take, put, searchsorted, compress
│   ├── linalg.rs             # eig, svd, lstsq, pinv, matrix_rank
│   ├── fft.rs                # FFT operations
│   ├── matmul.rs             # Matrix multiplication
│   ├── dot.rs, inner.rs      # Dot and inner products
│   ├── outer.rs              # Outer product
│   └── solve.rs              # Linear system solving
│
├── python/                   # PyO3 bindings (thin wrappers)
│   ├── mod.rs                # register_module, re-exports
│   ├── pyarray/              # PyRumpyArray class (split for maintainability)
│   │   ├── mod.rs            # struct, properties, misc methods, helpers
│   │   ├── dunder_ops.rs     # __add__, __sub__, __eq__, __and__, etc.
│   │   ├── dunder_item.rs    # __getitem__, __setitem__
│   │   └── methods_reductions.rs  # sum, mean, var, std, argmax, all, any
│   ├── creation.rs           # zeros, ones, arange, linspace, eye, full
│   ├── ufuncs.rs             # sqrt, sin, cos, add, multiply (math ops)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrocklin/rumpy](https://github.com/mrocklin/rumpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
