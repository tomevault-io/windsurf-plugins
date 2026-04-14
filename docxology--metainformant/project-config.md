---
trigger: always_on
description: - **`output/`**: All outputs from tests and real runs must go here by default. Treat as ephemeral and reproducible. Safe to delete. **CRITICAL**: Only program-generated execution outputs belong here. Never create documentation, reports, test scripts, or planning documents in `output/`. See `output/.cursorrules` for the complete policy.
---

# METAINFORMANT Cursor Rules

## Directory Structure

### Core Directories
- **`output/`**: All outputs from tests and real runs must go here by default. Treat as ephemeral and reproducible. Safe to delete. **CRITICAL**: Only program-generated execution outputs belong here. Never create documentation, reports, test scripts, or planning documents in `output/`. See `output/.cursorrules` for the complete policy.
- **`config/`**: Repository-level configuration files and options. Read via `metainformant.core.config`; allow env overrides.
- **`data/`**: Inputs such as datasets and local databases. Read-mostly, organized by domain and version.
- **`docs/`**: All documentation organized by domain (`docs/<domain>/<topic>.md`). NEVER create new docs in repo root.
- **`src/metainformant/`**: Source code organized by domain modules.
- **`tests/`**: Test files mirroring source structure (`tests/test_<module>_<submodule>.py`).

### Domain-Specific Output Paths
- **Core**: `output/core/<utility>/` (e.g., `cache/`, `logs/`, `workflows/`)
- **DNA**: `output/dna/<analysis_type>/` (e.g., `phylogeny/`, `population/`, `variants/`)
- **RNA**: `output/amalgkit/<species>/<step>/` (e.g., `quant/`, `work/`, `logs/`)
- **Protein**: `output/protein/<analysis_type>/` (e.g., `structures/`, `alignments/`)
- **Epigenome**: `output/epigenome/<analysis_type>/` (e.g., `methylation/`, `chipseq/`, `atac/`)
- **Ontology**: `output/ontology/<type>/` (e.g., `go/`, `annotations/`, `queries/`)
- **Phenotype**: `output/phenotype/<dataset>/` (e.g., `traits/`, `life_course/`, `curation/`)
- **Ecology**: `output/ecology/<analysis>/` (e.g., `community/`, `environmental/`, `interactions/`)
- **Math**: `output/math/<type>/` (e.g., `simulations/`, `models/`, `plots/`)
- **GWAS**: `output/gwas/<analysis_type>/` (e.g., `association/`, `plots/`, `qc/`)
- **Information Theory**: `output/information/<analysis_type>/` (e.g., `entropy/`, `complexity/`)
- **Life Events**: `output/life_events/<workflow>/` (e.g., `embeddings/`, `models/`, `plots/`)
- **Visualization**: `output/visualization/<type>/` (e.g., `plots/`, `animations/`, `trees/`)
- **Simulation**: `output/simulation/<type>/` (e.g., `sequences/`, `ecosystems/`)
- **Single-Cell**: `output/singlecell/<analysis>/` (e.g., `preprocessing/`, `clustering/`)
- **Quality**: `output/quality/<dataset>/` (e.g., `fastq/`, `reports/`)
- **Networks**: `output/networks/<network_type>/` (e.g., `ppi/`, `regulatory/`, `pathways/`)
- **ML**: `output/ml/<task>/` (e.g., `classification/`, `regression/`, `features/`)
- **Multi-Omics**: `output/multiomics/<integration>/` (e.g., `integrated/`, `plots/`)
- **Long Read**: `output/longread/<analysis_type>/` (e.g., `basecalling/`, `assembly/`, `methylation/`)
- **Metagenomics**: `output/metagenomics/<analysis_type>/` (e.g., `amplicon/`, `assembly/`, `functional/`)
- **Structural Variants**: `output/structural_variants/<analysis_type>/` (e.g., `detection/`, `annotation/`)
- **Spatial**: `output/spatial/<analysis_type>/` (e.g., `clustering/`, `deconvolution/`, `integration/`)
- **Pharmacogenomics**: `output/pharmacogenomics/<analysis_type>/` (e.g., `alleles/`, `clinical/`, `reports/`)
- **Metabolomics**: `output/metabolomics/<analysis_type>/` (e.g., `identification/`, `quantification/`, `pathways/`)

## Path and I/O

### Path Handling Rules
- **DO NOT write outside `output/`** unless an explicit user path is provided.
- Prefer functional APIs that accept a destination path; if omitted, default to `output/` with a sensible subpath.
- Use `metainformant.core.io` for file I/O (JSON/JSONL/CSV/TSV) and gzip-aware operations.
- Use `metainformant.core.paths` for path handling and containment checks.
- Use `metainformant.core.cache` for small JSON caches under `output/` if needed.
- Always use `Path` objects from `pathlib` for path manipulation.
- Expand user paths (`~`) and resolve to absolute paths using `paths.expand_and_resolve()`.
- Validate paths with `paths.is_within()` to prevent directory traversal attacks.

### Temporary Files and Disk Space Management

**CRITICAL**: On external drives, system `/tmp` may be full. Always use repository-local temp directories.

#### Temp Directory Strategy
- **Primary**: Use `.tmp/` directory in repository root (on external drive with 5.5TB space)
- **Python temp**: Set `TMPDIR` environment variable to `.tmp/python/`
- **Bash heredocs**: Set `TMPDIR` environment variable to `.tmp/bash/`
- **Cache files**: Use `.cache/` directory in repository root
- **Never rely on system `/tmp`**: It may be a small tmpfs (RAM) filesystem

#### Implementation Pattern

```python
import os
import tempfile
from pathlib import Path

# Set temp directory to repository root
REPO_ROOT = Path(__file__).resolve().parent.parent
TEMP_DIR = REPO_ROOT / ".tmp" / "python"
TEMP_DIR.mkdir(parents=True, exist_ok=True)

# Override Python temp directory
os.environ["TMPDIR"] = str(TEMP_DIR)
os.environ["TEMP"] = str(TEMP_DIR)
os.environ["TMP"] = str(TEMP_DIR)
tempfile.tempdir = str(TEMP_DIR)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/docxology) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
