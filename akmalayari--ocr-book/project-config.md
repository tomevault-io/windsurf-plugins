---
trigger: always_on
description: Reference file for AI coding agents. This project is a Python CLI pipeline that OCRs an entire book (page photos) into Markdown, using **PaddleOCR-VL-1.5** served locally by **llama-server** (llama.cpp, Vulkan backend).
---

# AGENTS.md — ocr-book

Reference file for AI coding agents. This project is a Python CLI pipeline that OCRs an entire book (page photos) into Markdown, using **PaddleOCR-VL-1.5** served locally by **llama-server** (llama.cpp, Vulkan backend).

---

## Project Overview

| | |
|---|---|
| **Name** | ocr-book |
| **Language** | Python 3.10 |
| **Type** | Local CLI tool (no web server, no deployment) |
| **Goal** | Convert a folder of book page photos into a single Markdown file, with HTML tables, extracted figures, and optional Obsidian export. |
| **Primary doc language** | English (docstrings, comments, README, docs). Commit messages in English. |

### Tech Stack

| Component | Role |
|-----------|------|
| **llama-server** (llama.cpp, Vulkan) | Local VLM inference of the PaddleOCR-VL-1.5 GGUF model |
| **paddleocr** (installed from git repo, not PyPI) | Orchestration: layout detection → prompt routing → VLM calls |
| **paddlepaddle 3.3.1** | CPU layout detection (ppdoclayout) |
| **paddlex[ocr] 3.4.3** | Table sub-pipeline (OTSL → HTML) |
| **openai** | HTTP client for paddleocr's `llama-cpp-server` backend |
| **requests** | llama-server health polling |
| **natsort** | Natural sorting of files and folders |

---

## Project Structure

```
ocr-livre/
├── src/                          # Main source code (10 modules)
│   ├── main.py                   # CLI entry point (argparse)
│   ├── config.py                 # Config dataclass (all default values)
│   ├── pipeline.py               # Full orchestration (servers, parallelism, parts, fallback)
│   ├── ocr_client.py             # OCR of an image via PaddleOCRVL + retry/timeout
│   ├── postprocess.py            # Text cleanup, page number extraction, headers
│   ├── images.py                 # Collection, renaming, copying from subfolders
│   ├── obsidian.py               # Obsidian export (wikilinks, figure migration)
│   ├── progress.py               # Configured logging + statistics (Stats dataclass)
│   ├── pdf.py                    # PDF processing (text extraction or render → OCR)
│   └── epub.py                   # EPUB extraction (Pandoc-based)
│
├── docs/                         # Project documentation
│   ├── architecture/overview.md  # Detailed pipeline architecture
│   ├── SETUP.md                  # Installation instructions
│   ├── tested.md                 # Results of all experiments
│   ├── issues.md                 # Work in progress / planned features
│   ├── dev/                      # Patches and development scripts
│   │   ├── apply_paddlex_patch_otsl.py      # Required patch: per-region VLM error handling
│   │   ├── apply_paddlex_patch_parallel.py  # Optional patch: intra-page parallelism
│   │   ├── paddlex_patch_otsl.md            # OTSL patch doc
│   │   └── paddlex_patch_parallel.md        # Parallel patch doc
│   └── paddleocr/                # Internal PaddleOCR docs (config, output, performance)
│
├── draft/                        # Explorations, informal tests, throwaway scripts (gitignored)
├── photos/                       # Source images (gitignored)
├── output/                       # Generated Markdown, logs, figures, reports (gitignored)
├── memory/                       # Inter-session memory notes (gitignored)
│
├── environment.yml               # Conda dependencies (Python 3.10 + pip packages)
├── setup.py                      # Automated installation script (conda env + patches)
├── README.md                     # User documentation
└── .gitignore                    # Python + data folders + draft/ + memory/
```

---

## Installation and Build

### System Prerequisites
- Windows (developed and tested on Windows)
- [miniforge](https://github.com/conda-forge/miniforge) or Anaconda
- [llama-server](https://github.com/ggerganov/llama.cpp) compiled with Vulkan (or another GPU backend)
- GGUF model: [PaddleOCR-VL-1.5-GGUF](https://huggingface.co/PaddlePaddle/PaddleOCR-VL-1.5) (`.gguf` + `.mmproj.gguf`)

### Installation Commands

```bash
# Full setup (creates conda env, installs deps, applies patch)
python setup.py

# Activate environment
conda activate ocr-livre

# Verify everything works
python -c "from paddleocr import PaddleOCRVL; print('OK')"
```

The `setup.py` script:
1. Removes the old conda env `ocr-livre` (if it exists)
2. Creates the env from `environment.yml`
3. Installs `paddleocr` from the git repo (the PyPI version does not contain the `llama-cpp-server` backend)
4. Applies the required patch `docs/dev/apply_paddlex_patch_otsl.py`

### Required paddlex patches

| Patch | Status | Command |
|-------|--------|----------|
| **OTSL** (`apply_paddlex_patch_otsl.py`) | **Required** | `python docs/dev/apply_paddlex_patch_otsl.py` |
| **Parallel** (`apply_paddlex_patch_parallel.py`) | Optional (~30% gain) | `python docs/dev/apply_paddlex_patch_parallel.py` |

These patches modify the installed file at `sys.prefix/Lib/site-packages/paddlex/inference/pipelines/paddleocr_vl/pipeline.py`. They accept `--check` and `--revert` arguments.

---

## Execution

All commands are run from `src/` with the `ocr-livre` environment activated.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akmalayari/ocr-book](https://github.com/akmalayari/ocr-book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
