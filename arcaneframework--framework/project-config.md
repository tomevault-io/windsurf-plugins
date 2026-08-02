---
trigger: always_on
description: Provides a unified interface for large sparse linear solvers:
---

# Arcane Framework — Agent Instructions

## Repository Overview

Arcane is a development platform for **massively parallel unstructured 2D/3D mesh computation codes**. Written by CEA/IFPEN, it provides mesh management, data structures, parallelism (MPI/threads), I/O, and a service/plugin architecture for scientific computing applications. The repository is a CMake-based monorepo.

## Repo Structure

| Directory | Purpose |
|-----------|---------|
| `arccore/` | Core library — concurrency, serialization, mesh geometry, accelerators, message passing |
| `arcane/` | Main framework — mesh management, I/O, services, driver, test infrastructure |
| `alien/` | Linear algebra interface (Hypre, PETSc, Trilinos) with Arcane coupling |
| `arccon/` | Custom CMake build-system layer (macros, install dirs, .NET helpers) — no C++ code |
| `axlstar/` | AXL XML-to-C++/C# code generator (.NET/`dotnet` app) |
| `arctools/` | Documentation generator (`adoc/`) and Neo CLI tools |
| `dependencies/` | Git submodule — NuGet packages + ArcDependencies CMake finder scripts |

**Build order** (automatic via top-level CMake): `arccon` → `axlstar` → `arccore` → `arcane` → `alien`.

---

## Component Details

### arccore — Core Library

Foundation for all Arcane components. Organized into CMake components:

| Component | Purpose |
|-----------|---------|
| `base` | Core types, exceptions, error handling, memory management |
| `common` | Math utilities, string handling, time, hash, UUID |
| `collections` | Containers (DynamicArray, HashTable, etc.) |
| `concurrency` | Thread pools, barriers, atomics |
| `serialize` | Serialization framework for all Arcane types |
| `trace` | Logging and tracing system |
| `message_passing` / `message_passing_mpi` | MPI and shared-memory communication |
| `accelerator` / `accelerator_native` | Unified GPU accelerator API (CUDA/HIP/SYCL) |

- **C++20+** required. Minimum: GCC 11 or Clang 16.
- Components are added via `arccore_add_component_directory(<name>)`.
- Generated config headers: `build/arccore/arccore_config.h`, `build/arccore_version.h`.

### arcane — Main Framework

The top-level application framework built on arccore. Key sub-libraries:

| Library | Purpose |
|---------|---------|
| `arcane_core` | Core runtime, service/module base classes, AXL processing |
| `arcane_mesh` | Mesh entities (nodes, edges, faces, cells), connectivities, ghost layers |
| `arcane_impl` | Implementation utilities |
| `arcane_utils` | Common utilities |
| `arcane_ios` | I/O readers/writers (VTK, MED, Gmsh, XMF) |
| `arcane_driver` | `.arc` case-file driver |
| `arcane_geometry` | Geometric computations (ray intersection, etc.) |
| `arcane_hdf5` | HDF5 I/O support |
| `arcane_parallel` | MPI and thread parallelism wrappers |

Additional modules:
- `std/` — Standard services: checkpoint, mesh partitioning (Metis/PTScotch/Zoltan), mesh generators (Sod, Cartesian, HoneyComb), I/O services, post-processing, profilers (PAPI, OTF2)
- `accelerator/` — GPU-accelerated mesh operations. Provides mesh-aware GPU abstractions built on top of arccore's low-level accelerator primitives:
  - `RunCommandLoop` / `RunCommandEnumerate` — kernel launch commands that iterate over mesh entities (cells, faces, nodes) with device-side views
  - `VariableViews` / `MaterialVariableViews` — zero-copy device pointers into Arcane mesh variables, enabling kernels to read/write mesh data directly on GPU
  - `Views.h` / `SpanViews.h` — device-compatible array views for work-item-local data
  - `NumArray.h` — GPU-resident numerical arrays with host/device synchronization
  - Generic algorithms: `Filter` (selective copy), `Reduce` (parallel reduction), `Scan` (exclusive/inclusive scan), `Sort` (key-value sort), `Partitioner` (work partitioning)
  - `Atomic.h`, `LocalMemory.h`, `RunQueue.h` — low-level GPU synchronization and memory management
  - Backend-specific directories: `cuda/`, `hip/`, `sycl/` for runtime-specific code
- `aleph/` — Numerical linear algebra components
- `corefinement/` — Mesh refinement/coarsening
- `launcher/` — Parallel launch infrastructure
- `lima/` — Lima mesh format support
- `driver/` — `.arc` case-file execution driver
- `materials/` — Multi-constituent material support
- `cartesianmesh/` — Cartesian mesh with AMR

**Public API boundary:** Only `core/`, `materials/`, `utils/`, `launcher/`, `accelerator/`, `cartesianmesh/`, and `hdf5/` are public API. All other `arcane/` subdirectories (`mesh/`, `ios/`, `driver/`, `geometry/`, `std/`, `aleph/`, `corefinement/`, `lima/`, `parallel/`, etc.) are private — they may be installed to `include/` but are not stable and should not be used by external code.

### alien — Linear Algebra Interface

Provides a unified interface for large sparse linear solvers:

- **`standalone/`** — Standalone Alien library (C++17, MPI, BLAS). Plugins for Hypre, Trilinos, PETSc, Ginkgo.
- **`ArcaneInterface/`** — Adapter layer connecting Alien to Arcane meshes and variables.

### arccon — Build System Layer

Pure CMake project (no C++). Provides:
- `find_package(Arccon)` — dependency finder
- `arccon_install_directory()` / `arccon_dotnet_install_publish_directory()` — install helpers
- `.NET/msbuild` integration macros (`ArcconDotNet.cmake`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcaneframework/framework](https://github.com/arcaneframework/framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
