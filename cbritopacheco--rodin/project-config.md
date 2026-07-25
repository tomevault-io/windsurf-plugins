---
trigger: always_on
description: Rodin is a modular C++20 finite element framework. It is designed as a set of orthogonal subsystems that must remain composable:
---

# GitHub Copilot Instructions for Rodin

## What Rodin is

Rodin is a modular C++20 finite element framework. It is designed as a set of orthogonal subsystems that must remain composable:

- **Geometry** handles mesh entities, topology, transformations, and evaluation points.
- **QF** handles quadrature formulas.
- **Variational** provides the high-level form language and problem composition.
- **Assembly** handles low-level assembly logic.
- **Solver** provides solver abstractions.
- Optional modules (PETSc, MPI, etc.) extend core behavior without replacing it.

When adding code, preserve this separation. New functionality should fit into the existing architecture instead of bypassing it.

## How to think before writing code

Before implementing anything, ask:

1. Which existing Rodin module should own this responsibility?
2. Is this a local evaluation abstraction, a global assembly abstraction, or a user-facing composition abstraction?
3. Am I extending an existing Rodin pattern, or introducing a foreign mini-framework?
4. Can the user still build problems explicitly using Rodin's existing style?
5. Am I hard-coding a prototype assumption as if it were generic?

Do not optimize for the shortest patch if it conflicts with Rodin's structure.

## Core architectural rules

### 1) Reuse existing abstractions first

If Rodin already has a good abstraction, build on it instead of duplicating it.

Examples:
- Use `Geometry::Point` for geometric quadrature-point context.
- Use existing FE space / basis abstractions instead of hard-coding one element type.
- Use `Variational::Problem` composition style where appropriate.
- Prefer existing field-oriented abstractions over raw vector-only APIs unless unavoidable.

### 2) Keep local and global responsibilities separate

- Geometry / FE / quadrature decide where and how evaluation happens.
- Constitutive laws compute local material response.
- Integrators / assembly accumulate local contributions globally.
- Problem objects / user code compose those pieces.

Do not mix these layers.

### 3) Prefer composable building blocks over monolithic managers

Rodin's style is compositional: trial/test functions, integrals, boundary conditions, explicit `Problem` construction. New modules should preserve this style and avoid introducing "manager owns everything" patterns unless an existing module in the same problem class already does this.

### 4) Do not confuse a working prototype with a generic backend

A prototype that only works for one element, one quadrature rule, one constitutive law, or one geometric assumption must not be represented as a generic module. If the module is intended to be generic, remove prototype assumptions in code, not only in documentation.

## Distinguish core layers explicitly

When reasoning about finite element mechanics and PDE modules, keep these concerns distinct:

- **Geometry** (entities, mappings, points)
- **FE evaluation** (shape functions, gradients, quadrature evaluations)
- **Local constitutive data/laws** (pointwise state and response)
- **Global assembly** (integration and accumulation into global operators)

This separation is critical for maintainability and reusability.

## Preserve the "user builds the problem" philosophy

Prefer APIs that keep user-level problem composition explicit. Rodin users should still be able to assemble and solve by composing trial/test functions, integrals, boundary terms, and solvers directly. Avoid hiding problem definition behind opaque orchestration objects.

The canonical user-facing pattern looks like:

```cpp
Mesh mesh;
mesh = mesh.UniformGrid(Polytope::Type::Triangle, {16, 16});
mesh.getConnectivity().compute(1, 2);

P1 Vh(mesh);
TrialFunction u(Vh);
TestFunction  v(Vh);

Problem problem(u, v);
problem = Integral(Grad(u), Grad(v))
        - Integral(f, v)
        + DirichletBC(u, Zero());

Solver::SparseLU solver;
problem.solve(solver);
```

New modules must integrate naturally into this composition style.

## Module structure overview

```
src/Rodin/
├── Alert/              # Logging and error reporting (compiled)
├── Assembly/           # FE assembly engine (compiled)
├── Context/            # Execution contexts: Local, MPI (compiled)
├── FormLanguage/       # Expression template DSL base (compiled)
├── Geometry/           # Mesh, polytopes, points, connectivity (compiled)
├── IO/                 # I/O: MFEM, MEDIT, XDMF, HDF5 (compiled)
├── Math/               # Linear algebra, vectors, matrices (compiled)
├── QF/                 # Quadrature formulas (compiled)
├── Serialization/      # Serialization support (compiled)
├── Solver/             # Linear solvers interface (header-only)
├── Utility/            # General utilities (compiled)
├── Variational/        # FE spaces and weak formulations (compiled)
│   ├── P0/             # Piecewise constant
│   ├── P0g/            # P0 with gradient enrichment
│   ├── P1/             # Piecewise linear
│   └── H1/             # Arbitrary-order Lagrange
├── Advection/          # Lagrangian advection (header-only)
├── Eikonal/            # Fast Marching Method (header-only)
├── LinearElasticity/   # Linear elasticity integrators (header-only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cbritopacheco/rodin](https://github.com/cbritopacheco/rodin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
