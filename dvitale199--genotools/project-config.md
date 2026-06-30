---
trigger: always_on
description: **AI-Assisted Development Guide for GenoTools**
---

# GenoTools AI Development Guide

**AI-Assisted Development Guide for GenoTools**

This guide provides patterns, conventions, and requirements for AI coding assistants working on the GenoTools codebase.

---

## Quick Reference

### Before You Start
1. Read this entire document
2. Understand the pipeline architecture (CLI → QC classes → PLINK execution)
3. Know the standard return dictionary format
4. Verify external tool availability (PLINK, PLINK2, KING)

### Critical Rules
- **Always validate inputs** at the start of every method
- **Always return the standard output dictionary** format
- **Never break the pipeline chain** - methods must produce valid PLINK2 pfiles
- **Log everything** via `concat_logs()`
- **Test with real PLINK files** - the pipeline depends on external executables

---

## Project Overview

GenoTools is a command-line tool for genotype quality control (QC) and ancestry prediction in genetic studies. It wraps PLINK/PLINK2 commands in a Python pipeline with ML-based ancestry inference.

### Key Components

| Component | Purpose |
|-----------|---------|
| `genotools/pipeline.py` | CLI argument parsing, pipeline orchestration |
| `genotools/qc.py` | `SampleQC` and `VariantQC` classes |
| `genotools/ancestry.py` | `Ancestry` class for ML predictions |
| `genotools/gwas.py` | `Assoc` class for GWAS/PCA |
| `genotools/utils.py` | Shell execution, file conversion helpers |
| `genotools/dependencies.py` | External tool management |

### Entry Points
```python
# Main CLI
genotools → genotools.__main__:handle_main

# Reference download
genotools-download → genotools.download_refs:handle_download
```

---

## Architecture

### Pipeline Flow
```
Input (bfile/pfile/vcf)
    ↓
Format Conversion → PLINK2 pfiles
    ↓
Ancestry Prediction (optional)
    ↓
Split by Ancestry
    ↓
┌─────────────────────────────────────┐
│ QC Pipeline (per ancestry group)    │
│   callrate → sex → het → related →  │
│   case_control → haplotype → hwe →  │
│   geno → ld → assoc                 │
└─────────────────────────────────────┘
    ↓
JSON Output + Cleaned Files
```

### Class Responsibilities

**SampleQC** - Sample-level quality control
- `run_callrate_prune()` - Remove samples with low call rates
- `run_sex_prune()` - Remove samples with sex discrepancies
- `run_het_prune()` - Remove samples with extreme heterozygosity
- `run_related_prune()` - Handle related/duplicate samples
- `run_confirming_kinship()` - Verify family relationships

**VariantQC** - Variant-level quality control
- `run_geno_prune()` - Remove variants with high missingness
- `run_case_control_prune()` - Remove variants with case/control differences
- `run_haplotype_prune()` - Remove haplotype-inconsistent variants
- `run_hwe_prune()` - Remove Hardy-Weinberg violating variants
- `run_ld_prune()` - Prune variants in linkage disequilibrium

**Ancestry** - Ancestry prediction
- PCA calculation and projection
- UMAP + XGBoost classification
- Container/cloud inference support
- Admixture handling

**Assoc** - Association analysis
- PCA preparation and execution
- GWAS execution
- Lambda/inflation calculation

---

## Code Conventions

### Standard Return Dictionary

**Every QC method must return this structure:**

```python
{
    'pass': bool,           # True if step completed successfully
    'step': str,            # Step identifier (e.g., 'callrate_prune')
    'metrics': {
        'outlier_count': int,  # Number of samples/variants pruned
        # ... other step-specific metrics
    },
    'output': {
        'pruned_samples': str,  # Path to pruned sample IDs (or None)
        'plink_out': str,       # Path to output pfiles (without extension)
        # ... other output files
    }
}
```

**Example:**
```python
out_dict = {
    'pass': process_complete,
    'step': step,
    'metrics': metrics_dict,
    'output': outfiles_dict
}
return out_dict
```

### Input Validation Pattern

**Every method must validate inputs before processing:**

```python
def run_some_prune(self, threshold=0.05):
    geno_path = self.geno_path
    out_path = self.out_path

    # 1. Check paths are set
    if geno_path is None or out_path is None:
        raise ValueError("Both geno_path and out_path must be set before calling this method.")

    # 2. Check input files exist
    if not os.path.exists(f'{geno_path}.pgen'):
        raise FileNotFoundError(f"{geno_path} does not exist.")

    # 3. Check parameter types
    if not isinstance(threshold, (int, float)):
        raise TypeError("threshold should be of type int or float.")

    # 4. Check parameter bounds
    if threshold < 0 or threshold > 1:
        raise ValueError("threshold should be between 0 and 1.")

    # ... proceed with implementation
```

### Shell Command Execution

**Use `shell_do()` for all external commands:**

```python
from genotools.utils import shell_do, concat_logs

# Execute PLINK command
plink_cmd = f"{plink2_exec} --pfile {geno_path} --mind {mind} --make-pgen psam-cols=fid,parents,sex,pheno1,phenos --out {out_path}"
shell_do(plink_cmd)

# Always log after execution
listOfFiles = [f'{out_path}.log']
concat_logs(step, out_path, listOfFiles)
```

### File Path Conventions

```python
# Input/output paths never include extensions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvitale199/GenoTools](https://github.com/dvitale199/GenoTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
