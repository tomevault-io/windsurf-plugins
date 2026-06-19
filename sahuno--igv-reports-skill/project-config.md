---
trigger: always_on
description: Use when the user wants an HTML, clickable, browseable, offline, or emailable viewer of genomic data — phrases like "HTML IGV report", "offline IGV", "self-contained HTML", "clickable viewer", "create_report", "igv-reports", "email this viewer", or any browseable HTML of reads at variants, fusion breakpoints, SV junctions, viral integrations, ChIP peaks, ROIs, or ONT 5mC/5hmC methylation views at promoters/gene bodies/DMRs. Trigger even when the user doesn't say "igv-reports" — giveaway is HTML/
---


# igv-reports

This skill builds **self-contained HTML genomic-region reports** with
[igv-reports](https://github.com/igvteam/igv-reports) (`create_report`).
Each report is a single browseable HTML containing the igv.js viewer plus
embedded data slices for every region. No server, no internet, no IGV
install needed at view time.

The skill has three entry points:
- **build** — one-shot: sites BED + BAM(s) ± VCF → HTML.
- **cohort** — multi-sample driver from a samplesheet → per-sample HTMLs + index.
- **prep-track** — utility: convert plain-gzip GFF/GTF/BED.gz into a
  bgzip + tabix-indexed track that igv-reports can load.

## What this skill is (and is not)

This skill is a **driver layer** on top of the upstream `igv-reports`
Python package by the IGV team
([github.com/igvteam/igv-reports](https://github.com/igvteam/igv-reports)).
The naming is unavoidable — both share the `igv-reports` name.

| Component | Source | Role |
|---|---|---|
| `create_report` CLI | upstream PyPI package `igv-reports` | does the actual HTML rendering |
| `scripts/build_igvreports.py` | **this skill** | wraps `create_report` with default-track resolution, cohort/samplesheet mode, SIF auto-detect |
| `scripts/verify_{report,cohort,anchors}.py` | **this skill** | post-render structural + content audits (not in upstream) |
| `scripts/prep_track.sh` | **this skill** | bgzip+tabix utility for annotation tracks |

The skill is not on PyPI — it's a directory of scripts. Use it by either
cloning this repo or copying `scripts/` next to your data.

## Off-MSKCC quickstart

Defaults assume MSKCC HPC (lab SIF, `databases_config.yaml`, `/data1/greenbab`
bind). Anywhere else:

```bash
# 1. Install the UPSTREAM igv-reports package (provides `create_report`).
#    Use -U to pick up the latest fixes; skill requires >=1.16.0.
pip install -U 'igv-reports>=1.16.0'

# 2. Get this skill's wrapper scripts (one of):
#    - clone:  git clone https://github.com/sahuno/igv-reports-skill.git
#              cd igv-reports-skill
#    - or copy scripts/ next to your project

# 3. Run the wrapper, bypassing the lab databases YAML with explicit paths:
python scripts/build_igvreports.py \
    --genome hg38 \
    --sites sites.hg38.bed \
    --bam tumor.bam normal.bam \
    --fasta /path/to/hg38.fa \
    --no-default-tracks \
    --extra-track /path/to/your_cpg_islands.bed.gz \
    --extra-track /path/to/gencode.v47.annotation.gff3.gz \
    --output report.hg38.html
```

If you only need raw `create_report` (no cohort mode, no verifiers, no
auto-tracks), skip the skill entirely and use upstream directly —
see [igvteam/igv-reports](https://github.com/igvteam/igv-reports) docs.

Environment overrides (all optional):

| Var | Effect |
|---|---|
| `IGV_REPORTS_DB_CONFIG` | Path to your own databases YAML (same schema as the lab's) |
| `IGV_REPORTS_SIF` | Path to your own `igv-reports` apptainer SIF |
| `SAMTOOLS_SIF_DEFAULT` | Path to your own `samtools` SIF (verifier only) |
| `IGV_REPORTS_BIND` | Colon-separated bind paths for singularity (default `/data1/greenbab`). Empty string disables binding. |

Driver flags `--fasta` and `--no-default-tracks` let you skip the databases
YAML entirely without setting any env var. `--no-apptainer` forces the
PATH `create_report` path even on a SLURM node. The hermetic `tests/unit/`
suite runs anywhere with `pytest` + Python ≥ 3.10.

## When to use which entry point

| User request | Entry point |
|---|---|
| "Make an HTML for these 5 SV breakpoints in tumor.bam" | **build** |
| "Give me one HTML per patient for the cohort integration calls" | **cohort** |
| "create_report fails with 'not BGZF' on this gencode" | **prep-track** |

## Defaults (locked in)

- Tracks always loaded, top-to-bottom in the viewer:
  1. CpG islands (BED, plain or bgzipped)
  2. Gencode full annotation (GFF3.gz, **transcripts + exons + CDS + UTRs**, NOT a gene-level-only file)
  3. RepeatMasker (BED.gz, bgzipped + tabix-indexed)
  Plus the user's BAM(s), VCF, and any extra tracks they pass.
- `--flanking 300` bp on either side of each site (good for SV breakpoints
  and point variants alike). Override per call if needed.
- `--standalone` so the HTML is offline-viewable.
- Output filename includes the genome tag — e.g. `cohort.hg38.html` —
  to pass `enforce-genome-tag.sh`.
- Reference FASTA is resolved from `databases_config.yaml`:
  `/data1/greenbab/users/ahunos/apps/llm_configs/claude/profiles/databases/databases_config.yaml`
  (lab default; override with `$IGV_REPORTS_DB_CONFIG` or `--no-default-tracks` + `--fasta`).
  Supported genome IDs: `hg38`, `mm10`, `mm39`, `t2t_CHM13v2_plusY`, `GRCh37`.
- Per-genome default track availability is recorded in
  `references/databases_config_paths.md` — read it before assembling tracks
  so the skill doesn't try to load a track that doesn't exist for the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sahuno/igv-reports-skill](https://github.com/sahuno/igv-reports-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
