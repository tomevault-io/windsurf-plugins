---
trigger: always_on
description: **Analysis Date:** 2025-12-26
---

# pyflow-ChIPseq: Repository Structure and Overview

**Analysis Date:** 2025-12-26
**Branch:** modernize-2025 (based on pairend)
**Purpose:** Documentation for understanding and modernizing the ChIP-seq pipeline

---

## Table of Contents

1. [Repository Purpose](#repository-purpose)
2. [Directory Structure](#directory-structure)
3. [Key Components](#key-components)
4. [Technology Stack](#technology-stack)
5. [Workflow Overview](#workflow-overview)
6. [Configuration System](#configuration-system)
7. [Modernization Priorities](#modernization-priorities)

---

## Repository Purpose

**pyflow-ChIPseq** is a Snakemake-based bioinformatics pipeline for processing and analyzing Chromatin Immunoprecipitation Sequencing (ChIP-seq) data. It automates the complete workflow from raw sequencing reads to peak calling and chromatin state analysis.

### Key Features

- Processes both public (GEO/SRA) and in-house ChIP-seq data
- Supports single-end and paired-end sequencing reads
- Handles both short (<70bp) and long (>70bp) reads
- Automated quality control and reporting
- Advanced downstream analysis (super-enhancers, chromatin states)
- Cluster deployment support (SLURM/DRMAA)

### Publication

This pipeline was published in the Journal of Visualized Experiments (JOVE):
*"An Integrated Platform for Genome-wide Mapping of Chromatin States Using High-throughput ChIP-sequencing in Tumor Tissues"*

---

## Directory Structure

```
pyflow-ChIPseq/
├── Snakefile                    # Main workflow definition (21,957 bytes)
├── config.yaml                  # Pipeline configuration and parameters
├── cluster.json                 # Cluster job submission resource specs
├── samples.json                 # Sample metadata (auto-generated)
│
├── sample2json.py               # Metadata converter (TSV → JSON)
├── sbatch_cluster.py            # SLURM job submission wrapper
├── pyflow-ChIPseq.sh            # Main execution script (SLURM)
├── pyflow-drmaa-ChIPseq.sh      # DRMAA-based execution (LSF)
├── jobscript.sh                 # Job script template
│
├── scripts/
│   └── sraDownload.R            # SRA data downloader
│
├── SRR.txt                      # Sample metadata template (GEO data)
├── meta.txt                     # Custom metadata template (in-house data)
│
├── README.md                    # User documentation
├── LICENSE                      # MIT License
├── rulegraph.png                # Workflow DAG visualization
├── GEO_rulegraph.png            # Alternative workflow visualization
└── TCGA_related files           # TCGA barcode documentation

### Output Directories (Created During Execution)

- `00log/` - Log files for all rules
- `01seq/` - Merged FASTQ files
- `02fqc/` - FastQC quality control reports
- `03aln/` - Aligned BAM files and indices
- `04aln_downsample/` - Downsampled BAM files
- `05phantompeakqual/` - Phantom peak quality metrics
- `06bigwig_inputSubtract/` - Input-subtracted bigWig tracks
- `07bigwig/` - RPKM-normalized bigWig tracks
- `08peak_macs1/` - MACS1 peak calls
- `09peak_macs2/` - MACS2 peak calls
- `10multiQC/` - MultiQC quality summary report
- `11superEnhancer/` - Super enhancer calls (ROSE)
- `12bed/` - BED format files
- `13chromHMM/` - Chromatin state predictions
```

---

## Key Components

### 1. Snakefile (Main Workflow)

The core orchestrator containing all pipeline rules, organized into:

#### Data Preparation Rules
- `merge_fastqs` - Combines multiple FASTQ files per sample/mark
- `fastqc` - Quality control on raw reads

#### Alignment Rules
- `align` - BWA mapping (supports both single/paired-end, short/long reads)
  - Short reads (<70bp): Uses `bwa aln + sampe/samse`
  - Long reads (>70bp): Uses `bwa mem`
  - Integrates `samblaster` for duplicate marking

#### BAM Processing Rules
- `flagstat_bam` - Counts mapped/unmapped reads
- `down_sample` - Normalizes read depth (default: 50M reads)

#### Quality Control Rules
- `phantom_peak_qual` - ChIP-seq quality assessment
- `multiQC` - Comprehensive quality report aggregation

#### Peak Calling Rules
- `call_peaks_macs1` - MACS v1.4.2 peak calling (narrow, nomodel)
- `call_peaks_macs2` - MACS v2.1.1 peak calling (broad peaks)

#### Visualization Rules
- `make_bigwigs` - RPKM-normalized bigWig generation
- `make_inputSubtract_bigwigs` - Input-subtracted bigWig tracks

#### Advanced Analysis Rules
- `superEnhancer` - Super-enhancer identification via ROSE
- `bam2bed` - BAM to BED conversion
- `chromHmm_binarize` - Binarize data for ChromHMM
- `chromHmm_learn` - Learn chromatin state models

### 2. sample2json.py (Python 3)

**Purpose:** Converts tab-delimited metadata into JSON format for Snakemake

**Input Format (meta.txt):**
```
sample_name     fastq_name              factor          reads
LKR10           Input_LKR10_1.fq.gz     Input           R1
LKR10           Input_LKR10_2.fq.gz     Input           R2
V6_5            Mll4_V6_5_1.fq.gz       Mll4            R1
```

**Output:** `samples.json` with nested structure:
```json
{
  "sample_name": {
    "factor": {
      "R1": ["path/to/file1.fq.gz"],
      "R2": ["path/to/file2.fq.gz"]
    }
  }
}
```

**Key Functions:**
- Walks directory to find all `.fq.gz` or `.fastq.gz` files
- Validates that declared files exist
- Groups files by sample, factor, and read direction

### 3. sbatch_cluster.py (Python 3)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crazyhottommy/pyflow-ChIPseq](https://github.com/crazyhottommy/pyflow-ChIPseq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
