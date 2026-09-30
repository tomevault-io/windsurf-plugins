---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Code analysis

- Always prefer Rust LSP to grep when exploring the code.
- For “go to definition”, use LSP goToDefinition, not Grep.
- For “find references”, use LSP findReferences, not Grep.
- For type information or docs, use LSP hover.
- Use Grep/Glob only for discovery:
  - finding files
  - searching plain-text patterns
  - locating candidate symbols before LSP
- After identifying the relevant Rust file/symbol, switch to LSP for navigation and understanding.
- Do not use Grep as a substitute for semantic reference search in Rust code when LSP is available.

## Permissions

- Do not ask permissin to run shell commands "with consecutive quote characters at word boundaries or word start"; to run compound commands (piped, chained with `&&`/`;`, or using subshells); any commands involving `git`.

## Commands

```sh
# Build all workspace crates
cargo build

# Build in release mode
cargo build -r

# Run all tests
cargo test

# Run tests for a specific crate
cargo test -p molar
cargo test -p molar_membrane

# Run a single test by name
cargo test -p molar <test_name>

# Run tests with output shown
cargo test -p molar -- --nocapture

# Check compilation without building
cargo check

# Build documentation
cargo doc --open

# Build Python bindings (from molar_python directory)
cd molar_python && maturin build -r && python -m pip install .
```

## Architecture

MolAR is a Cargo workspace (**Rust edition 2024**, MSRV 1.96) with these crates:

| Crate | Purpose |
|---|---|
| `molar` | Core library: SoA atom storage, selections, IO, topology, analysis tasks |
| `molar_gromacs` | Gromacs TPR support via a runtime-`dlopen`ed plugin (built only when Gromacs env vars are set; no compile-time dependency on Gromacs) |
| `molar_ff` | Force-field atom typing (GAFF/GAFF2) and partial charges (espaloma) |
| `molar_membrane` | Lipid membrane analysis (lipid order, curvature, etc.) |
| `molar_bin` | CLI utility (`last`, `rearrange`, `solvate`, `tip3to4` commands) |
| `molar_python` | Python bindings via PyO3/maturin (wheel: `pymolar`) |

All file formats are **pure Rust** (the former `molar_molfile` VMD-plugin crate has been removed).
PowerSASA is an external git dependency, not a workspace crate.

### Core data model (`molar/src/`)

- **`Topology`** (`topology.rs`) — molecules, `atoms: AtomStorage`, and `bonds: BondStorage`; usually read once from file
- **`AtomStorage`** (`atom_storage.rs`) — **Struct-of-Arrays** atom storage: one column per property.
  Ten always-present *core* columns (`name`, `resname`, `resid`, `resindex`, `atomic_number`, `mass`,
  `charge`, `chain`, `bfactor`, `occupancy`) plus four *optional* force-field/chemistry columns
  (`type_name`, `type_id`, `formal_charge`, `flags`) stored as `Option<Vec<T>>` — a `None` column costs
  nothing; when present it is full-length. Atoms are accessed through the borrowed **proxies**
  `AtomRef` / `AtomRefMut` (a two-word `{storage, index}` handle) — there is **no `&Atom`** to borrow.
- **`Atom`** (`atom.rs`) — the owned, densely-packed atom *row*, retained as the detached
  construction/interchange type (builders, IO readers, `From<&AtomLike>`); `AtomStorage::push`
  scatters it into the columns. `AtomFlags` holds the ring/aromatic bits (no longer packed into `type_id`).
- **`AtomLike`** (read getters) / **`AtomLikeMut`** (setters) — the atom interface, implemented by
  `Atom`, `AtomRef`, and `AtomRefMut`. Getters for the four *optional* properties return `Option`
  (e.g. `get_type_name() -> Option<&str>`); `charge` is the partial/working charge, `formal_charge`
  is the integer formal charge (kept separate).
- **`BondStorage`** (`bond_storage.rs`) — **Struct-of-Arrays** bond storage, same discipline as
  `AtomStorage`: an always-present pair column (`u32` internally, `usize` at every API boundary —
  caps a system at 4·10⁹ atoms) plus an *optional* `Option<Vec<BondOrder>>` order column, absent for
  connectivity-only sources (PDB CONECT / GRO / TPR) so MD systems allocate nothing for it. Bonds are
  read through the borrowed **`BondRef`** proxy — there is **no `&Bond`** to borrow. The owned
  `Bond` row (`bond.rs`) is the detached construction type; `BondStorage::push` scatters it.
- **`BondAdjacency`** (`bond_storage.rs`) — the per-atom bonded-neighbor index (compressed rows),
  cached inside `BondStorage`. `get_adjacency()` is cheap and parallel-safe; `ensure_adjacency(n_atoms)`
  builds it (a plain `&mut` field, **not** a `OnceCell` — interior mutability would cost `Topology` its
  `Sync`-through-`&` sharing across rayon). Structural change invalidates it, but **`set_order` does
  not** — that asymmetry is the point of splitting the columns. Anything changing the *atom count*
  must call `invalidate_adjacency()`, since `offsets` is sized `n_atoms + 1`.
  Also usable standalone via `BondAdjacency::build(n, pairs)`, which is how `molar_ff` indexes a
  remapped local subgraph. **Neighbor order within an atom's run is a guaranteed invariant**
  (ascending bond index) — the GAFF port indexes neighbors positionally and truncates to the first

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yesint/molar](https://github.com/yesint/molar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
