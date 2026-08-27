---
trigger: always_on
description: Semantic grep for agents built on the Bog stack: `../ese` (static embeddings,
---

# gorp (repo dir: gorp/)

Semantic grep for agents built on the Bog stack: `../ese` (static embeddings,
256-dim, compiled-in weights) + `../anny` (HNSW). README.md is usage; everything
else is in `docs/`, and a `§9.1` in a source comment means `docs/RESEARCH.md` —
the only numbered document, so there is nothing else to name.

- `docs/DESIGN.md` — the v1 design, the statement of intent.
- `docs/RESEARCH.md` — the research log, and the doc most cited from code.
  Distilled 2026-08-16: every measured result kept with its numbers and its
  verdict, the procedure and restatement around them cut. **Its section numbers
  are an anchor namespace rather than an outline** — 428 comments cite them and
  `crates/gorp-core/tests/docs.rs` fails the build when a cited section stops
  existing, so sections may be shortened but never renumbered.
- `docs/CHANGELOG.md` — what changed, release by release.

The same pass folded away three documents: the defect ledger, the session-level
behavior audit of `eval/sim/`, and the evaluation of `../fold` as a store for
the repair overlay (which found a fjall-lock blocker and stopped there). What
each one was load-bearing for now lives where it is checked — in the comments
and tests that were written from it — so a comment states the measurement
instead of pointing at it.

## Layout

### Layers

`crates/gorp-core` is organized in layers, bottom up. **A layer may call
downward and not upward.** That is the rule to check a change against, and it is
what keeps the tree navigable: `rank` never touches the filesystem, `store` never
ranks, `cache` never scores, `search` orchestrates rather than computes.

- `trace.rs` — stage timing. A leaf every layer may use, and the one module
  outside the stack: `Stage` is a closed enum, each path declares a
  `SCHEDULE_*`, and every stage belongs to exactly one `Bucket` so
  `walk`/`load`/`rank` are *derived* sums and `unattributed_ms` means "work
  nothing is timing". `crates/gorp-core/tests/trace.rs` bounds that residual.
- `corpus/` — directory into files into chunks. `mod` walks, `chunk` cuts and
  re-reads, `pass` drives the parallel read, `diff` compares a tree against an
  index, `funcchunk` cuts on function boundaries (tree-sitter, `func-chunk`).
- `text/` — text into representations. `token` (code-aware tokenizer), `embed`,
  `sif` (rarity-weighted pooling, §9.1), `prose` (how text is rendered before
  embedding, §14.2/§20) over `prose_vocab` (the frozen keyword tables — data
  with a provenance: each was measured against a published arm, so editing one
  silently re-defines that arm).
- `rank/` — query plus representations into ordered ids. `bm25` (one scorer over
  a `Postings` trait), `vec` (kernels and quantization), `topk`, `fuse`, `mmr`,
  `prf`, `maxsim`, `bridge` (vocabulary-gap expansion, §33).
- `store/` — representations on disk. `build` (+ `build/embed`, `build/sif`),
  `load`, `bm25` (the flat mmap layout).
- `cache/` — which index answers, and keeping it honest. `mod` (discovery,
  fill), `compat` (generations), `budget`, `repair` (the read-repair overlay).
- `search/` — orchestration, and the tail that turns ids into hits. `indexed`
  (warm), `stream` (cold), `rows` (the union id space), `options`
  (`SearchOptions`, the type the CLI mirrors flag for flag), `query` (phrase
  splitting, §31), `hit` (candidate sequencing) over `rerank` (the fine
  rerank, §29.1) and `materialize` (span overlap, budget, best-line re-read),
  `unit` (the unit view's rows, §34), `checklist` (the learned blend, §35.2).
- `keyword.rs` — the exact-match escape hatch, independent of all of it.

`crates/gorp` is the CLI, one binary named `gorp`. The 2026-08 rename went
all the way through — `GORP_*` env vars, `~/.cache/gorp`, `.gorp/` index
dirs, and a `gorp: ` stderr prefix that lives in one constant
(`out::PROG`) rather than the ~26 literals it used to. That is the
expensive half of a rename and it invalidated every index built before it
(RESEARCH.md §19.9); they rebuild on first search. The old `sg` and
`semgrep` binary names are gone.
Its modules: `cli` (flags), `cmd/` (one file per verb), `out`
(every write to stdout or stderr), `telemetry` (the `GORP_TRACE_FILE`
envelope — schema `gorp.trace/1`, and the contract the eval harnesses
read). **stdout is data, stderr is commentary** —
`crates/gorp/tests/cli.rs` enforces it.

Engine integration tests split by concern under `crates/gorp-core/tests/`:
`e2e_general.rs` (the four modes, and indexed==unindexed parity),
`e2e_cache.rs` (§8 cache behaviors, and `cold_and_warm_return_identical_results`
itself), `e2e_publish.rs` (publication is a swap), `repair.rs` (a warm index
plus its overlay answers as a fresh build would), over a shared
`common/mod.rs`. Each is its own process and therefore its own cache dir.

### Harnesses

- **The agent harnesses and the perf benchmark live in `../gorp-bench`**
  (github.com/nlaz/gorp-bench): SWE-Explore-Bench and Loc-Bench campaigns,
  the PATH shim, guess harvesting, and the grep/ripgrep comparison. They run
  live agents against real repositories, which costs money and hours — the
  opposite lifecycle from a `cargo test`, which is why they moved out. That
  repo consumes this one as a sibling checkout: `GORP_BIN` for the binary,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nlaz/gorp](https://github.com/nlaz/gorp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
