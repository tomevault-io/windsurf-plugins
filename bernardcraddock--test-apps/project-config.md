---
trigger: always_on
description: This file provides guidance to Claude Code when working with the pumped-scripts repository.
---

# CLAUDE.md - Pumped Scripts Development Guide

This file provides guidance to Claude Code when working with the pumped-scripts repository.

## Project Overview

**pumped-scripts** is a multi-platform utility and experimental sandbox repository containing:
- Test applications and proof-of-concepts across multiple languages (Python, JavaScript, Bash, Java, Dart, Go, etc.)
- Architecture exploration and learning projects
- Integration experiments (MCP servers, GitHub integrations)
- Data processing tools (PDF extraction, image processing, diagram generation)

**Primary workspace**: `/test-apps` - All experimental projects and learning materials go here.

## Quick Reference Commands

### Python Projects
```bash
# Activate virtual environment (required for all Python work)
cd test-apps
source .venv/bin/activate  # macOS/Linux
# or: .venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt  # if exists in specific folder
# or: pip install pdf2image Pillow matplotlib numpy

# Run a Python script
python3 script_name.py

# Run tests
pytest -q test-folder -k quick  # fast tests
pytest test-folder              # full test suite
```

### JavaScript/Node Projects
```bash
cd test-apps
npm install

# Render mermaid diagrams
npm run mermaid:render -- path/to/file.mmd
npm run mermaid:render:all  # all .mmd files in current directory
```

### General
```bash
# View repository structure
ls -la test-apps/

# Check file type before viewing
file ./path/to/file
```

## Repository Structure

```
pumped-scripts/
├── .github/
│   ├── copilot-instructions.md  ← Master instructions (read by all AI models)
│   └── workflows/               ← CI/CD automation
├── test-apps/                   ← MAIN WORKSPACE FOR EXPERIMENTS
│   ├── .venv/                   ← Python virtual environment (auto-created)
│   ├── python/                  ← Python learning/test projects
│   ├── js/                       ← JavaScript/Node projects
│   ├── bash/                     ← Bash utilities
│   ├── mcp/                      ← Model Context Protocol experiments
│   ├── ai_studio/                ← Google AI Studio projects
│   ├── AntiGravity/              ← AntiGravity experiments
│   ├── clang/                    ← C/Objective-C projects
│   ├── dart/                     ← Dart/Flutter projects
│   ├── android/                  ← Android Studio projects
│   └── [other folders]
├── backend/                      ← Java/Maven backend (production)
├── frontend/                      ← Flutter frontend (production)
├── soafee/                       ← SOAFEE architecture docs
├── README.md                     ← Repository overview
└── CLAUDE.md                     ← This file
```

## Critical Conventions

### Python Projects in test-apps

**Virtual Environment:**
- Located at: `/test-apps/.venv`
- **Must activate before running**: `source .venv/bin/activate`
- Python version: 3.11+ (check with `python --version`)

**Common Libraries Already Available:**
- `pdf2image` - PDF to image conversion
- `Pillow` - Image processing
- `matplotlib` - Plotting and visualization
- `numpy` - Numerical operations
- System dependencies: macOS requires `brew install poppler`

**File Organization:**
- Use descriptive filenames: `extract_elevations.py`, `pdf_to_images.py`
- Output files go in appropriately named subdirectories (e.g., `extracted_images/`)
- Documentation: Create a `<folder>_README.md` explaining what the project does

**Execution Pattern:**
```bash
# Inspect file type first
file ./path/to/file

# Use appropriate viewer
less ./path/to/script.py    # for text
hexdump -C ./data.bin | head  # for binary

# Execute
./.venv/bin/python path/to/script.py
```

### Documentation Standards

Each major project in `test-apps/<folder>` should have:
- **Filename**: `<folder>_README.md` (e.g., `ai_studio/ai_studio_README.md`)
  - This naming convention makes READMEs uniquely discoverable
  - If a plain `README.md` is submitted, merge its contents into `<folder>_README.md`
- **Content**: What the project does, how to set it up, tools used, lessons learned
- **Process docs**: Document iteratively as you attempt different approaches (not just final results)

### Smoke Tests & CI Requirements

Each `test-apps/<folder>` should include:
- **Smoke test**: Fast executable script (must complete in <15 seconds, print `SMOKE: OK` on success)
  - Examples: `python/smoke-test.sh`, `js/tests/smoke-js-my-app.sh`
- **Automated tests**: Unit/functional tests in conventional locations
  - Python: `python/tests/` (run with `.venv/bin/python -m pytest -q python -k quick`)
  - JavaScript: `js/tests/` (run with simple shell script or npm command)
- **Document exact commands** in the `<folder>_README.md` for running smoke-tests and unit tests

## Build, Test, and Lint Commands

### Python (test-apps)
```bash
# Activate environment
source test-apps/.venv/bin/activate

# Run smoke test (must pass before committing)
bash test-apps/python/smoke-test.sh

# Run quick unit tests
pytest -q test-apps/python -k quick

# Full test suite
pytest test-apps/python
```

### JavaScript (test-apps)
```bash
cd test-apps
npm install

# Mermaid rendering
npm run mermaid:render -- path/to/diagram.mmd

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bernardcraddock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
