---
trigger: always_on
description: This document provides comprehensive guidance for AI agents working with the **quantms** bioinformatics pipeline. These guidelines ensure code quality, maintainability, and compliance with project standards.
---

# AI Agent Guidelines for quantms Development

This document provides comprehensive guidance for AI agents working with the **quantms** bioinformatics pipeline. These guidelines ensure code quality, maintainability, and compliance with project standards.

## 🚨 Critical: Mandatory Validation Before ANY Commit

**ALWAYS run pre-commit hooks before committing ANY changes:**

```bash
pre-commit run --all-files
```

This is **non-negotiable**. All code must pass formatting and style checks before being committed.

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [Validation Workflow](#validation-workflow)
4. [Testing Strategy](#testing-strategy)
5. [Development Conventions](#development-conventions)
6. [CI/CD Awareness](#cicd-awareness)
7. [Common Tasks](#common-tasks)
8. [Troubleshooting](#troubleshooting)

---

## Project Overview

**quantms** is an nf-core bioinformatics best-practice analysis pipeline for **Quantitative Mass Spectrometry (MS)**. It supports three major analytical workflows:

- **DDA-LFQ**: Data-dependent acquisition with label-free quantification
- **DDA-ISO**: Data-dependent acquisition with isobaric labeling (TMT, iTRAQ)
- **DIA-LFQ**: Data-independent acquisition with label-free quantification

**Key Features:**

- Built with Nextflow DSL2
- Integrates multiple search engines: Comet, MSGF+, Sage, DIA-NN
- Uses OpenMS tools for proteomics processing
- Statistical analysis with MSstats
- Quality control with pmultiqc
- Complies with nf-core standards

**Repository:** https://github.com/bigbio/quantms
**Documentation:** https://quantms.readthedocs.io/

---

## Technology Stack

### Core Technologies

- **Nextflow**: >=25.04.0 (DSL2 syntax)
- **nf-schema plugin**: 2.5.1 (parameter validation)
- **nf-test**: Testing framework (config: `nf-test.config`)
- **nf-core tools**: Pipeline standards and linting
- **Containers**: Docker/Singularity/Apptainer/Podman (Conda deprecated)

### Key Configuration Files

- `nextflow.config` - Main pipeline configuration (541 lines)
- `nextflow_schema.json` - Parameter schema (auto-generated)
- `nf-test.config` - Testing configuration
- `.nf-core.yml` - nf-core compliance settings
- `modules.json` - Module dependencies
- `.pre-commit-config.yaml` - Pre-commit hooks

### Project Structure

```
quantms/
├── main.nf                    # Pipeline entry point
├── workflows/                 # Main workflows (quantms.nf, lfq.nf, tmt.nf, dia.nf)
├── subworkflows/local/        # Reusable subworkflows
├── modules/                   # Process definitions
│   ├── local/                 # Custom modules
│   ├── bigbio/                # BigBio shared modules
│   └── nf-core/               # nf-core modules
├── conf/                      # Configuration files
│   ├── base.config            # Resource definitions
│   ├── modules/               # Module-specific configs
│   └── tests/                 # Test profile configs (13 profiles)
├── tests/                     # nf-test test cases
├── bin/                       # Utility scripts (R scripts for MSstats)
└── assets/                    # Pipeline assets and schemas
```

---

## Validation Workflow

### 1. Pre-commit Hooks (MANDATORY)

**Installation:**

```bash
pip install pre-commit
pre-commit install  # Install git hooks (one-time setup)
```

**Run before EVERY commit:**

```bash
pre-commit run --all-files
```

**Configured Hooks** (`.pre-commit-config.yaml`):

1. **Prettier** (v3.1.0 with prettier@3.6.2)
   - Formats code consistently across multiple file types
   - Auto-fixes formatting issues

2. **trailing-whitespace**
   - Removes trailing whitespace (preserves markdown linebreaks)

3. **end-of-file-fixer**
   - Ensures files end with a single newline

**Excluded Files:**

- `CHANGELOG.md` (manually maintained)
- `modules/nf-core/**` (managed by nf-core)
- `subworkflows/nf-core/**` (managed by nf-core)
- `*.snap` (test snapshots)

**Auto-fix in CI:**
If you forget to run pre-commit locally, comment on your PR:

```
@nf-core-bot fix linting
```

The bot will run pre-commit and push fixes automatically.

### 2. Pipeline Linting (RECOMMENDED)

**Run before creating PR:**

```bash
nf-core pipelines lint
```

**For master branch PRs:**

```bash
nf-core pipelines lint --release
```

This validates:

- nf-core pipeline standards compliance
- File structure and naming
- Configuration completeness
- Documentation requirements

### 3. Schema Validation (REQUIRED for parameter changes)

**After adding/modifying parameters in `nextflow.config`:**

```bash
nf-core pipelines schema build
```

This updates `nextflow_schema.json` with interactive prompts to add descriptions and validation rules.

---

## Testing Strategy

### Testing Philosophy

**Do NOT run the full test suite before every commit.** The CI system runs comprehensive tests automatically. Instead:

1. **Pre-commit hooks**: ALWAYS (fast, catches style issues)
2. **Targeted tests**: Run tests relevant to your changes
3. **CI validation**: Trust the CI to catch integration issues

### When to Run Tests Locally

#### 🟢 Documentation/Config-Only Changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigbio/quantms](https://github.com/bigbio/quantms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
