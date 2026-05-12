---
trigger: always_on
description: Typed multiple dispatch for Clojure — Julia-style polymorphic arithmetic with devirtualization. (Raster & Juliet / Julia pun)
---

# Raster

Typed multiple dispatch for Clojure — Julia-style polymorphic arithmetic with devirtualization. (Raster & Juliet / Julia pun)

## Project Structure

```
src/raster/
  core.clj                   # Public API: deftm, ftm, defvalue, specialize macros
  numeric.clj                # Polymorphic +, -, *, /, mod, rem, bitwise, comparisons
  math.clj                   # Julia Base math (trig, exp/log, hyperbolic, fma, etc.)
  number_types.clj           # Abstract type hierarchy (Real, AbstractFloat, etc.)
  algebraic_types.clj        # Algebraic lattice (Magma → Ring → Field)
  promote.clj                # Type promotion lattice
  arrays.clj                 # Polymorphic aget/aset/alength
  complex.clj                # Complex number support
  float16.clj                # Float16 value type via Valhalla
  traits.clj                 # Julia-style trait system
  strings.clj                # Julia-style string operations
  random.clj                 # RNG wrappers
  nn.clj                     # NN primitives (dense, relu, softmax) with rrules
  par.clj                    # Parallel primitives (map!/reduce!/scan!/broadcast!)
  tooling/
    inspect.clj              #   REPL tools (bytecode disassembly, JIT diagnostics)
  typed.clj                  # TypedClojure integration

  compiler/                  # Nanopass compiler pipeline
  #   Dependency layering (no cycles allowed):
  #     core/* → pipeline
  #     ir/* defines compiler IR layers
  #     passes/* contains scalar and parallel lowering/optimization
  #     backend/* contains JVM and GPU code generation
  #   requiring-resolve only for genuine cycles (dispatch↔core, bytecode↔inline),
  #   optional deps (TypedClojure, GPU runtimes), or dynamic/computed symbols

  ad/                        # Automatic differentiation
    forward.clj              #   Forward-mode AD (Dual numbers)
    reverse.clj              #   Reverse-mode AD (closures-as-tape)
    rrule.clj                #   Reverse-mode rule registry
    jet.clj                  #   Jet type (truncated Taylor series)
    activity.clj             #   Activity analysis (active vs constant nodes)
    purity.clj               #   Purity analysis (beichte integration)

  linalg/                    # Linear algebra
    core.clj                 #   Dense & fixed-size (Vec2-4, Mat2-4, solve, LU, Cholesky)
    lapack.clj               #   Panama FFI bindings to OpenBLAS
    svd.clj, qr.clj, eigen.clj  # Matrix decompositions
    iterative.clj            #   Krylov methods (CG, GMRES, BiCGSTAB, Lanczos)
    pca.clj                  #   Principal component analysis
    sparse.clj               #   Sparse vectors (sorted compressed format)

  sci/                       # Scientific computing
    special.clj              #   Special functions (gamma, beta, erf, Bessel)
    distributions.clj        #   Probability distributions (Normal, Beta, Gamma, ...)
    stats.clj                #   Statistical tests (t-test, chi-squared, KS)
    optim.clj                #   Optimization (L-BFGS, Nelder-Mead, Newton)
    roots.clj                #   Root finding (bisection, Brent, Newton)
    quadrature.clj           #   Numerical integration (Gauss-Kronrod, Simpson)
    interpolation.clj        #   Splines (linear, cubic, Akima)
    signal.clj               #   Signal processing (windows, PSD, convolution)
    fft.clj                  #   FFT (radix-2 Cooley-Tukey)

  ode/                       # Differential equations
    core.clj                 #   ODE integrators (Euler, RK4, DP5, Tsit5) + GenericODEProblem for sensitivity
    pde.clj                  #   Method-of-lines PDE solver
    sde.clj                  #   Stochastic DEs (Euler-Maruyama)

  sym/                       # Symbolic computation
    core.clj                 #   Symbolic expressions (Sym value type)
    diff.clj                 #   Symbolic differentiation
    analysis.clj             #   Linearity/sparsity analysis
    taylor.clj               #   Taylor series expansion
    fn_algebra.clj           #   Function algebra (D operator, composition)

  ga/                        # Geometric algebra
    core.clj                 #   Cl(p,q,r) multivectors, products, Hodge dual
    compile.clj              #   Valhalla value class generation for GA

  gpu/                       # GPU runtime
    core.clj                 #   Unified session layer (backend dispatch)
    ze_runtime.clj           #   Level Zero (Intel GPUs, Panama FFM)
    ocl_runtime.clj          #   OpenCL ICD (portable, Panama FFM)

  dl/                        # Deep learning
  abm/                     # Agent-based simulation (GPU-compiled)
  vk/                        # Vulkan rendering engine

test/raster/                 # Tests for all modules
bench/                       # Performance comparison benchmarks
design/                      # Architecture documents
```

## Running Tests

### Standard JDK (no Valhalla)
```bash
clojure -M:test
```
Sensitivity tests requiring `Dual4` are guarded by `dual4-available?` and will be skipped.

### Valhalla JDK (full test suite including Dual4)
```bash
JAVA_HOME=~/Development/valhalla-jdk/build/linux-x86_64-server-release/images/jdk \
PATH="$JAVA_HOME/bin:$PATH" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [replikativ/raster](https://github.com/replikativ/raster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
