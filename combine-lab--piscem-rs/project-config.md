---
trigger: always_on
description: This is a Rust port of the C++ `piscem` bioinformatics tool for k-mer-based read mapping. The Rust implementation (`piscem-rs`) must produce **semantically equivalent** outputs to the C++ version (not byte-identical). It depends on `sshash-rs` (git dependency on `https://github.com/COMBINE-lab/sshash-rs.git`, branch `main`) for the compressed k-mer dictionary. A local checkout at `./sshash-rs/` is used for development and automatically picked up by Cargo.
---

# piscem-rs Development Context

## Project Overview

This is a Rust port of the C++ `piscem` bioinformatics tool for k-mer-based read mapping. The Rust implementation (`piscem-rs`) must produce **semantically equivalent** outputs to the C++ version (not byte-identical). It depends on `sshash-rs` (git dependency on `https://github.com/COMBINE-lab/sshash-rs.git`, branch `main`) for the compressed k-mer dictionary. A local checkout at `./sshash-rs/` is used for development and automatically picked up by Cargo.

The full implementation plan with C++ → Rust type mappings, architectural notes, and phased roadmap is in `implementation_plan.md`. Read it before starting new phases.

## Current Status

### Completed Phases

- **Phase 0**: Project bootstrap with CLI skeleton and parity harness scaffolding
- **Phase 1A–1E: Index data structures + build pipeline** — ContigTable (EF offsets + packed entries), RefInfo, ReferenceIndex, EqClassMap, end-to-end build from cuttlefish output
- **Phase 2: PoisonTable** — `AHashMap<CanonicalKmer, u64>` with fixed-seed ahash, serialization (`PPOIS01\0`), query methods
- **Phase 3: Mapping core** — ProjectedHits, PiscemStreamingQuery (sshash-rs wrapper + unitig-end cache), HitSearcher (PERMISSIVE/STRICT modes)
- **Phase 4: Mapping infrastructure** — `map_read<K, S>()` kernel, MappingCache, SketchHitInfo trait, RadWriter, Protocol trait
- **Phase 5: Protocol implementations + CLI** — Bulk/scRNA/scATAC mapping CLIs, ChromiumProtocol, custom geometry parser
- **Phase 6: Hardening** — UnitigEndCache (DashMap), overlap detection, genome binning, parity harness
- **Phase 7: Poison builder + CanonicalKmer** — `build-poison` CLI, CanonicalKmer newtype
- **Phase 8: scATAC parity** — Triple-file input, every-kmer mode, bin-based merge, 100% record parity
- **Phase 9: Idiomatic paraseq refactor** — Replaced custom crossbeam producer-consumer pipeline with paraseq's native `ParallelProcessor`/`PairedParallelProcessor`/`MultiParallelProcessor` traits. Eliminated intermediate `ReadPair`/`ReadTriplet` owned copies; reads processed in-place from paraseq buffers (zero-copy for single-line FASTQ). Per-thread stats flushed once via `on_thread_complete()` instead of per-chunk atomics.
- **Phase 10: Multi-file parallel decompression** — Switched from concatenated single-reader streams (`open_concatenated_readers` + `fastq::Reader`) to paraseq's `Collection` API (`fastx::Collection` with `CollectionType::Paired`/`Single`/`Multi`). Enables parallel decompression across multiple input file sets. Processor trait impls updated from `fastq::RefRecord` to `fastx::RefRecord`.

### Parity Status

| Mode | Dataset | Mapping Rate | Record-Level Parity |
|------|---------|-------------|-------------------|
| Bulk PE | gencode_pc_v44 (no poison) | 100% match (96.46%) | 100% (964,594/964,594) |
| Bulk PE | gencode_pc_v44 (with poison) | 100% match | 100% (961,505/961,505) |
| Bulk PE (strict) | gencode_pc_v44 | 100% match | 100% |
| Bulk SE | gencode_pc_v44 | 100% match | 83.65% (tie-breaking differences expected) |
| scRNA | SRR12623882 (Chromium V3) | 100% match | 100% |
| scRNA | PBMC 1k v3 (33.4M reads) | 100% match (86.64%) | 100% (28,968,858/28,968,858) |
| scATAC | 5M ATAC reads (hg38 k25) | 100% match (98.33%) | 100% (4,916,721/4,916,721) |

### Performance Status

Rust is **faster than C++** across both bulk and scRNA workloads (Apple Silicon M2 Max):

**Bulk PE** (1M reads, gencode v44):

| Threads | C++ | Rust | Ratio |
|--------:|----:|-----:|------:|
| 1 | 14.3s | 14.0s | 0.98x |
| 4 | 3.9s | 3.8s | 0.96x |
| 8 | 3.3s | 2.4s | 0.71x |

**scRNA** (PBMC 1k v3, 33.4M reads, Chromium V3, gencode v44, 237K refs):

| Platform | Threads | C++ | Rust | Ratio |
|----------|--------:|----:|-----:|------:|
| Apple Silicon M2 Max | 8 | 114s | 111s | 0.97x |
| x86-64 Linux | 8 | 55s | 47s | 0.85x |

Mapping counts are identical: 28,968,858 / 33,436,697 (86.64%) for both implementations.

Key optimizations applied:
- **AHashMap for hit_map**: Replaced `nohash-hasher` (identity hash) which caused pathological SwissTable H2 collisions with sequential transcript IDs (~38% regression on scRNA with 237K refs). `AHashMap` properly distributes hash bits for SwissTable SIMD probing.
- **AHashSet for observed_ecs**: Replaced standard `HashSet<u64>` (SipHash) with `AHashSet<u64>` matching C++ `ankerl::unordered_dense::set` performance.
- **rapidhash in sshash-rs**: Replaced ahash for MPHF and minimizer hashing. ahash switches algorithm when AES-NI is available (via `target-cpu=native`), silently breaking serialized indices. rapidhash is CPU-feature independent.
- **Optional UnitigEndCache**: Only scATAC uses the cache; bulk and scRNA pass `None`, avoiding DashMap overhead. This was the primary source of the x86-64 performance gap.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [COMBINE-lab/piscem-rs](https://github.com/COMBINE-lab/piscem-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
