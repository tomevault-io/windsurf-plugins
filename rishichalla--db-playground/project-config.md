---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.

## What this project is

A from-scratch, educational implementation of database internals in Rust. The
goal is to understand how real databases work under the hood — not to build a
production datastore. Currently the codebase implements a **disk-backed,
paged B+ Tree index**, plus the trait skeleton for a `Database` that will
eventually sit on top of it.

## Layout

```
src/
  lib.rs        - Crate root. Declares `pub mod btree` and the `Database<Key, Record>`
                   trait (insert/remove/update/lookup), which nothing implements yet.
  btree/mod.rs   - The entire B+ Tree implementation and its tests (~650 lines,
                   single file, no submodules).
```

Everything currently lives in `src/btree/mod.rs`: key types, page/serialization
logic, the tree node structure, insertion/traversal logic, and tests. There is
no separate `Database` implementation yet — `lookup_record`/`insert_record`/etc.
are not wired to `BPlusTree` yet.

## Core architecture (src/btree/mod.rs)

**Paging model**
- The index is a single flat file split into fixed-size `PAGE_SIZE` (4096-byte)
  pages, addressed by `PageOffset(usize)` (a page number, not a byte offset).
- Page 0 is reserved for `BPlusTreeInner` (tree metadata: root offset, size,
  depth, next free page, free list). `PageOffset::inner()` returns this
  reserved offset, and writing/reading a `TreeNode` there is treated as a bug
  (`IndexingError::MetadataOverwrite`).
- `PageOffset::default()` is `1`, since page 0 is metadata.
- New pages come from `BPlusTree::get_insertion_point`, which prefers reusing
  freed pages (`inner.free_list`) before bumping `inner.next_page`.

**Serialization**
- `bincode` 2.x is used directly (not serde) via manual `Encode`/`Decode`
  impls where custom framing is needed (see `StringKey`). Standard
  `#[derive(Encode, Decode)]` is used everywhere else.
- Encoding uses a fixed configuration: little-endian, fixed-width ints, capped
  at `PAGE_SIZE` (`PageOffset::bincode_config()`). Writes that would exceed a
  page are rejected (`IndexingError::WritePageOverflow`) rather than spanning
  multiple pages.
- `PageOffset::write_page` / `read_page` are the only places that touch the
  file directly (seek to `offset * PAGE_SIZE`, encode/pad/write, or read/decode).

**Caching**
- `BPlusTree.node_cache` is an `LruCache<PageOffset, TreeNode<Key>>` sized to
  hold ~4MB of pages (`LRU_CACHE_SIZE`), wrapped in a `RefCell` since lookups
  need mutable cache access (to record recency) through a `&self` API.
- `PageOffset::load_node` is the single entry point for fetching a node: check
  cache, else read from disk and populate the cache.

**Tree structure**
- `TreeNode<Key>` is generic over the key type and used for both branch and
  leaf nodes; `variant: TreeNodeVariant` (`Leaf` | `Branch`) distinguishes
  them. `splits: Vec<TreeNodeChild<Key>>` holds `(key, child)` pairs, where
  `child` is a `RecordRow` (leaf) or `PageOffset` (branch) stored as a raw
  `usize` (no enum — the variant on the node tells you how to interpret it).
- Nodes carry `left`/`right` sibling pointers (for sequential leaf traversal)
  and a `parent` pointer (for upward traversal on split/insert).
- `TreeNode::max_splits()` computes fan-out from `PAGE_SIZE` and
  `size_of::<Key>()`, so bigger keys (e.g. `StringKey<512>`) yield shallower
  fan-out and deeper trees — this is deliberately exploited in
  `test_deep_tree` to force multi-level splits.
- `IndexKey` is a marker trait (`Debug + Ord + Encode + Decode<()> + Clone`)
  blanket-implemented for anything that satisfies the bounds, used to keep
  trait bounds short elsewhere.
- `StringKey<const MAX_SIZE: usize>` wraps `arrayvec::ArrayString` to give a
  fixed-size, `Ord`-able string key, with hand-written `Encode`/`Decode` that
  writes a length prefix followed by a zero-padded fixed-size buffer (needed
  because `bincode`/`ArrayString` don't support this framing out of the box).

**Transactional writes**
- Mutations (insert, and future remove) never touch disk or the live cache
  directly. They operate on cloned/loaded nodes and accumulate results in a
  `HashMap<PageOffset, TreeNode<Key>>` (`mutated_nodes`) plus a modified copy
  of `BPlusTreeInner`. Using a `HashMap` keyed by `PageOffset` guarantees at
  most one final version per page per transaction.
- `BPlusTree::commit_transaction` is the only place that performs the actual
  disk writes for a set of changes: it checkpoints the previous state,
  attempts all writes, and rolls back to the checkpoint on any failure. Only
  on full success are the in-RAM `inner` and node cache updated. See the
  `IndexingError` variants (`Reversion`, `Io`, etc.) for the failure/rollback
  contract.
- `TreeNode::insert_item` implements the recursive insert/split/promote logic
  entirely against `mutated_nodes` and a mutable `inner`, so it can be unit
  tested and reasoned about without any I/O.

## Coding style conventions to follow

- **Extensive doc comments (`///`) on nearly every item** — structs, fields,
  enum variants, and functions all get a comment explaining *why*/*what it
  guarantees*, not just a restatement of the name. Match this density when

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RishiChalla/db-playground](https://github.com/RishiChalla/db-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
