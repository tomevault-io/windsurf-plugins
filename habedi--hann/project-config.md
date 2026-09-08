---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Hann is an approximate nearest neighbor search library for Go.
It provides a set of index data structures (HNSW, PQIVF, and RPT) behind one interface, with distance computation written
in C and vectorized with AVX instructions.
Priorities, in order:

1. Correctness of index operations: insertion, deletion, update, and search must keep the index consistent.
2. Search quality and speed, measured by recall and query latency on the example datasets.
3. Clean separation between the shared interface and helpers (`core/`) and the index implementations (`hnsw/`, `pqivf/`, and `rpt/`).
4. Safety of the cgo boundary: no out-of-bounds reads, and no pointers into Go memory that outlive the call.

## Core Rules

- Use English for code, comments, docs, and tests.
- Prefer small, focused changes over large refactoring.
- Add comments only when they clarify non-obvious behavior.
- Do not add features, error handling, or abstractions beyond what is needed for the current task.
- Keep external dependencies minimal: do not add new `go.mod` entries without prior discussion.

## Backward Compatibility

Hann is a public Go module that other programs import. The following must stay backward-compatible:

- The `core.Index` interface. Adding a method breaks every implementation outside this repository, so a new capability belongs on the concrete index
  types, or on a separate optional interface that callers can assert, like `core.BulkIndex` and `core.Trainer`.
- Exported types and constructor signatures (`hnsw.New`, `pqivf.New`, and `rpt.New`, each returning `(*Index, error)`). A new tuning parameter is a
  new functional option, never a change to an existing signature.
- The shapes of `core.Neighbor` and `core.IndexStats`: fields may be added, not removed or renamed.
- The gob encoding written by `Save`. An index file written by an older version must still load. The `serializedIndex`, `serializedPQIVF`, and
  `rptSerialized` structs are the on-disk format, so fields may be added with sensible zero values, but they may not be removed, renamed, or reordered
  in meaning.
- The names of the built-in metrics in the `core` registry and the names reported by `IndexStats.Distance`.
- The environment variables `HANN_SEED` and `HANN_BENCH_NTRD`, along with the values they accept. `HANN_LOG` is accepted and ignored,
  because the library no longer logs.
- The minimum Go version declared in `go.mod`. Raising it drops users, so it is a deliberate decision, not a side effect of using a newer standard
  library function.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes, in documentation or in code comments. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "rate limiter" not "smart rate limiter".
- Prefer noun phrases for checklist items over imperative verbs. Write "rate limit enforcement" not "enforce rate limits".
- Headings in Markdown files must be in title case: "Build from Source" not "Build from source". Minor words stay lowercase unless they are the first
  word: the articles (a, an, the), the coordinating conjunctions (and, but, or, nor, so, yet, for), and the short prepositions (in, on, at, to, by,
  of, up, as, from, with, into, over). The prepositions are named because "from" has to be lowercase for "Build from Source" to be correct.
- Do not bold the lead-in of a list item. Write "Unit tests: ..." not "**Unit tests**: ...".
- Use sentence case for the lead-in of a list item. Write "Seed selection: ..." not "Seed Selection: ...". Proper nouns keep their capitals.
- Capitalize only the first part of a hyphenated compound: "Nearest-neighbor Search" in a heading, "Nearest-neighbor" at the start of a sentence, and
  "nearest-neighbor search" elsewhere. Never write "Nearest-Neighbor".
- Start each sentence with a capital letter, capitalize proper nouns (Go, AVX, SIMD, HNSW, PQIVF, RPT), and leave common nouns lowercase in the middle
  of a sentence.
- Write correct and complete sentences.
- Avoid made-up words.
- Do not use a colon in place of a verb. Three uses are fine: joining two clauses inside a complete sentence (the replacement the em-dash rule above
  calls for), introducing the gloss of a list item, and introducing an enumeration, whether as a list or inline ("Targets: `make test`,
  `make lint`, ..."). What a colon must not do is turn a sentence into a label and a definition: write "Splits a vector into subspaces, then quantizes
  each one" rather than "Product quantization: splits a vector into subspaces". That shape belongs to a list item, and carrying it into prose (a doc
  comment summary, a paragraph) leaves a fragment where a sentence was required.
- Use participial phrases and abbreviations scarcely.

## Repository Layout

- `core/`: the shared interface and helpers. `index.go` declares `Index`, the optional `BulkIndex` and `Trainer` interfaces, `Neighbor`, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habedi/hann](https://github.com/habedi/hann) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
