---
trigger: always_on
description: This document provides guidance for AI assistants (Claude Code, GitHub Copilot, etc.) working on this repository.
---

# AGENTS.md

This document provides guidance for AI assistants (Claude Code, GitHub Copilot, etc.) working on this repository.

## Overview

viral-ngs is a consolidated monorepo for viral NGS (Next-Generation Sequencing) analysis tools. It provides:

- **Core utilities**: Read manipulation, Illumina demultiplexing, file handling, QC
- **Assembly**: Genome assembly, scaffolding, gap filling
- **Classification**: Metagenomic classification, taxonomy filtering, k-mer analysis
- **Phylogenetics**: Variant calling, consensus generation, annotation

**Related resources:**
- Command-line documentation: https://viral-ngs.readthedocs.org/
- Higher-level pipelines: https://github.com/broadinstitute/viral-pipelines

---

## Development Environment

### Docker-Centric Development

Development is **intentionally docker-centric**. Developers need:
- Docker
- Git
- Text/code editor

### Development Workflow

1. Clone the repository:
   ```bash
   git clone https://github.com/broadinstitute/viral-ngs.git
   ```

2. Run the container with local checkout mounted:
   ```bash
   docker run -it --rm \
     -v $(pwd):/opt/viral-ngs/source \
     quay.io/broadinstitute/viral-ngs:main-core
   ```

3. If modifying conda dependencies, install them inside the container:
   ```bash
   micromamba install <packages>
   ```

4. Test code interactively:
   ```bash
   cd /opt/viral-ngs/source
   pytest -rsxX -n auto tests/unit
   ```

5. Push changes to GitHub for automated CI testing

### Running Tests

```bash
# Run all unit tests in the core image
docker run --rm \
  -v $(pwd):/opt/viral-ngs/source \
  quay.io/broadinstitute/viral-ngs:main-core \
  pytest -rsxX -n auto /opt/viral-ngs/source/tests/unit

# Run specific module tests
docker run --rm \
  -v $(pwd):/opt/viral-ngs/source \
  quay.io/broadinstitute/viral-ngs:main-classify \
  pytest -rsxX -n auto /opt/viral-ngs/source/tests/unit/classify
```

**Important: Testing source code changes requires re-installing the package.**
The `-v` mount makes your local files visible on disk, but `viral_ngs` is already installed as a package inside the container image. Python imports resolve to the *installed* copy, not your mounted source files. If you've modified files under `src/viral_ngs/`, you must re-install before running tests:

```bash
# Run tests with local source changes applied
docker run --rm \
  -v $(pwd):/opt/viral-ngs/source \
  quay.io/broadinstitute/viral-ngs:main-core \
  bash -c "pip install -e /opt/viral-ngs/source --quiet && pytest -rsxX -n auto /opt/viral-ngs/source/tests/unit"
```

Changes to test files (`tests/`) and test inputs (`tests/input/`) are picked up automatically via the volume mount — the re-install is only needed when modifying the `src/viral_ngs/` package code.

Running pytest directly on the host will generally not work — most dependencies (bioinformatics tools, conda packages) are only available inside the Docker containers. Always test inside Docker.

**Test conventions:**
- Uses pytest (not nose or unittest)
- Test files in `tests/unit/<module>/`
- Test input files in `tests/input/<TestClassName>/`
- Access via `viral_ngs.core.file.get_test_input_path(self)`
- Custom marker: `@pytest.mark.slow` for slow tests

---

## Code Architecture

### Directory Structure

```
viral-ngs/
├── pyproject.toml              # Package configuration
├── src/viral_ngs/
│   ├── __init__.py             # Version detection
│   ├── py.typed                # PEP 561 marker
│   │
│   ├── # Command modules (CLI entry points)
│   ├── illumina.py             # Illumina demux commands
│   ├── read_utils.py           # Read manipulation commands
│   ├── assembly.py             # Assembly commands
│   ├── metagenomics.py         # Classification commands
│   ├── interhost.py            # Phylo commands
│   │
│   ├── core/                   # Core library (shared utilities + tool wrappers)
│   │   ├── __init__.py         # Tool/InstallMethod classes
│   │   ├── samtools.py         # Tool wrapper
│   │   ├── picard.py           # Tool wrapper
│   │   ├── file.py             # File utilities
│   │   ├── misc.py             # General utilities
│   │   └── ...
│   │
│   ├── assemble/               # Assembly tool wrappers
│   │   ├── __init__.py
│   │   ├── spades.py
│   │   └── ...
│   │
│   ├── classify/               # Classification tool wrappers
│   │   ├── __init__.py
│   │   ├── kraken2.py
│   │   └── ...
│   │
│   └── phylo/                  # Phylogenetics tool wrappers
│       ├── __init__.py
│       ├── mafft.py
│       └── ...
│
├── docker/
│   ├── Dockerfile.baseimage    # Base with conda/python
│   ├── Dockerfile.core         # Core tools
│   ├── Dockerfile.assemble     # + assembly tools
│   ├── Dockerfile.classify     # + classification tools
│   ├── Dockerfile.phylo        # + phylo tools
│   ├── Dockerfile.mega         # All tools combined
│   ├── install-conda-deps.sh
│   └── requirements/
│       ├── baseimage.txt
│       ├── core.txt
│       ├── core-x86.txt        # x86-only core packages
│       ├── assemble.txt
│       ├── assemble-x86.txt    # x86-only assembly packages
│       ├── classify.txt
│       ├── classify-x86.txt    # x86-only classify packages
│       ├── phylo.txt

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [broadinstitute/viral-ngs](https://github.com/broadinstitute/viral-ngs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
