---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Eukan is a eukaryotic genome annotation pipeline that integrates ab initio gene prediction (GeneMark, AUGUSTUS, SNAP, CodingQuarry) with homology-based evidence (protein alignments via spaln/gth) and transcript assemblies to produce consensus gene models via EVidenceModeler (EVM). It optionally adds UTRs via PASA and functional annotation via phmmer-against-UniProt or hmmscan-against-KOfam (adapted from KofamKOALA), plus hmmscan-against-Pfam.

## Build and Run

Uses Poetry for package management. The pipeline runs inside Docker with a GeneMark-ES/ET/EP+ license required.

```bash
# Install locally for development (pulls in pytest, ruff, mypy)
poetry install --with dev

# Run tests
poetry run pytest tests/ -v

# CLI (all subcommands)
poetry run eukan --help
poetry run eukan mask-repeats -g genome.fasta
poetry run eukan annotate -g genome.fasta -p proteins.fasta --kingdom protist
poetry run eukan assemble -g genome.fasta -l left.fq -r right.fq -A -T -P
poetry run eukan func-annot -p proteins.faa --gff3 genes.gff3
poetry run eukan prep-submission -t submission.sbt --organism "Genus species"
poetry run eukan gff3toseq -g genome.fa -i genes.gff3 --output-format protein -o proteins.faa
poetry run eukan db-fetch -o databases/
poetry run eukan compare -r ref.gff3 -p pred.gff3                # single
poetry run eukan compare -r ref.gff3 -p p1.gff3 -p p2.gff3 -p p3.gff3 \
    -o details.tsv                                                 # multi-pred + per-feature TSV

# Dev tooling (not exposed via main CLI)
python tests/run_pipeline.py setup-test-data
python tests/run_pipeline.py test-pipeline --kingdom fungus -n 8
python tests/run_pipeline.py clean-test-data --all
python scripts/generate-env.py -o environment.yml

# Docker build and run
docker build -t eukan -f docker/Dockerfile .
./eukan-docker annotate -g genome.fasta -p proteins.fasta --kingdom protist
```

## Architecture

### CLI (`eukan/cli/`)

Click-based CLI package with subcommands: `annotate`, `assemble`, `mask-repeats`, `func-annot`, `prep-submission`, `gff3toseq`, `db-fetch`, `check`, `status`, `compare`. One file per subcommand under `eukan/cli/`; shared infrastructure (option-group rendering, `numcpu_option`, `genome_option`, error formatting) lives in `eukan/cli/_framework.py`. Entry point defined in `pyproject.toml` as `eukan = "eukan.cli:cli"`. All subcommands use harmonized option groups ("Required input", "Pipeline parameters", "Re-run steps"). Step re-run flags follow the `--run-*` pattern (e.g., `--run-genemark`, `--run-star`).

### Package Structure

```
eukan/
├── cli/                # Click CLI package (one file per subcommand)
│   ├── _framework.py   # Shared Click helpers: option groups, common options, error formatting
│   ├── annotate.py     # eukan annotate
│   ├── assemble.py     # eukan assemble
│   ├── mask_repeats.py # eukan mask-repeats
│   ├── func_annot.py   # eukan func-annot
│   ├── prep_submission.py # eukan prep-submission
│   ├── compare.py      # eukan compare
│   ├── db_fetch.py     # eukan db-fetch
│   ├── gff3toseq.py    # eukan gff3toseq
│   ├── check.py        # eukan check (pre-flight tool/database checks)
│   └── status.py       # eukan status (manifest reader)
├── settings.py         # PipelineConfig, AssemblyConfig, RepeatsConfig, FunctionalConfig, SubmissionConfig (pydantic-settings)
├── validation.py       # FASTA/GFF3 validation and genome header sanitization
├── exceptions.py       # ConfigurationError and friends
│
├── infra/              # Runtime infrastructure (cross-pipeline)
│   ├── runner.py       # run_cmd(), run_piped(), run_parallel() — subprocess execution
│   ├── concurrency.py  # parallel_map() and friends
│   ├── manifest.py     # RunManifest, pipeline_step() — run tracking and reproducibility
│   ├── steps.py        # step_dir(), step_complete(), validate_or_raise() — step dir mgmt
│   ├── pipeline.py     # StepSpec + run_simple_pipeline() / run_orchestrated_step() driver
│   ├── layout.py       # PIPELINE_SUBDIRS, step_work_dir(), sibling_step_dir() — per-step run dir layout
│   ├── artifacts.py    # Artifact enum + find() — cross-pipeline artifact registry
│   ├── logging.py      # get_logger(), setup_logging(), md5_file()
│   ├── genome.py       # ContigIndex, FASTA helpers
│   ├── genetic_code.py # Genetic code table abstractions
│   ├── health.py       # Tool/database probes for `eukan check`
│   ├── tools_registry.py # Tool metadata loaded from data/tools.toml
│   ├── conda_env.py    # Conda env var setup at CLI startup
│   └── environ.py      # Env var helpers
│
├── gff/                # GFF3 format operations
│   ├── transforms.py   # Transform callbacks for gffutils.create_db(transform=fn)
│   ├── concordance.py  # Genomic interval operations (concordance, overlap, merging)
│   ├── intervals.py    # Lower-level interval primitives
│   ├── hierarchy.py    # gene>mRNA>CDS hierarchy fixers, prettify_gff3()
│   ├── normalize.py    # GFF3 cleanup before downstream tools (e.g., table2asn)
│   └── io.py           # featuredb2gff3_file(), count_gff3_features(), iter_assembled_sequences()
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BFL-lab/eukan](https://github.com/BFL-lab/eukan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
