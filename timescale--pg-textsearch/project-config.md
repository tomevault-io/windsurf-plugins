---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

pg_textsearch is a Postgres extension providing full-text search with BM25
ranking. (Internal name: Tapir) It implements a memtable-based architecture
similar to LSM trees, with in-memory structures that spill to disk segments
for scalability.

**Current Version**: 1.2.0-dev

**Postgres Version Support**: 17, 18 (tested in CI)

**Schema Organization**: Currently uses public schema. Future versions may
consider a dedicated `pg_textsearch` schema for cleaner namespace management.

## Important Notes for Development

- **pg_textsearch requires shared_preload_libraries** - Add `pg_textsearch`
  to `shared_preload_libraries` in postgresql.conf and restart the server
  before CREATE EXTENSION. A server restart is also required after updating
  the binary (e.g., after `make install` during development).

- **Every test failure matters.** We keep CI green on main. Do not
  dismiss failures as "pre-existing" — they almost never are. If a
  test fails, investigate and fix the root cause. If you believe a
  failure is unrelated to your changes, verify by checking out main
  and running the same test. Even if it does reproduce on main, it
  still needs to be investigated and fixed, not ignored.

## Core Architecture

### Storage Architecture

The index uses a hybrid storage approach:
- **Memtable**: In-memory inverted index with term dictionary and posting
  lists, stored in shared memory for concurrent access
- **Segments**: Immutable disk-based structures using V2 block storage format
  with skip lists for efficient top-k queries

### Query Optimization

- **Block-Max WAND (BMW)**: Top-k query optimization that uses block-level
  upper bounds to skip non-contributing blocks
- **Skip Lists**: Segment format includes skip entries for fast block seeking

### Source Code Structure

The `src/` directory is organized in layers (see CONTRIBUTING.md for
details):

- **Layer 1 (Postgres interface):** `access/` (AM handler, build,
  scan, vacuum), `types/` (bm25query, bm25vector), `planner/`
  (optimizer hooks, cost estimation)
- **Layer 2 (Index coordination):** `scoring/` (BM25, Block-Max
  WAND), `index/` (state lifecycle, registry, metapage, posting
  source abstraction)
- **Layer 3 (Storage):** `memtable/` (in-memory inverted index),
  `segment/` (on-disk segments, merge, compression)
- **Cross-cutting:** `debug/` (dump utilities), `mod.c` (init)

### Data Types

- `bm25vector` - Stores term frequencies with index context for BM25 scoring
  - Format: `"index_name:{lexeme1:freq1,lexeme2:freq2,...}"`
- `bm25query` - Represents queries for BM25 scoring with optional index context
  - Format: `"query terms"` or `"index_name:{query terms}"` (via to_bm25query)

## Development Commands

### Building and Testing
```bash
make                   # build extension
make install           # install to Postgres
make test              # run SQL regression tests only
make installcheck      # run all tests (SQL + shell scripts)
make test-all          # same as installcheck

# Run a single test
$(pg_config --pgxs | xargs dirname)/../../src/test/regress/pg_regress \
  --inputdir=test --outputdir=test basic  # runs basic.sql only

# Individual test types
make test-concurrency  # multi-session concurrency tests
make test-recovery     # crash recovery tests
make test-segment      # multi-backend segment tests
make test-stress       # long-running stress tests
make test-shell        # run all shell-based tests
make test-local        # run tests with dedicated Postgres instance
make expected          # generate expected output from test results
```

### Debug Builds
```bash
# Enable debug index dumps (add to Makefile or command line)
make PG_CPPFLAGS="-I$(pwd)/src -g -O2 -DDEBUG_DUMP_INDEX"
```

### Code Quality
```bash
make format            # auto-format C code with clang-format
make format-check      # check C code formatting (alias: lint-format)
make format-diff       # show formatting differences
make format-single FILE=path/to/file.c  # format specific file
```

## Configuration

### GUC Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `pg_textsearch.default_limit` | Default limit for queries without LIMIT | 1000 |
| `pg_textsearch.log_scores` | Log BM25 scores during scans | false |
| `pg_textsearch.log_bmw_stats` | Log BMW blocks scanned/skipped | false |
| `pg_textsearch.memory_limit` | Hard cap on memtable DSA memory (soft limits derived internally) | 2GB |
| `pg_textsearch.bulk_load_threshold` | Terms/xact to trigger spill | 100000 |
| `pg_textsearch.memtable_spill_threshold` | Posting entries to trigger spill (deprecated) | 32000000 |
| `pg_textsearch.segments_per_level` | Segments before compaction | 8 |
| `pg_textsearch.compress_segments` | Enable compression for new segment blocks | true |


### Index Options

| Option | Description | Default |
|--------|-------------|---------|
| `text_config` | Postgres text search configuration | (required) |
| `k1` | BM25 term frequency saturation | 1.2 |
| `b` | BM25 length normalization | 0.75 |

## Test Structure

- **SQL tests** (`test/sql/`): Core functionality, BM25 scoring (scoring1-6),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timescale/pg_textsearch](https://github.com/timescale/pg_textsearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
