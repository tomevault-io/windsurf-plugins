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

**Current Version**: 1.4.0-dev

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

- **Physical replication**: All page mutations are WAL-logged via
  `GenericXLog` records. There is no custom resource manager;
  pg_textsearch does not register an rmgr. Stock PostgreSQL replay
  reconstructs every page on a streaming standby or during crash
  recovery — including the on-disk memtable chain pages, segment
  pages, and the metapage. This is what lets PostgreSQL's
  single-page WAL-redo helper (and any other no-extension-load
  replay context) work without loading `pg_textsearch.so`.
  **The on-disk memtable design is spec'd at
  [`docs/memtable_v2.md`](docs/memtable_v2.md); read it before
  changing the write/read/spill flow.** Closes #345, #349,
  #350, #374.

- **Standby-safe segment reclaim (#380)**: A segment merge does not
  free the displaced source pages to the FSM immediately. Doing so is
  safe on the primary but unsafe for in-flight hot-standby queries,
  because there is no custom rmgr to resolve recovery conflicts during
  replay. Instead, displaced pages are *parked* in a WAL-logged
  (`GenericXLog`) tombstone chain off the metapage (`pending_free_head`),
  stamped with the merge's `FullTransactionId`. They return to the FSM
  only once a later VACUUM (or the next merge) observes that the stamp
  precedes `GetOldestNonRemovableTransactionId` — the standby-safe
  reclaim horizon. **`hot_standby_feedback = on` is required** on hot
  standbys serving queries, so their oldest snapshot holds the
  primary's horizon back until they finish reading the pages. Observe
  the parked count with `bm25_pending_free_pages(index_name)`.

## Core Architecture

### Storage Architecture

The index uses an LSM-like layered storage approach:
- **Memtable (L0)**: An on-disk paged structure stored in the index
  relation itself. Writes append doc records (ctid + length +
  packed bm25vector bytes) to a tail page chained off the
  metapage. Mutations are WAL-logged via `GenericXLog`.
  See [`docs/memtable_v2.md`](docs/memtable_v2.md).
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
- **Layer 3 (Storage):** `memtable/` (on-disk paged L0 — chain of
  doc-record pages mutated under buffer locks, WAL-logged via
  `GenericXLog`; see [`docs/memtable_v2.md`](docs/memtable_v2.md)),
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
make installcheck      # run SQL regression tests
make test-all          # run all tests (SQL + shell scripts)

# Run a single test
$(pg_config --pgxs | xargs dirname)/../../src/test/regress/pg_regress \
  --inputdir=test --outputdir=test basic  # runs basic.sql only

# Individual test types
make test-concurrency  # multi-session concurrency tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timescale/pg_textsearch](https://github.com/timescale/pg_textsearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
