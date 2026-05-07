---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
make fmt          # cargo fmt --all
make fmt-check    # check formatting without modifying
make lint         # cargo clippy --workspace -- -D warnings
make test         # cargo test --workspace
make build        # debug build
make release      # release build
make check        # cargo check --workspace
make install-cli  # install vareffect binary locally
```

Run a single test:
```bash
cargo test -p vareffect test_name
```

Integration tests are `#[ignore]`-gated and require runtime data files (`data/vareffect/GRCh38.bin`, `data/vareffect/transcript_models.bin`). To run them:
```bash
cargo test -p vareffect -- --ignored
```

Generate data files with `vareffect setup` (one-time, ~10 min, ~3 GB disk).

## Architecture

**Workspace layout:** Two crates — `vareffect` (core library) and `vareffect-cli` (data provisioning + VCF annotation CLI).

### Core library (`vareffect/`)

The annotation pipeline flows through three main components:

1. **`TranscriptStore`** (`transcript.rs`) — In-memory store of MANE/RefSeq Select transcript models loaded from MessagePack binary. Indexed by COITree per chromosome for O(log n + k) overlap queries and by accession HashMap for O(1) lookup. Arc-backed for cheap thread sharing.

2. **`FastaReader`** (`fasta.rs`) — Memory-mapped flat binary reference genome (~3.1 GB for GRCh38). Zero-copy random access (~5 ns per base). Maps UCSC/NCBI/Ensembl chromosome naming conventions automatically.

3. **`VarEffect`** (`var_effect.rs`) — Stateful entrypoint bundling TranscriptStore + FastaReader. Main API: `annotate(chrom, pos, ref_allele, alt_allele) → Vec<ConsequenceResult>`. Wrap in `Arc<VarEffect>` for multi-threaded use.

**Annotation pipeline** for a single variant:
- `VarEffect::annotate` queries overlapping transcripts via interval tree
- For each transcript: `locate_variant`/`locate_indel` (`locate/`) classifies position (CDS, intron, UTR, splice site)
- Consequence assignment (`consequence/snv.rs`, `indel.rs`, `complex.rs`) translates ref/alt codons and assigns SO terms
- HGVS notation generated (`hgvs_c.rs` for c./n., `hgvs_p.rs` for p.)
- NMD prediction (`consequence/nmd.rs`) via 50-nucleotide rule on truncating variants

### CLI (`vareffect-cli/`)

- **`setup`** — Downloads GRCh38 FASTA + MANE GFF3, builds flat binary genome and MessagePack transcript store. Idempotent.
- **`annotate`** — Parallel VCF annotation (rayon). Writes CSQ INFO field in VEP `--vcf` format.
- **`models`** — Standalone transcript model builder from GFF3.
- Config in `vareffect_build.toml` (download URLs, output paths).

## Critical Conventions

**All coordinates are 0-based, half-open** (BED/UCSC style). GFF3 input (1-based, fully-closed) is converted at build time. Getting this wrong silently produces off-by-one annotation errors.

**Chromosome names** are UCSC-style (`chr17`, `chrM`). The `chrom` module handles UCSC ↔ RefSeq accession mapping.

**`chrM` uses NCBI genetic code table 2** (vertebrate mitochondrial). Standard chromosomes use table 1. The `codon` module handles this automatically.

## Design Principles

- **No embedded data** — Library ships no reference genomes or transcripts; all data built offline via CLI
- **Thread-safe by default** — `VarEffect`, `TranscriptStore`, `FastaReader` are `Send + Sync` (compile-time assertion in `lib.rs`)
- **VEP concordance** — Targets Ensembl VEP release 115/116 output; intentional divergences documented in `vareffect/VEP_DIVERGENCES.md`
- **Biotype forward compatibility** — `Biotype` enum uses `Other(String)` for unknown labels so new upstream biotypes don't break deserialization

## Code Style and Formatting

- **MUST** use meaningful, descriptive variable and function names
- **MUST** follow Rust API Guidelines and idiomatic Rust conventions
- **MUST** use 4 spaces for indentation (never tabs)
- **NEVER** use emoji, or unicode that emulates emoji (e.g. ✓, ✗). The only exception is when writing tests and testing the impact of multibyte characters.
- Use snake_case for functions/variables/modules, PascalCase for types/traits, SCREAMING_SNAKE_CASE for constants
- Limit line length to 100 characters (rustfmt default)

## Documentation

- **MUST** include doc comments for all public functions, structs, enums, and methods
- **MUST** document function parameters, return values, and errors
- Keep comments up-to-date with code changes
- Include examples in doc comments for complex functions

Example doc comment:

````rust
/// Calculate the total cost of items including tax.
///
/// # Arguments
///
/// * `items` - Slice of item structs with price fields
/// * `tax_rate` - Tax rate as decimal (e.g., 0.08 for 8%)
///
/// # Returns
///
/// Total cost including tax
///
/// # Errors
///
/// Returns `CalculationError::EmptyItems` if items is empty
/// Returns `CalculationError::InvalidTaxRate` if tax_rate is negative
///
/// # Examples
///
/// ```
/// let items = vec![Item { price: 10.0 }, Item { price: 20.0 }];
/// let total = calculate_total(&items, 0.08)?;
/// assert_eq!(total, 32.40);
/// ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LIA-Genomics/vareffect](https://github.com/LIA-Genomics/vareffect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
