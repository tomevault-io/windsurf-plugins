---
trigger: always_on
description: Notes for coding agents working in this repository. The README explains what the tool
---

# AGENTS.md

Notes for coding agents working in this repository. The README explains what the tool
does and how to run it; this file covers what will bite you. Read both.

## Ground rules

- **`reference/` is immutable.** `reference/legacy_generate_families.py` is the
  behavioural oracle for the whole port, vendored byte-identically from the `mgnifams`
  pipeline. It is excluded from every `pre-commit` hook. Never format, lint, or "fix" it.
  When you need to know what the code *should* do, read it.
- **Never add a dependency.** The four scientific packages are upper-bounded on purpose:
  they decide hit retention, alignment, and serialised bytes, so the reproducibility
  contract holds only for the resolved set in `uv.lock`. Use `uv run ...` for everything.
- Use `uv lock --check && uv sync --frozen` before trusting a test result.
- Commits and pushes are the human's call.

## Before you change anything in `generate_families.py`

Five invariants are load-bearing. Each was a real bug at some point, each is guarded by
exactly one test, and each survives a passing end-to-end run if you break it.

1. **`hmmsearch` must be called with `parallel="queries"`, explicitly.** Left to choose,
   pyhmmer picks `parallel="targets"` whenever the query count is below `cpus` — which
   the last wave of a batch usually is. Target-parallelism merges each worker's stored
   hits while re-thresholding only the reporting flags, so results start depending on the
   machine's core count. All searching goes through `search()`; keep it that way.

2. **Extraction reads `top_hits.reported`, never `top_hits` directly.** The raw list
   holds hits that failed `--recruit_evalue_cutoff`. Iterating it is what the legacy code
   did, and it is the bug 1.0.0 fixed. Guard:
   `test_unreported_hit_is_not_recruited_and_prefetch_matches` — query `4497037939_1_144`
   stores 55 hits, reports 54, and the extra one is sequence `6320430079`.

3. **Hit records are plain tuples, copied out immediately.** A pyhmmer `Domain` holds a
   reference to its `Hit`, which holds the whole `TopHits`. Caching `Domain` objects pins
   every result graph for the batch. Guard:
   `test_extracted_records_do_not_retain_pyhmmer_results`.

4. **Reported domains are sorted by score within each hit.** HMMER reports domains in
   positional order, but row 0 supplies representative metadata and length. The
   top-ranked hit's highest-scoring domain must lead. Guard:
   `test_extract_records_puts_top_scoring_domain_first`.

5. **The SSI index guard is an `if ... raise`, not an `assert`.** `python -O` strips
   asserts, and the failure it prevents is a silent wrong-sequence read from a stale
   index. Guard: `test_index_mismatch_guard_and_optimized_python`.

Also: `Family.advance()` and `Family.finish()` must stay side-effect free. Family ids are
the rank among *successful* families in cluster-file order, so nothing may be written
until a family's fate is known. All writing happens in `emit_family()`, called in cluster
order. A family that converges in round 1 must not record itself ahead of an earlier
family still running in round 3. Discarded families must never appear in
`converged_families`. Every family must leave `emit_family()` on exactly one side of the
generated/discarded split; an internal crash is recorded as a discard so the chunk can
finish, then also fails the completed run with exit 3. Exit 1 means containment could not
leave coherent output, so that output must not be consumed.

## Deliberate differences from the legacy script

The legacy oracle records the starting behaviour, not every current guarantee.
`CHANGELOG.md` documents the intentional fixes; the historical PLAN files describe
earlier decisions and must not be used to reverse them.

- On convergence and at `MAX_ROUNDS`, `Family.advance()` skips the re-align/trim tail.
  The seed remains the one that built the final search model. Legacy ran an unused
  final trim and exported a model built from a seed it had never searched with.
  Guard: `test_final_round_leaves_the_searched_seed_in_place`.
- `renumber_msa()` names both output alignments, so seed and full Stockholm files
  contain `#=GF ID <family>`. HMMER posterior annotations are omitted, but the RF line
  remains. Legacy's `renumber_sto_msa` stripped the family ID along with those annotations.
  Guard: `test_declared_outputs_parse_and_long_fixture_runs`.

There are two distinct clipping functions. `clip_env_ends()` reads the `#=GC RF` line and
trims envelope overhangs from seed alignments. `clip_ends()` reads gap occupancy and runs
after redundancy trimming. They are not interchangeable.

Legacy `clip_ends()` dropped the last column that passed the
occupancy threshold, and reported a full span when no column passed. Both were fixed in
1.0.0, so it now diverges from `reference/legacy_generate_families.py` on purpose. If you
diff against the legacy baseline, expect every model to be one match state wider.

`split_slice_name()` is another deliberate divergence. Legacy recovered a slice's
parent protein with `split("_")` on exactly three fields, which truncated any protein
name that itself contains underscores, raised on non-numeric trailing fields, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vagkaratzas/mgnifam](https://github.com/vagkaratzas/mgnifam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
