---
trigger: always_on
description: Welcome to `Tinyopt`, a high-performance, header-only C++20 optimization library engineered for unconstrained optimization and non-linear least squares (NLLS) problems.
---

# AGENTS.md — Development Guidelines for Tinyopt

Welcome to `Tinyopt`, a high-performance, header-only C++20 optimization library engineered for unconstrained optimization and non-linear least squares (NLLS) problems.

As an AI agent or engineer working in this repository, you **must** uphold the highest engineering standards: zero compiler warnings, zero dynamic memory allocations in critical paths, rigorous mathematical and gradient verification, and consistent code formatting.

> **CRITICAL AGENT COMMIT POLICY**:
> AI agents must **NEVER** automatically create git commits (`git commit`) unless explicitly commanded to do so by the user within the active session (e.g., "ok commit now", "create a commit"). Keep working changes in the working tree or staging area.

---

## 1. Repository Architecture & Core Philosophy

Tinyopt achieves superior computational speed and memory efficiency through its **Accumulation Pattern**:
- Unlike traditional NLLS libraries that allocate and buffer large arrays of residual vectors and Jacobian matrices, `Tinyopt` empowers users and solvers to accumulate gradients ($J^T r$) and Hessian approximations ($J^T J$) directly into the linear system.
- This curtailment of memory allocation minimizes cache misses and unlocks rapid convergence for small-to-medium and structured optimization problems.
- For a comprehensive architectural deep-dive, see [docs/architecture.md](docs/architecture.md).

### Directory Layout

```text
tinyopt/
├── include/tinyopt/          # Header-only core library
│   ├── tinyopt.h             # Umbrella include header
│   ├── traits.h              # Template metaprogramming & type traits
│   ├── types.h               # Eigen aliases (VecX, MatX, etc.) & container types
│   ├── cost.h                # Cost evaluation and residual wrappers
│   ├── optimize.h            # High-level entry point (Optimize templates)
│   ├── stop_reasons.h        # Convergence and termination criteria
│   ├── time.h                # High-resolution profiling utilities
│   ├── optimizers/           # Iterative optimization algorithms
│   │   ├── optimizer.h       # Base iterative loop, trust region / step logic
│   │   ├── lm.h              # Levenberg-Marquardt optimizer
│   │   ├── gn.h              # Gauss-Newton optimizer
│   │   ├── gd.h              # Gradient Descent optimizer
│   │   └── options.h         # Solver options & parameters
│   ├── solvers/              # Linear system solvers
│   │   ├── lm.h, gn.h, gd.h  # Linear step solvers
│   │   └── base.h            # Solver base interface
│   ├── diff/                 # Differentiation mechanisms
│   │   ├── auto_diff.h       # Automatic differentiation (Jet-based)
│   │   ├── jet.h             # Dual numbers / Jet class implementation
│   │   ├── num_diff.h        # Finite-difference numerical differentiation
│   │   └── gradient_check.h  # Mathematical derivative verification utilities
│   ├── losses/               # Loss functions & M-estimators
│   │   ├── norms.h           # L1, L2, squared L2
│   │   ├── robust_norms.h    # Huber, Cauchy, Tukey, etc.
│   │   └── activations.h     # Activation functions
│   └── 3rdparty/             # Adapters for Sophus, Lie++, Ceres
├── tests/                    # Catch2 v3 unit test suite
├── benchmarks/               # Performance benchmarks (Catch2 & Ceres comparison)
├── examples/                 # Real-world usage examples (gravitational lensing, triangulation)
├── docs/                     # Documentation (architecture, style, guidelines, API)
├── cmake/                    # Modular CMake configuration files
├── pixi.toml                 # Pixi environment & dependency manager
└── .clang-format             # Code formatting rules (2-space, Google-based)
```

---

## 2. Basic Software Development Guidelines

1. **KISS & Readability First**: Optimization mathematics can be complex. Avoid speculative abstraction; keep implementations clear, concise, and mathematically self-evident.
2. **Single Responsibility Principle (SRP)**:
   - A *loss function* evaluates scalar metrics and its derivatives.
   - A *step solver* computes the linear update step $\delta x$.
   - An *optimizer* governs the trust region, damping parameter $\lambda$, step acceptance, and stopping criteria.
3. **Defensive Numerical Programming**:
   - Check for non-finite values (`std::isnan`, `std::isinf`) in gradients and Hessians.
   - Always map numerical breakdown to a clean [StopReason](include/tinyopt/stop_reasons.h) rather than producing undefined behavior or crashes.
4. **Regression Tests are Mandatory**:
   - If a solver, optimizer, loss, matrix backend, or autodiff path breaks or regresses, add a dedicated regression test covering the failing mode before shipping.
   - For backend-specific bugs (dense vs. sparse, CPU vs. macOS, robust losses vs. plain residuals), include a minimal reproducer in the relevant test file and keep it narrow but exact to the failure.
   - Do not close a bugfix by only changing production code; the failing scenario must be exercised by at least one test.
5. **Zero Compiler Warnings**: No warning will be tolerated under `-Wall -Wextra -Werror`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [julien-michot/tinyopt](https://github.com/julien-michot/tinyopt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
