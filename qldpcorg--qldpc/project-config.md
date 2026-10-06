---
trigger: always_on
description: This guide is for people and agents changing qLDPC.
---

# Agent and contributor guide

This guide is for people and agents changing qLDPC.
For a human-facing explanation of the data model and package relationships, start with the [library map](docs/source/library_map.rst).
For exact signatures and per-object literature, use the source docstrings or the generated [API reference](https://qldpc.readthedocs.io/autoapi/index.html).

## What to trust when documents disagree

When sources disagree, use this order:

1. Current implementation and its invariant tests.
2. Public docstrings and the `__all__` lists in package `__init__.py` files.
3. The [library map](docs/source/library_map.rst) and executable [examples](examples/).
4. Historical discussion, issues, or review notes.

Re-derive behavior from the current branch before documenting or changing it.
Do not copy transient project history, machine-specific paths, or local-session instructions into production files.

## Public API and compatibility

- [`src/qldpc/__init__.py`](src/qldpc/__init__.py) exports subpackages rather than flattening their classes and functions.
- Each stable subpackage has an explicit `__all__` in its `__init__.py`.
  Preserve those import paths when moving implementation code.
- Treat imports from ordinary `qldpc.*` packages as public and keep them working.
  Use a tested `DeprecationWarning` shim for a necessary rename or move rather than breaking an import.
  For a renamed module-level name, see [`qldpc._util.get_deprecated_alias`](src/qldpc/_util.py).
- Everything under [`src/qldpc/experimental/`](src/qldpc/experimental/) is explicitly unstable and can change without a deprecation period.
  Do not infer that this weaker guarantee applies elsewhere.
- Keep complete lists of public symbols in `__all__` and AutoAPI.
  Human-written docs should explain what packages do and show representative tasks, not duplicate a class catalogue.
- A module should not use a private (underscore-prefixed) name from another module unless it is a narrowly shared internal helper or an intentional decoder-composition boundary.
  For example, `decoders/construction/resolution.py` calls private backend builders kept beside their implementations; they are not public APIs merely because the resolver imports them.
  Otherwise, needing a cross-module private name indicates that it should be public and documented.
  A test module may use private names of the module that it tests.

## Module organization and ordering

- Preserve a module's existing section headings and organization when adding code.
- Add a section when new functionality does not fit an existing one.
- Put important user-facing APIs first within modules and sections.
- Keep related public entry points together.
- Place private implementation helpers toward the bottom of the relevant section or module.
- Keep user-facing decode methods near the top of decoder classes.
- Place private helper methods after the public decode methods.

## Repository map

| Area | What it does | Tests and examples |
| --- | --- | --- |
| [`src/qldpc/codes/common.py`](src/qldpc/codes/common.py) | `AbstractCode`, `ClassicalCode`, `QuditCode`, and `CSSCode`; logicals, stabilizers, distance, concatenation, and error-rate interfaces | [`common_test.py`](src/qldpc/codes/common_test.py) |
| [`src/qldpc/codes/monte_carlo.py`](src/qldpc/codes/monte_carlo.py) | Fixed-weight sample allocation, rate estimation, and statistical uncertainty | [`monte_carlo_test.py`](src/qldpc/codes/monte_carlo_test.py) |
| [`src/qldpc/codes/code_capacity.py`](src/qldpc/codes/code_capacity.py) | Code-capacity detector error models, observable-decoder orchestration, and sector reuse | [`code_capacity_test.py`](src/qldpc/codes/code_capacity_test.py), [`common_test.py`](src/qldpc/codes/common_test.py) |
| [`src/qldpc/codes/classical.py`](src/qldpc/codes/classical.py) | Classical code families | [`classical_test.py`](src/qldpc/codes/classical_test.py), [`basics.ipynb`](examples/basics.ipynb) |
| [`src/qldpc/codes/quantum.py`](src/qldpc/codes/quantum.py) | Quantum, CSS, subsystem, product, and geometric code families | [`quantum_test.py`](src/qldpc/codes/quantum_test.py), [`bivariate_bicycle_codes.ipynb`](examples/bivariate_bicycle_codes.ipynb) |
| [`src/qldpc/codes/distance.py`](src/qldpc/codes/distance.py) | Exact binary classical and quantum distance enumeration | [`distance_test.py`](src/qldpc/codes/distance_test.py) |
| [`src/qldpc/abstract/`](src/qldpc/abstract/) | Groups, group rings, `RingArray`, semisimple linear algebra, and Wedderburn--Artin transforms | Co-located `*_test.py` files in the same directory |
| [`src/qldpc/math.py`](src/qldpc/math.py) | Symplectic and finite-field array helpers | [`math_test.py`](src/qldpc/math_test.py) |
| [`src/qldpc/objects.py`](src/qldpc/objects.py) | Pauli labels, graph nodes, Cayley complexes, and chain complexes | [`objects_test.py`](src/qldpc/objects_test.py) |
| [`src/qldpc/decoders/external/`](src/qldpc/decoders/external/) | Integrations and immediate builders for ldpc, PyMatching, Relay-BP, Tesseract, and Frontier | Co-located `*_test.py` files |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qLDPCOrg/qLDPC](https://github.com/qLDPCOrg/qLDPC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
