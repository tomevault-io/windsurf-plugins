---
trigger: always_on
description: **fastq-dl** is a Python CLI tool for downloading FASTQ sequencing files from the European Nucleotide Archive (ENA) or Sequence Read Archive (SRA). It accepts various accession types (Project, Study, BioSample, Sample, Experiment, Run) and handles provider fallback, retry logic, and optional run merging.
---

# CLAUDE.md - fastq-dl Development Guide

## Project Overview

**fastq-dl** is a Python CLI tool for downloading FASTQ sequencing files from the European Nucleotide Archive (ENA) or Sequence Read Archive (SRA). It accepts various accession types (Project, Study, BioSample, Sample, Experiment, Run) and handles provider fallback, retry logic, and optional run merging.

- **Version**: 4.0.0
- **License**: MIT
- **Python**: >=3.10, <3.14
- **Repository**: https://github.com/rpetit3/fastq-dl

## Quick Reference

```bash
# Install dependencies
poetry install

# Run tests (unit only, excludes integration)
just test-cov

# Run integration tests (makes real API calls)
just test-integration

# Format code
just fmt

# Lint code
just lint

# Full check (format + lint)
just check

# Build package
just build
```

## Project Structure

```
fastq-dl/
├── fastq_dl/                    # Main package
│   ├── __init__.py              # Version from importlib.metadata
│   ├── cli/
│   │   ├── __init__.py
│   │   └── download.py          # CLI entry point (click-based)
│   ├── providers/
│   │   ├── __init__.py
│   │   ├── generic.py           # Provider coordination and fallback
│   │   ├── ena.py               # ENA downloads via wget (FTP or HTTPS)
│   │   └── sra.py               # SRA downloads via sracha
│   ├── constants.py             # Shared constants (URLs, suffixes, sentinels)
│   ├── exceptions.py            # Custom exception hierarchy (8 types)
│   └── utils.py                 # Utilities (execute, md5sum, merge_runs, write_tsv)
├── tests/                       # Test suite
│   ├── conftest.py              # Shared fixtures
│   ├── test_cli.py              # CLI option/argument tests
│   ├── test_download.py         # Download logic tests
│   ├── test_integration.py      # Real API integration tests
│   ├── test_providers_ena.py    # ENA provider tests
│   ├── test_providers_generic.py # Generic provider tests
│   ├── test_providers_sra.py    # SRA provider tests
│   └── test_utils.py            # Utility function tests
├── test/                        # Test data files (sample FASTQs, TSVs)
├── .claude/
│   └── skills/
│       └── update-catalog/
│           ├── skill.md             # Skill definition for /update-catalog
│           └── scripts/
│               └── update_catalog.py # Regenerates catalog.json and llms.txt
├── .github/
│   ├── workflows/
│   │   ├── fastq-dl.yml         # CI: unit tests, integration tests, failure notification
│   │   └── release.yml          # CD: publish to PyPI on version tags
│   └── FUNDING.yml
├── pyproject.toml               # Poetry config, dependencies, pytest/ruff/coverage settings
├── justfile                     # Task runner commands
├── poetry.toml                  # Poetry local config
├── poetry.lock                  # Locked dependencies
├── environment.yml              # Conda environment (for external tools)
├── catalog.json                 # Machine-readable project metadata
├── llms.txt                     # AI-discovery document
├── citation.cff                 # Citation metadata
├── CHANGELOG.md                 # Release history
├── README.md                    # User documentation
├── LICENSE                      # MIT license
├── codecov.yml                  # Codecov configuration
├── .pre-commit-config.yaml      # Pre-commit hooks
└── .gitignore
```

## Architecture

### Core Flow
1. **CLI** (`cli/download.py`) → parses args via Click, sets up logging, orchestrates workflow
2. **Validation** (`utils.validate_query`) → validates accession format via regex
3. **Metadata Query** (`providers/generic.get_run_info`) → queries primary provider, falls back to secondary
4. **Download** (`cli/download.py:_download_with_fallback`) → attempts primary provider, falls back if needed
5. **Post-processing** → optional run merging, writes TSV metadata files

### Provider Pattern
- **ENA**: Uses HTTP API for metadata, wget for FTP/HTTPS downloads, MD5 validation
- **SRA**: Uses pysradb for metadata, sracha for download + FASTQ conversion + compression
- Automatic fallback between providers (configurable with `--provider` and `--only-provider`)

### Entry Point
- `pyproject.toml` defines: `fastq-dl = "fastq_dl.cli.download:main"`
- `main()` calls `fastqdl(["--help"])` when no args given, otherwise `fastqdl()`
- `fastqdl()` is the Click command that wraps `_run_download()` with exception handling

### Exception Hierarchy
```
FastqDLError (base)
├── ValidationError          # Invalid accession format
├── ProviderError            # API/query failures (attrs: provider, status_code)
├── EmptyResultError         # Provider returned HTTP 200 with no data (attrs: provider)
├── DownloadError            # File download failures (attrs: accession, provider)
├── AccessionNotFoundError   # Run not found on any provider (attrs: failed_runs)
├── MissingFastqsError       # FASTQs not yet available/synced (attrs: failed_runs)
└── PartialDownloadError     # Some runs succeeded, others failed (attrs: failed_runs, successful_runs)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rpetit3/fastq-dl](https://github.com/rpetit3/fastq-dl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
