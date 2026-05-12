---
trigger: always_on
description: This document provides comprehensive guidance for AI agents working with the **quantmsdiann** bioinformatics pipeline. These guidelines ensure code quality, maintainability, and compliance with project standards.
---

# AI Agent Guidelines for quantmsdiann Development

This document provides comprehensive guidance for AI agents working with the **quantmsdiann** bioinformatics pipeline. These guidelines ensure code quality, maintainability, and compliance with project standards.

## Critical: Mandatory Validation Before ANY Commit

**ALWAYS run pre-commit hooks before committing ANY changes:**

```bash
pre-commit run --all-files
```

This is **non-negotiable**. All code must pass formatting and style checks before being committed.

---

## Project Overview

**quantmsdiann** is a [bigbio](https://github.com/bigbio) bioinformatics pipeline, built following [nf-core](https://nf-co.re/) guidelines, for **DIA-NN-based quantitative mass spectrometry**. It is a standalone pipeline focused exclusively on **Data-Independent Acquisition (DIA)** workflows using the DIA-NN search engine.

**This pipeline does NOT support DDA, TMT, iTRAQ, LFQ-DDA, or any non-DIA workflows.** Those are handled by the parent `quantms` pipeline.

**Key Features:**

- Built with Nextflow DSL2
- DIA-NN for peptide/protein identification and quantification
- Supports DIA-NN v1.8.1, v2.1.0, v2.2.0, v2.3.2, and v2.5.0 (latest)
- QuantUMS quantification method (DIA-NN >= 1.9.2)
- Parquet-native output with decoy reporting (DIA-NN >= 2.0)
- MSstats-compatible output generation (via quantms-utils conversion, no MSstats analysis)
- Quality control with pmultiqc
- Complies with nf-core standards

**Repository:** https://github.com/bigbio/quantmsdiann

---

## Technology Stack

### Core Technologies

- **Nextflow**: >=25.04.0 (DSL2 syntax)
- **nf-schema plugin**: 2.5.1 (parameter validation)
- **nf-test**: Testing framework (config: `nf-test.config`)
- **nf-core tools**: Pipeline standards and linting
- **Containers**: Docker/Singularity/Apptainer/Podman (Conda deprecated)
- **DIA-NN**: Primary search engine (versions 1.8.1 through 2.5.0)

### Key Configuration Files

- `nextflow.config` - Main pipeline configuration
- `nextflow_schema.json` - Parameter schema (auto-generated)
- `nf-test.config` - Testing configuration
- `.nf-core.yml` - nf-core compliance settings
- `modules.json` - Module dependencies
- `.pre-commit-config.yaml` - Pre-commit hooks

### Project Structure

```
quantmsdiann/
├── main.nf                    # Pipeline entry point
├── workflows/
│   ├── quantmsdiann.nf        # Main workflow orchestrator
│   └── dia.nf                 # DIA-NN analysis workflow
├── subworkflows/local/        # Reusable subworkflows
│   ├── input_check/           # SDRF validation
│   ├── file_preparation/      # Format conversion
│   └── create_input_channel/  # SDRF metadata parsing
├── modules/local/
│   ├── diann/                 # DIA-NN modules (7 steps)
│   │   ├── generate_cfg/
│   │   ├── insilico_library_generation/
│   │   ├── preliminary_analysis/
│   │   ├── assemble_empirical_library/
│   │   ├── individual_analysis/
│   │   ├── final_quantification/
│   │   └── diann_msstats/
│   ├── openms/                # mzML indexing, peak picking
│   ├── pmultiqc/              # QC reporting
│   ├── sdrf_parsing/          # SDRF parsing
│   ├── samplesheet_check/     # Input validation
│   └── utils/                 # decompress, mzml stats
├── conf/
│   ├── base.config            # Resource definitions
│   ├── modules/               # Module-specific configs
│   ├── tests/                 # Test profile configs (DIA only)
│   └── diann_versions/        # DIA-NN version-override configs for merge matrix
├── tests/                     # nf-test test cases
└── assets/                    # Pipeline assets and schemas
```

---

## DIA-NN Workflow

The pipeline executes the following steps:

1. **SDRF Validation & Parsing** - Validates input SDRF and extracts metadata
2. **File Preparation** - Converts RAW/mzML/.d/.dia files (ThermoRawFileParser)
3. **Generate Config** - Creates DIA-NN config from enzyme/modifications (`quantmsutilsc dianncfg`)
4. **In-Silico Library Generation** - Predicts spectral library from FASTA (or uses provided library)
5. **Preliminary Analysis** - Per-file calibration and mass accuracy determination
6. **Assemble Empirical Library** - Builds consensus library from preliminary results using .quant files
7. **Individual Analysis** - Per-file search with empirical library (optional, for large datasets)
8. **Final Quantification** - Summary quantification with protein/peptide/gene matrices
9. **MSstats Format Conversion** - Converts DIA-NN report to MSstats-compatible CSV (`quantmsutilsc diann2msstats`)
10. **pmultiqc** - Quality control reporting

### DIA-NN Version-Specific Features

| Feature                                     | Min Version | Parameter              |
| ------------------------------------------- | ----------- | ---------------------- |
| Core workflow, library-free, .quant caching | 1.8.1       | (default)              |
| QuantUMS quantification                     | 1.9.2       | `--quantums true`      |
| Parquet output format                       | 2.0         | (automatic in 2.0+)    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigbio/quantmsdiann](https://github.com/bigbio/quantmsdiann) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
