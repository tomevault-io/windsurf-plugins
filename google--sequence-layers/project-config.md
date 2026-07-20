---
trigger: always_on
description: > [!IMPORTANT] Read `DESIGN.md` (§7) for the rationale behind this architecture.
---

# SequenceLayers Multi-Backend: Coding Guide

> [!IMPORTANT] Read `DESIGN.md` (§7) for the rationale behind this architecture.

## Scope

Design rules (🏛️) are universal and apply whenever extracting shared interfaces.
Toolchain rules (🔧) apply to files checked with pyrefly (currently `specs/` and
new files; pyrefly is available via `pyproject.toml`).

## Framework Status

1.  **JAX**: Stable, backward-compat required. Do not change public APIs (names,
    interfaces, type signatures) without explicit justification.
2.  **TF**: Deprecated. Ignore this directory and its conventions.
3.  **MLX**: In progress. See Porting Workflow below.

## Porting Workflow

When porting or refactoring across backends, you are likely in one of these
scenarios (or a **Full Port** combining them):

1.  **Interface Extraction**: JAX and MLX implementations exist but don't
    inherit from a shared interface. Find shared config fields, layer classes,
    methods, and arguments. Codify the shared interface in `specs/*.py`. Both
    backends should inherit from the spec class (in appropriate MRO order; see
    rule 4), provided it does not break JAX backward compatibility.
2.  **Test Unification**: JAX and MLX implementations exist but don't share
    tests. Given the unified interface in `specs/*.py`, refactor shared test
    logic into `specs/*_behaviors.py`. *Prefer JAX tests as the basis when they
    cover equivalent features.*
3.  **Full Port / Feature Porting**: A JAX-only layer needs an MLX port. **Start
    by abstracting the tests** (TDD): codify the interface in `specs/*.py` and
    tests in `specs/*_behaviors.py` (preferring JAX tests as basis), then create
    or update `mlx/*(_test).py`.
4.  **Backend-specific supersets**: When a backend implements extra features
    beyond the shared spec, common functionality goes in `specs/*_behaviors.py`,
    while backend-specific extensions stay in `<backend>/*_test.py`. Consider
    whether extended features could be generalized into the shared spec.

--------------------------------------------------------------------------------

## Design Rules 🏛️

### Architecture

1.  **Up-front readability**: Backend files must be self-contained. Re-declare
    all defaults, docstrings, function signatures, and Config fields. Users
    should never need to read `specs/` to understand a backend's API.
    -   *Exception*: Pure functions that are part of the contract all backends
        must fulfill (e.g., test utilities like `zip_longest`, `named_product`)
        may live in `specs/` and be aliased by backends.
2.  **Generics and specialization**: Spec classes (layers and Configs) are
    generic (e.g., over `DTypeT`, `SequenceT`). Backends specialize with
    concrete types.
3.  **Rigid signatures / LSP**: Match spec parameter names and signatures
    exactly. No `**kwargs`. Include all protocol parameters (e.g., `training:
    bool`) even if unused by a particular backend — this maintains Liskov
    Substitution Principle compliance.
4.  **MRO**: The abstract spec class should be the last one inherited. Example:
    `class StatelessEmitting(Emitting, spec.StatelessEmitting)`
5.  **Circular import prevention**: When submodules import root-level aliases
    from `__init__.py`, ensure all root-level alias imports are placed at the
    **top** of `__init__.py`, before importing any submodule classes.
6.  **Decoupled instantiation**: Use `Layer.from_config(config)` factory methods
    on the framework-specific class, not `Config.make(backend=...)`. Spec
    configs remain abstract.
7.  **Deferred initialization for stateless backends**: Backends without eager
    parameter allocation (e.g., MLX) should use lazy submodule creation within
    `_ensure_initialized` rather than maintaining a separate wrapper class. The
    public class accepts a `Config` and lazily creates its internal submodules
    on the first call to `layer()`.
8.  **Config specs nested**: In `specs/` files, `Config` classes are nested
    within the layer classes they configure, paralleling the structure in
    backend implementations.

### Testing

1.  **Behavior tests via inheritance**: `specs/*_behaviors.py` defines
    backend-agnostic test cases. `<backend>/*_test.py` inherits from these.
    -   No `abc.ABC` in behavior test classes (they won't be discovered by
        pytest since files are named `*_behaviors.py`, not `*_test.py`).
    -   No cross-importing between behavior files. Prefer duplicating small
        helpers or using shared bases in non-behavior modules.
    -   Inherit from `test_utils.SequenceLayerTest` (or similar shared base). In
        `<backend>/*_test.py`, subclass `test_utils.SequenceLayerTest` first
        (MRO convention).
2.  **Backend-native syntax in tests**: In `<backend>/*_test.py`, use
    backend-specific types (`jnp`, `sl.Sequence`, etc.). Import the backend as
    `sl` (e.g., `import sequence_layers.mlx as sl`).
3.  **Avoid `super()` in diamond test hierarchies**: When dealing with diamond
    inheritance (test base + backend-specific mock), `super()` calls can be
    brittle. Use explicit class delegation (e.g.,
    `backend_sl.types.Stateless.step(self, ...)`).
4.  **Capture `self.sl` before nested classes**: Capture `backend_sl = self.sl`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [google/sequence-layers](https://github.com/google/sequence-layers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
