---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**panpatch** is a C++ tool that uses pangenome graphs to patch fragmented assemblies into telomere-to-telomere (T2T) chromosomes. It supports three types of patches:
1. Filling gaps inside scaffolds (Ns)
2. Scaffolding disconnected contigs
3. Completing missing terminal telomeres (with `-T`)

The tool operates on chromosome graphs in `.vg` format created by minigraph-cactus, which include a reference contig (e.g., CHM13) and multiple assemblies.

## Building and Testing

### Build Commands
```bash
# Standard build
make

# Static build
make static

# Clean (fast - only removes panpatch binary and object files)
make cleanFast

# Clean (full - removes all built libraries)
make clean
```

### Running Tests
```bash
# Run the full test suite
make test
# This runs: cd test && python3 panpatchTest.py
```

The test suite uses `vg` tools to generate test graphs and validates output against truth files using `diff`.

### Building Dependencies
Dependencies are built automatically via the Makefile. The main dependency is `libbdsg-easy` which provides graph handling libraries (libbdsg, libhandlegraph, libsdsl, etc.). If `libbdsg` is missing, run `make` which will automatically build it via `deps/libbdsg-easy`.

## Code Architecture

### Core Components

**Entry Point (`panpatch_main.cpp`):**
- Command-line argument parsing
- Graph loading and initialization
- Multi-threaded execution setup (OpenMP)
- Reference path selection and validation
- Haplotype binning orchestration
- FASTA output generation

**Core Algorithm (`panpatch.cpp`):**
The patching algorithm follows this pipeline:
1. **Haplotype Matching** (`compute_overlap_identity`, `select_sample_covers`): Bins contigs by haplotype using windowed identity calculations, since input assemblies may not be trio-phased
2. **Anchor Finding** (`find_anchors`): Identifies nodes on the reference path where assembly paths start, end, or branch
3. **Path Threading** (`thread_intervals`): Searches left-to-right through anchors, greedily selecting highest-priority paths at each junction
4. **Smoothing & Extension** (`smooth_intervals`, `extend_intervals`): Merges adjacent intervals on the same path and extends to telomeres
5. **Validation** (`excise_bad_interior_grafts` then `revert_bad_patch`): First a partial-patch pass excises repeat-region-misjoin foreign interior grafts in place (splicing the target's own sequence back, keeping the rest of the patch) — judging each graft by content if it replaced real sequence (k-mer recovery, `--graft-recovery`/`--graft-min-bp`) or by flank anchoring if it filled an N-gap (`--min-flank`/`--flank-window`, since there is no replaced sequence to recapitulate). Then `revert_bad_patch` reverts the whole contig to the input if it still fails a quality check — too short (`--min-cover`), a same-sample interior splice or un-excisable foreign graft, a discarded target telomere, or (with `-T`) failed telomere validation. See the README *Why a patch is rejected* section for the full list and the messages each emits.

**Graph Representation:**
- Uses `bdsg::PathHandleGraph` interface from libbdsg for graph access
- Applies `ReferencePathOverlayHelper` for position-based queries
- Intervals represented as `tuple<step_handle_t, step_handle_t, bool>` where bool indicates reverse orientation

### Key Data Structures

- **Anchors**: `unordered_map<int64_t, int64_t>` mapping node IDs to reference positions
- **Intervals**: `vector<tuple<step_handle_t, step_handle_t, bool>>` representing path segments with orientation
- **Coverage Map**: `unordered_map<path_handle_t, double>` tracking identity scores between haplotypes

### Multi-threading

OpenMP is used for parallelization. Configure with:
- `-t/--threads N` command-line option
- OpenMP flags already set in Makefile: `-fopenmp -pthread`
- Use `get_thread_count()` to check active threads

## Usage Pattern

```bash
panpatch <graph.vg> [graph2.vg ...] -r <reference> -s <sample-to-patch> -s <patch-source-1> -s <patch-source-2> ...
```

Sample names come from minigraph-cactus seqfile (first column, without `.1/.2` haplotype suffixes). Priority is determined by order of `-s` flags.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glennhickey/panpatch](https://github.com/glennhickey/panpatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
