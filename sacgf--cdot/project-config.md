---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**cdot** converts RefSeq/Ensembl GTF/GFF3 annotation files into JSON format, providing high-performance transcript data for two Python HGVS variant annotation libraries: **biocommons HGVS** and **PyHGVS**.

Performance comparison: UTA public DB ~1s/transcript vs cdot JSON.gz ~500-1000 transcripts/second.

## Important Constraints

**Never run code against full datasets.** Processing full GTF/GFF3 files or complete JSON.gz data releases takes a very long time. Always use test data (in `tests/`) when verifying changes.

**PyHGVS is abandoned — prefer biocommons HGVS.** The `cdot/pyhgvs/` integration exists for legacy compatibility but PyHGVS is no longer maintained. Do not write new features that require significant PyHGVS-specific work. Focus new development on the biocommons HGVS path (`cdot/hgvs/dataproviders/`). If a feature is straightforward to support in both libraries, fine; if it requires real effort for PyHGVS, skip it and biocommons-only is acceptable.

**Keep the changelog up to date.** `CHANGELOG.md` is for **users of the client code** — only add entries for changes such a user would care about: client API/behaviour changes, dependency/compatibility changes (eg dropping a Python version), and changes to the **data content** they consume (note when a change only affects data and not client code). Add the entry to the `[unreleased]` section under `### Added`, `### Changed`, or `### Fixed`, and reference the GitHub issue it is done against (eg `#27`).

Do **NOT** add changelog entries for things a client-code user never sees: documentation, the paper (`paper/`), benchmark/analysis tooling (`analysis/`, `tests/benchmark_*.py`), CI/dev infrastructure, or the data-generation/build pipeline (`generate_transcript_data/`, Snakemake) when it doesn't change the released data. When in doubt, ask "would someone who only does `pip install cdot` and calls the library (or downloads a data release) notice this?" — if no, leave it out.

**The `cdot_private` repo holds real-world data — never copy examples from it into this repo.** It is checked out as a sibling directory (`../cdot_private`, github.com/SACGF/cdot_private) and contains a corpus of real search-bar HGVS strings (`combined_search_hgvs.csv`, built by its `process_search_hgvs.py`) plus `analyze_cleaning.py`, which runs that corpus through `cdot.hgvs.clean.clean_hgvs` to report rescue rates and failure patterns (issue #112). Use it to find what cleaning should handle, but **no example string sourced from `cdot_private` may ever appear in this repo's tests, comments, docstrings, or changelog** — synthesise equivalents from the standard public examples already used in the tests (eg `NM_000059.4` BRCA2, `NM_001754.5` RUNX1).

**Benchmarking.** `paper/scripts/benchmark_resolution.py` resolves real ClinVar (g.HGVS, c.HGVS) pairs (in `tests/test_data/clinvar_hgvs/`) through a pluggable provider (REST/JSON/UTA) to measure resolution accuracy, recovery (cleaning + version-bump), and speed; supports `--prefetch` and a local `--fasta`. This is how the README performance numbers are produced.

**Sequence fetching: use `FastaSeqFetcher` (local genome FASTA) whenever Ensembl transcripts are involved.** Resolving a `c.`/`n.` HGVS needs the transcript sequence (biocommons `c_to_g` fetches it), served by SeqRepo, cdot's `FastaSeqFetcher` (splices the transcript's exon ranges out of a local genome FASTA), or `ChainedSeqFetcher` (tries several in order). SeqRepo is RefSeq-centric and does not hold every Ensembl (`ENST`) transcript sequence, so a SeqRepo-only run fails on those with `Failed to fetch ENST... from SeqRepo`, which surfaces as `no_data` and looks like a cdot coverage gap when it is really a sequence-layer gap. `FastaSeqFetcher` avoids this: it reproduces an Ensembl transcript exactly (Ensembl transcript sequence == genome), and it is also faster and offline (no remote NCBI fetch). Use it (or `ChainedSeqFetcher(SeqRepo, FastaSeqFetcher)`) for any Ensembl work or benchmark, e.g. `benchmark_resolution.py ... --fasta GENOME.fna.gz`. Caveats: (1) the FASTA must use the same contig names as cdot's data (NC_ accessions), so use the NCBI GRCh38 assembly (`GCF_000001405.39`, `NC_000001.11`...), not an Ensembl-named FASTA (`1`,`2`,...); (2) `FastaSeqFetcher` is not guaranteed to match a curated RefSeq transcript exactly (RefSeq sequence can differ from the genome at a few positions), so for an exact RefSeq sequence prefer SeqRepo or chain SeqRepo first. On this dev machine the GRCh38 FASTA is `/data/annotation/fasta/GCF_000001405.39_GRCh38.p13_genomic.fna.gz` (indexed).

**Avoid AI writing tells in documentation and paper edits.** When writing or editing prose (docs, `README`, `paper/`, comments, commit/PR text), do not use em-dashes. Prefer plain punctuation: commas, periods, parentheses, or a rephrase. Also avoid other common LLM tells: "It's not just X, it's Y" constructions, hedging filler ("it's worth noting", "importantly", "delve"), over-use of bold for emphasis, and formulaic tricolon lists. Write plainly and directly, matching the voice of the surrounding text.

## Commands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SACGF/cdot](https://github.com/SACGF/cdot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
