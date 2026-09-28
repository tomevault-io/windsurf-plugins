---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

IssunDB is an embedded graph database with vector and full-text search, written in Rust.
Priorities, in order:

1. Correct storage behavior: ACID transactions, adjacency consistency, and ID uniqueness.
2. Clear boundaries between the storage engine, query layer, vector and text indexes, and public facade.
3. Reproducible, benchmark-backed performance; no premature optimization before correctness is covered.
4. Idiomatic Rust: ownership, zero-cost abstractions, and `unsafe` only where necessary and documented.

## Core Rules

- Use English for code, comments, docs, and tests.
- Prefer small, focused changes over broad rewrites.
- Keep the workspace modular: `issundb-core` owns graph storage, `issundb-vector` owns vector search, `issundb-text` owns full-text search,
  `issundb-retrieval` owns hybrid retrieval, `issundb-cypher` owns the query layer, `issundb` is the public facade, and the consumer crates
  (`issundb-cli`, `issundb-rest`, `issundb-mcp`, `issundb-py`, `issundb-wasm`) use only the `issundb` facade. See Dependency Boundaries.
- Keep all mutable state inside `Graph` and `Storage`; do not introduce module-level `static mut` or `lazy_static` globals for runtime state.
- Writes are serialized via the `parking_lot::ReentrantMutex<()>` write lock on `Graph`; LMDB enforces the same constraint at the storage level. Do
  not bypass either.
- Add comments only when they clarify a non-obvious storage invariant, an LMDB lifetime constraint, or an algorithm kernel's ordering or
  duplicate-handling rule.
- Maintain the permissive license boundary of the workspace (MIT or Apache-2.0). Do not add dependencies or statically link libraries with copyleft,
  weak copyleft, or source-available licenses (such as GPL, MPL, or SSPL). Keep comparison or benchmarking harnesses that link to such external
  engines excluded from the root Cargo workspace.
- Format with `rustfmt` (`make format`) and lint with Clippy (`make lint`) before declaring a change done.

## Writing Style

- Write in simple, plain English. Use short sentences and everyday words.
- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "adjacency query" not "blazing adjacency query".
- Prefer noun phrases for checklist items over imperative verbs. Write "temp directory teardown" not "tear down the temp directory".
- Headings in Markdown files must be in title case: "Build from Source" not "Build from source". Minor words stay lowercase unless they are the first
  word: the articles (a, an, the), the coordinating conjunctions (and, but, or, nor, so, yet, for), and the short prepositions (in, on, at, to, by, of,
  up, as, from, with, into, over).
- Do not bold the lead-in of a list item. Write "Vector and set similarity: ..." not "**Vector and set similarity**: ...".
- Use sentence case for the lead-in of a list item. Write "Seed selection: ..." not "Seed Selection: ...". Proper nouns keep their capitals.
- Capitalize only the first part of a hyphenated compound: "Full-text Search" in a heading, "Breadth-first" at the start of a sentence, and
  "breadth-first search" elsewhere. Never write "Breadth-First".
- Start each sentence with a capital letter, capitalize proper nouns (Rust, Cypher, LMDB), and leave common nouns lowercase in the middle of a sentence.
- Write correct and complete sentences. Avoid made-up words.
- Do not use a colon in place of a verb. A colon may join two clauses inside a complete sentence, introduce the gloss of a list item, or introduce an
  enumeration. It must not turn a sentence into a label and a definition: write "Merges vector search seeds with text search seeds, then expands via
  BFS" rather than "Hybrid retrieval: merges vector search seeds with text search seeds".
- Use participial phrases and abbreviations scarcely.

## Repository Layout

An entry says what a module owns and where a new thing belongs. How a module works lives in the crate's own `AGENTS.md`, named at the end of this
section, and a public method's contract lives under Component APIs. Do not invent modules that do not yet exist; place new modules according to this map.

- `crates/issundb-core/`: storage engine. Public surface is `Graph` and the schema types; the source tree is the module map. Only the files below carry a
  rule the module name does not show.
    - `src/bin/gen_testdata.rs`: the `gen_testdata` binary that regenerates the versioned LMDB storage-format snapshot (`make testdata`).
    - `src/array.rs`: `Array<T>`, the owned-or-mapped array behind the CSR snapshot and the property columns. Mutate one only through `with_mut`,
      which copies a mapped view onto the heap; a mapped cache file is never written through.
    - `src/cache_file.rs`: the on-disk cache files for the CSR snapshot and the property columns (`lmdb` feature only), keyed by database identity and
      commit generation, refused on any mismatch, and memory-mapped rather than read on load. The only save sites are `Graph::rebuild_csr` and the `materialize_*_columns` methods; no lazy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IssunDB/issun-db](https://github.com/IssunDB/issun-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
