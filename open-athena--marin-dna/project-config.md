---
trigger: always_on
description: **MarinDNA** is a framework for developing genomic language models (gLMs).
---

# Project Guidelines

## Project Overview

**MarinDNA** is a framework for developing genomic language models (gLMs).

## Domain Conventions

- **Coordinate system.** The codebase consistently uses 0-based, half-open intervals for all genomic coordinates. Assume this everywhere; call out any deviation explicitly. Conversions to/from 1-based closed formats (GTF, VCF, SAM) happen at the tool boundary, not inside our code.
- **Canonical public human genome.** Prefer the authenticated S3 mirror at `s3://oa-bolinas/data/genomes/homo_sapiens/GRCh38/ensembl-release-115/` whenever the caller has access and runs in the same region (`us-east-2`); it offers lower and more predictable latency for byte-range queries. For credential-free, public, or non-AWS consumers, use `marin-dna/human-genome` pinned to revision `11b9433582981bb929af333bc6422f10a8fd71b4`: uncompressed FASTA for `pyfaidx` + `fsspec` HTTP queries, or BGZF plus its indexes for HTSlib/samtools and full downloads. Never migrate already-materialized inputs merely to standardize their source. Both mirrors contain the Ensembl 115 GRCh38 soft-masked primary assembly with Ensembl sequence names (`1`...`MT`). MarinDNA code and `pyfaidx` slicing use 0-based, half-open intervals; `pyfaidx.get_seq()` and samtools-style region strings use 1-based, closed intervals and require conversion at the tool boundary. Other `hg38` variants are not interchangeable.

- **VEP chromosome split.** For labeled variant-effect prediction (VEP) data, use odd-numbered autosomes (`1, 3, …, 21`) and chromosome X for training, validation, development, model selection, probing, and hyperparameter tuning. Reserve even-numbered autosomes (`2, 4, …, 22`) and chromosome Y for final test evaluation; accessing their variant labels, effect measurements, predictions, or aggregate metrics requires explicit user permission and should be a rare final-evaluation event. This holdout applies to labeled variants and VEP-derived supervision, not to genomic sequence itself: unlabeled/reference-sequence pretraining may use every chromosome, as may functional-genomics data describing reference sequences unless the dataset defines a stricter split.

## Research Code Values

This is research code. Prioritize **reproducibility** and **correctness** over architectural elegance.

- **Put Python logic in the owning project package so pytest can reach it.** Reusable genomic primitives belong in the root `src/marin_dna/`; pipeline-specific functions belong in that pipeline's local `src/` package. Inline Python in Snakemake `run:` blocks should be thin glue calling tested functions. Maintained pipeline CLIs are package entry points, not loose files under a `scripts/` directory.
- **Duplication beats premature abstraction *within* the library.** The "testable home" rule governs *entry* into `src/marin_dna/` — move logic in freely, even if similar code already exists elsewhere. A separate, weaker rule governs *deduplication*: only merge two similar functions into one shared helper when the shape has stabilized and they're genuinely doing the same thing. Until then, two near-copies in two pipeline modules is better than a premature abstraction coupling unrelated experiments.
- **Modularity is a means, not a goal.** Don't refactor for reuse that may never come. Straight-line code that reads top-to-bottom is often preferable to layered abstractions.
- **Test aggressively.** Every non-trivial function in `src/marin_dna/` should have tests — that's the whole reason logic lives there. For pipelines, add sanity checks on outputs (row counts, value ranges, coordinate invariants) rather than trusting that "it ran".
- **Assert defensively, everywhere.** Use `assert` liberally for invariants that *should* hold: coordinate bounds, dataframe shapes, no NaNs where none are expected, set membership, monotonicity, matching lengths between parallel arrays. A loud failure near the bug is worth far more than a silently corrupted result feeding into training.
- **Fail fast on silent-corruption risks.** Bioinformatics is full of off-by-one errors, strand mix-ups, and reference-build mismatches. When a result could be quietly wrong, prefer a check that crashes over a comment saying "this should be correct".
- **No premature generalizations.** If asked to implement a specific backend, dataset, or model variant, stick to that. Don't generalize to related use-cases on your own — offer the option, but only expand the scope when explicitly told to.
- **Stay in scope.** Don't remove or rewrite unrelated code in other pipelines or library modules while working on a task. Unrelated experiments may depend on exact current behavior.

## Code Structure

The codebase has five main components:

1. **Python core** (`src/marin_dna/`) - lightweight reusable genomic primitives shared by independent projects. Core must not depend on Torch, Transformers, Snakemake, WandB, plotting libraries, or pipeline-specific upload clients.

2. **Pipelines** (`snakemake/`) - Data processing workflows implemented in Snakemake
   - Read the pipeline's README before working on it — each `snakemake/<pipeline>/` has its own. If you change pipeline behaviour, update the README in the same PR so the next human or agent can onboard from it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-Athena/marin-dna](https://github.com/Open-Athena/marin-dna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
