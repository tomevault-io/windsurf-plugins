---
trigger: always_on
description: Instructions for AI agents working on this repository.
---

# AGENTS.md — Personal Genome Pipeline

Instructions for AI agents working on this repository.

## Project Context

This is a public, open-source WGS (Whole Genome Sequencing) analysis pipeline designed for consumer hardware. It must be:
- **Generic**: No personal data, no hardcoded paths, no user-specific defaults
- **Reproducible**: Every command must work on any Linux amd64 machine with Docker
- **Well-documented**: Target audience includes non-bioinformaticians analyzing their own genome data

## Critical Rules

### No Personal Information
- NEVER commit personal paths (e.g., `/mnt/user/Multimedia/`, server hostnames, IP addresses)
- NEVER use specific sample names as defaults (use `your_name` or `$SAMPLE` placeholder)
- All environment variables must require user to set them: `${VAR:?Set VAR to...}`
- Docker mount point is always `:/genome` (not locale-specific)

### Script Conventions
- Shebang: `#!/usr/bin/env bash`
- Error handling: `set -euo pipefail`
- Parameters: `SAMPLE=${1:?Usage: $0 <sample_name>}`
- Environment: `GENOME_DIR=${GENOME_DIR:?Set GENOME_DIR to your data directory}`
- Docker: always use `--cpus N --memory Xg` limits, `-v "${GENOME_DIR}:/genome"` mount, `--rm` flag
- Add `--user root` when the container needs write access to bind mounts
- Validate all input files exist before running Docker commands
- Print clear status messages: step name, input files, output location

### Documentation Conventions
- Each pipeline step has a matching doc in `docs/XX-name.md` and script in `scripts/XX-name.sh`
- Docs must include: What it does, Why, Tool name, Docker image, Command, Output, Runtime estimate, Notes
- README.md step table must stay in sync with actual docs and scripts
- All Docker images must include the exact tag (not just `:latest` unless no versioned tags exist)

### Lessons Learned
- **ALWAYS update `docs/lessons-learned.md`** when encountering a new failure, workaround, or non-obvious behavior
- Include: what failed, why it failed, and the fix
- This is the most valuable document for future users — every Docker image issue, permission error, path confusion, and tool quirk should be recorded here

### Testing Changes
- After modifying any script, verify:
  1. No personal paths remain (`grep -r '/mnt/user\|watchtower\|sergio\|annais' scripts/ docs/`)
  2. All scripts use `GENOME_DIR` not `GENOMA_DIR`
  3. Docker mount is `:/genome` not `:/genoma`
  4. `shellcheck` passes on all scripts (if available)

### Git Practices
- Commit messages: descriptive, multi-line for large changes
- Never force-push to main
- Keep commits atomic: docs + scripts for the same feature in one commit

## Architecture

```
personal-genome-pipeline/
  README.md                    # Main entry point, pipeline overview, quick start
  LICENSE                      # GPL-3.0
  AGENTS.md                    # This file
  .gitignore                   # Excludes BAM, VCF, tar.gz, etc.
  docs/
    00-reference-setup.md      # One-time reference data downloads
    01-ora-to-fastq.md         # Step docs (one per pipeline step)
    ...
    20-mtoolbox.md
    hardware-requirements.md   # Disk, RAM, CPU, runtime breakdown
    vendor-guide.md            # Data formats from each WGS vendor
    chip-data-guide.md         # Using 23andMe/MyHeritage/AncestryDNA chip data
    interpreting-results.md    # Plain-language guide for non-experts
    multi-sample.md            # Comparing two or more samples (partners, family)
    glossary.md                # Alphabetical glossary of genomics terms
    quick-test.md              # Verify setup with public test data
    resources.md               # Free courses, databases, and learning resources
    troubleshooting.md         # Comprehensive troubleshooting by symptom
    lessons-learned.md         # Every failure and fix (KEEP UPDATED)
  scripts/
    01-ora-to-fastq.sh         # Step scripts (one per pipeline step)
    ...
    27-cpic-lookup.sh
    chip-to-vcf.sh             # Chip data converter (23andMe/MyHeritage/AncestryDNA → GRCh38 VCF)
    02a-alignment-bwamem2.sh   # Alternative aligner (BWA-MEM2, outputs to aligned_bwamem2/)
    03a-gatk-haplotypecaller.sh # Alternative caller (GATK HC, outputs to vcf_gatk/)
    03b-freebayes.sh           # Alternative caller (FreeBayes, outputs to vcf_freebayes/)
    04a-tiddit.sh              # Alternative SV caller (TIDDIT, outputs to sv_tiddit/)
    03c-strelka2-germline.sh   # Alternative small variant caller (Strelka2, outputs to vcf_strelka2/)
    benchmark-variants.sh      # Concordance benchmarking (bcftools isec / hap.py)
    run-all.sh                 # Orchestrator: runs all steps with parallelism
    validate-setup.sh          # Pre-flight check: Docker, refs, images, sample
    generate-report.sh         # Text summary report aggregating all outputs
  .github/workflows/
    lint.yml                   # ShellCheck + markdownlint
    smoke-test.yml             # Dry-run validation of all scripts
```

## Data Flow

```
User's FASTQ/BAM/VCF
  │
  ├─ Step 2: minimap2 alignment (FASTQ → BAM)
  ├─ Step 3: DeepVariant variant calling (BAM → VCF)
  │
  ├─ VCF-dependent steps: 6, 7, 9, 11, 12, 13, 14, 17, 25, 26
  ├─ BAM-dependent steps: 4, 10, 15, 16, 18, 19, 20, 21

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GeiserX/Personal-Genome-Pipeline](https://github.com/GeiserX/Personal-Genome-Pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
