---
trigger: always_on
description: Working notes for the PDX repo.
---

# AGENTS.md

Working notes for the PDX repo.

## What it is

Fast IVF-based similarity search for high-dim vector embeddings, on `float32` **or quantized** vectors (`sq8`) — you pass `float32`, it quantizes internally. Faster than FAISS at equal quality (~1M×1536 in seconds). Header-only **C++17** + **Python bindings**; CPUs (ARM + x86). Paper: https://arxiv.org/html/2503.04422.

## Core idea: Efficient memory layout

PDX is a data layout that transposes vectors in a column-major order. This layout unleashes the true potential of dimension pruning. Pruning means avoiding checking all the dimensions of a vector to determine if it is a neighbour of a query, accelerating index construction and similarity search by factors. Our main pruner is based on ADSampling (paper: https://dl.acm.org/doi/pdf/10.1145/3589282), which is almost-lossless (< 0.005 recall loss). 

Docs: `README.md`, `INSTALL.md`, `BENCHMARKING.md`, `CONTRIBUTING.md`, `examples/README.md`.

### The data layout

PDX is a transposed layout (a.k.a. columnar, or decomposed layout), meaning that the dimensions of different vectors are stored sequentially. This decomposition occurs within a block (e.g., a cluster in an IVF index). The layout evolved from the one presented in the initial paper to reduce random access, and adapted it to work with 8-bit.

#### float32
For float32, the first 25% of the dimensions are fully decomposed. We refer to this as the "vertical block." The rest (75%) are decomposed into subvectors of 64 dimensions. We refer to this as the "horizontal block." The vertical block is used for efficient pruning, and the horizontal block is accessed on the candidates that were not pruned. This horizontal block is still decomposed every 64 dimensions. The idea behind this is that we still have a chance to prune the few remaining candidates every 64 dimensions.

The split is computed by `GetPDXDimensionSplit()` in `include/pdx/common.hpp`: the 25/75 rule holds for `d > 128`; for `d <= 128` the proportions flip (75% vertical), the horizontal block is rounded to a multiple of `H_DIM_SIZE = 64`, and for `d <= 64` everything is vertical. The `static_assert`s next to it are the reference table.

#### 8 bits
Smaller data types are not friendly to PDX, as we must accumulate distances on wider types, resulting in asymmetry. We can work around this by changing the PDX layout. For 8 bits, the vertical block is decomposed every 4 dimensions. This allows us to use dot-product instructions (VPDPBUSD on x86 and UDOT/SDOT on NEON) to calculate L2 or IP kernels while still benefiting from PDX. The horizontal block remains decomposed every 64 dimensions.


## Index types:
All in `include/pdx/indexes/`, templated on `Quantization` (`F32`/`U8`) and sharing `IPDXIndex`
(`ivf_vanilla.hpp`); the search loop itself is `PDXearch` in `searcher.hpp`.
- Vanilla IVF: Plain $k$-means partitioned centroids — `PDXIndex` (`ivf_vanilla.hpp`, storage `IVF` in
  `ivf_core.hpp`). Python: `IndexPDXIVF` / `IndexPDXIVFSQ8`.
- Tree IVF: A layer of mesoclusters is added on top of the plain IVF centroids, where PDX-pruning is also
  applied — `PDXTreeIndex` (`ivf_tree.hpp`, storage `IVFTree`). Python: `IndexPDXIVFTree` /
  `IndexPDXIVFTreeSQ8` (the fastest, README's default).

Serialization / benchmark ids follow `PDXIndexType` in `common.hpp` (`pdx_f32`, `pdx_u8`, `pdx_tree_f32`, `pdx_tree_u8`).

## Resumable search (cursor)

`PDXearch<Q>::IterativeSearch<FILTERED>` allows for: i) concurrent queries on one index, ii) resume a search. The API of a resumable search is: `Next(n)`: probes the next n clusters ranked once at `Begin`; `Done()`: the clusters are exhausted. 

Single-shot `Search`/`FilteredSearch` are thin wrappers: a non-thread-safe `TopKHeap`, one cursor over the n_probe-clamped ranking. Note: The tree's meso-cluster (L0) layer is not supported by cursors or by `FilteredSearch`. Both rank all leaf centroids flat, so a tree cursor is a vanilla IVF search over the tree's leaves.

## Maintenance (SPFresh-like appends/deletes)

Every index implements `Append(row_id, embedding)` / `Delete(row_id)` `PDXTreeIndex` additionally keeps the meso-cluster layer (L0) in sync. The leaf-level helpers they share live in `indexes/ivf_utils.hpp`. 
- **Append**: normalize+rotate → nearest centroid (vanilla: exact scan of all centroids; tree: PDX search over L0). Centroids never move on a plain append.
- **Delete**: tombstone the slot (`DeleteEmbedding`), mark the mapping `DELETED_MARKER`, `CheckClusterHealth`. Search masks tombstones; `Save()` compacts them away.
- **DestroyAndMergeCluster**: swap-and-pop the dead cluster (fix `id`, centroid and mapping of the moved one), then `ReassignEmbeddings` (nearest centroid via a `skmeans::BatchComputer` GEMM) with merges disabled to avoid cascades.
- **Invariants**: `ReserveClusterSlotIfNeeded()` before holding a `cluster_t&` (splits `push_back`); every structural change ends with `ComputeClusterOffsets()` (the searcher sizes its buffers from `max_cluster_capacity` on each query); single writer thread.
- Knobs: `indexes/cluster.hpp` (`CAPACITY_THRESHOLD`, `MIN_CAPACITY_THRESHOLD`, `MIN_MAX_CAPACITY = 256`, so small clusters need 256 slots before they split). Split knobs: `common.hpp`.

## Verification gate (definition of done)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cwida/PDX](https://github.com/cwida/PDX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
