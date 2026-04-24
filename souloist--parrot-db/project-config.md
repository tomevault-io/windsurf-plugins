---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Parrot-db is a toy LMDB-inspired key-value storage engine in Python. It supports MVCC transactions with snapshot isolation, and is being developed in stages toward full WAL-based durability and B+ tree indexing.

**Current Stage:** 5 (Freelist, page reclamation, and compaction) - Complete

See `storage-engine-spec.md` for the full staged development plan and `progress.md` for current status.

## Commands

```bash
uv sync                      # Install dependencies
uv run pytest                # Run all tests
uv run pytest tests/test_models.py  # Run model tests only
uv run pytest tests/test_pager.py   # Run storage tests only
uv run ruff check .          # Lint
uv run ruff format .         # Format
uv run python client.py      # Interactive REPL (legacy)
uv run python tools/db_inspect.py --db ./tmp/dev.db --summary  # Inspect database
uv run python tools/db_copy.py source.db dest.db              # Copy database
uv run python tools/db_copy.py source.db dest.db -c           # Copy with compaction
```

## Project Structure

```
parrot-db/
├── models/                  # Pydantic models for on-disk structures
│   ├── storage.py          # KeyValue, PageHeader, PageType
│   ├── wal.py              # WALEntry, WALOperation
│   └── metadata.py         # DBMetadata, MAGIC
├── storage/                 # Page-based storage layer
│   ├── pager.py            # Pager: page I/O, allocation, file layout
│   ├── pages.py            # Page types: Header, Meta, Leaf, Branch, Freelist
│   ├── freelist.py         # In-memory freelist for page reuse
│   └── btree.py            # Copy-on-write B+ tree implementation
├── tools/
│   ├── db_inspect.py       # Database inspection CLI tool
│   └── db_copy.py          # Database copy with optional compaction
├── txn/                     # Transaction layer
│   └── transaction.py      # ReadTransaction, WriteTransaction
├── tests/
│   ├── test_models.py      # Model serialization tests
│   ├── test_pager.py       # Storage layer tests
│   ├── test_btree.py       # B+ tree tests
│   ├── test_transactions.py # Transaction and MVCC tests
│   ├── test_freelist.py    # Freelist and page reclamation tests
│   └── test_db.py          # Database tests (legacy)
├── parrot_db.py            # ParrotDB: main database class
├── storage-engine-spec.md  # Staged development plan
└── progress.md             # Current progress tracker
```

## Models (Stage 1)

All models use Pydantic with `struct`-based binary serialization. Each model file contains:
- Struct format reference in the module docstring
- Format constants (e.g., `KEY_VALUE_FMT = "<IIQ"`)
- `to_bytes()` / `from_bytes()` methods

| Model | File | Purpose |
|-------|------|---------|
| `KeyValue` | storage.py | Key-value entry with MVCC version |
| `PageHeader` | storage.py | 9-byte page header (type, id, checksum) |
| `WALEntry` | wal.py | WAL record (op, key, value, txn_id, timestamp) |
| `DBMetadata` | metadata.py | 32-byte file header (magic, version, page_size, etc.) |

## Storage Layer (Stage 2)

Page-based storage with dual meta pages for atomic commits.

**File Layout:**
- Page 0: Header page (magic `PRRT`, version, page_size)
- Page 1: Meta page 0 (txn_id, root_page_id, freelist_page_id)
- Page 2: Meta page 1 (alternating meta page)
- Page 3+: Data pages (freelist, branch, leaf)

**Key Components:**

| Component | File | Purpose |
|-----------|------|---------|
| `Pager` | pager.py | Page I/O, allocation, file management |
| `HeaderPage` | pages.py | Database file header with magic bytes |
| `MetaPage` | pages.py | Root pointers and txn_id for atomic commits |
| `LeafPage` | pages.py | B+ tree leaf with key-value cells |
| `BranchPage` | pages.py | B+ tree internal node with separators |
| `FreelistPage` | pages.py | Persisted list of free page IDs |
| `Freelist` | freelist.py | In-memory free page tracking with MVCC support |

**Checksum:** All pages use CRC32 checksums computed over the entire page content.

## Freelist and Page Reclamation (Stage 5)

MVCC-aware page management with deferred freeing for snapshot isolation.

**Deferred Freeing:**
Pages orphaned by writes can't be immediately freed if readers might still reference them. The freelist tracks:
- **Free pages**: immediately available for allocation
- **Pending-free pages**: keyed by txn_id, released when oldest reader advances past that txn_id

**Page Reclamation Flow:**
1. Write transaction modifies tree, creating new pages (CoW)
2. On commit, orphaned pages (old tree - new tree) are marked pending-free
3. When readers complete, `release_pending()` moves pages to free set
4. Future allocations reuse free pages before extending file

**Database Copy and Compaction:**

```python
# Raw copy (preserves layout, includes holes)
db.copy("backup.db")

# Compact copy (contiguous pages, smaller file)
db.copy("compact.db", compact=True)
```

CLI tool:
```bash
uv run python tools/db_copy.py source.db backup.db        # Raw copy
uv run python tools/db_copy.py source.db compact.db -c    # Compact copy
```

**Key Components:**

| Method | Location | Description |
|--------|----------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Souloist) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
