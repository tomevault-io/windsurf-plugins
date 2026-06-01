---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Zig

```bash
# Type-check everything (fast — no codegen)
zig build check

# Run all tests
zig build test --summary all

# Format all source files in-place
zig build fmt

# Check formatting without modifying (used by CI)
zig fmt --check src/ bench/ build.zig tests/ examples/

# Run the co-authorship example
zig build example

# Run benchmarks
zig build bench

# Build and deploy docs locally (output: zig-out/docs/)
zig build docs

# Build the shared library for Python bindings
zig build lib -Doptimize=ReleaseFast

# Run Zig benchmarks
zig build bench
```

**Required Zig version:** `0.17.0-dev.242+5d55999d2` (set in `build.zig.zon`).

`zig build test` currently exits non-zero due to an issue with the `--listen=-` IPC protocol and the debug-level test logging — all tests actually pass. Run the test binary directly to confirm: `./.zig-cache/o/<hash>/test`.

### Python (requires [uv](https://github.com/astral-sh/uv))

```bash
# Install / sync all dev dependencies
uv sync

# Build the shared library and copy it into the package, then sync
just install-py

# Run Python tests
uv run pytest tests-python/ -v

# Lint and format check
uv run ruff check python/ tests-python/
uv run ruff format --check python/ tests-python/

# Format in-place
uv run ruff format python/ tests-python/
uv run ruff check --fix python/ tests-python/

# Build a platform wheel locally (runs hatch_build.py, which compiles Zig)
just wheel

# Run Python benchmarks (pyperf — statistical, mirrors Zig output)
uv run python bench-python/bench_pyperf.py
uv run python bench-python/bench_pyperf.py --fast   # quick pass
uv run python bench-python/bench_pyperf.py -o results.json
uv run python -m pyperf compare_to baseline.json results.json

# Run Python benchmarks with pytest-benchmark (quick table output)
uv run pytest bench-python/bench_pytest.py -v --benchmark-only

# Shortcuts
just install-py   # build lib + copy + uv sync
just test-py      # uv run pytest
just bench-zig    # zig build bench
just bench-py     # pyperf (statistical, save with -o results.json)
just bench-pytest # pytest-benchmark (quick table)
just lint         # ruff check + format check
just fmt          # ruff format + fix in-place
```

## Architecture

The entire library lives in a single file: `src/hypergraphz.zig`. Everything else is consumer code (tests, bench, examples).

### The generic type

`HypergraphZ(H, V, options)` is a comptime-parametric type. `H` is the hyperedge payload struct, `V` is the vertex payload struct. Both must be structs; `H` must have an integer field named by `options.weight_field` (default `"weight"`) — this is enforced with a comptime `assert`. The returned type contains all methods as `pub fn`.

### Internal storage

Each vertex and hyperedge stores a `*T` pointer (from a `MemoryPool`) plus an `ArrayList(HypergraphZId)` of relations:
- **Hyperedge relations**: ordered list of vertex IDs (may contain duplicates; order defines direction via consecutive pairs).
- **Vertex relations**: unordered list of hyperedge IDs (the reverse index).

Both use `AutoArrayHashMap(HypergraphZId, DataRelations)` so that `getAllVertices()` / `getAllHyperedges()` return IDs in deterministic insertion order — which is why any function that emits a sorted/canonical result iterates the map rather than a separate list.

### Two-phase model

After `init`, the graph is in **build phase**: mutations write only to the forward index (hyperedge → vertices). The reverse index (vertex → hyperedges) does not exist yet. Any query that needs the reverse index returns `HypergraphZError.NotBuilt`.

Calling `build()` constructs the reverse index in a single pass and switches to **query phase**. Subsequent mutations maintain the reverse index incrementally. A second `build()` call is only needed after a large batch of insertions where incremental maintenance would be too expensive.

### Method sections (in source order)

| Section | What lives here |
|---|---|
| Core | `init`, `deinit`, `clone`, `build`, `clear`, `createVertex/Hyperedge`, `getVertex/Hyperedge`, `getAllVertices/Hyperedges`, `countVertices/Hyperedges` |
| Mutations | Append/prepend/insert/delete vertices in hyperedges; `deleteHyperedge`, `deleteVertex` |
| Queries | `getHyperedgeVertices`, `getVertexHyperedges`, degree/indegree, adjacency, intersections, union, shortest path endpoints, topological sort |
| Traversal | `findShortestPath`, BFS/DFS, `getAllPaths`, `isConnected`, `randomWalk`, `getTransitiveClosure` |
| Algorithms | `computeCentrality`, `computePageRank`, `getInclusions`, `getNestednessProfile`, `isKUniform`, `isHypergraph`, `getConnectedComponents`, `getVertexNeighborhood`, `isComplete`, `findCutVertices` |
| Projections | `getDual`, `getSubhypergraph` (by vertices or hyperedges), `getCore` (vertex-degree k-core), `getCore(s,t)` (bipartite s,t-core), `expandToGraph`, `expandToStar`, `getLineGraph`, `toIncidenceMatrix`, `toIncidenceMatrixCOO`, `toLaplacian` |
| Codec | `save` / `load` with custom serialize/deserialize callbacks; `defaultSerialize` / `defaultDeserialize` for pointer-free types |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yamafaktory/hypergraphz](https://github.com/yamafaktory/hypergraphz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
