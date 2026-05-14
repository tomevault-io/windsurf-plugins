---
trigger: always_on
description: Validates configuration before training to prevent wasted GPU time.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Merlina is a magical LLM training system with support for ORPO, DPO, SimPO, CPO, IPO, KTO, and SFT training modes. It provides a delightful wizard-themed web interface for fine-tuning language models with LoRA adapters. The system supports flexible dataset loading from multiple sources and automatic chat template formatting.

**New in v1.3:**
- **Messages Format Support**: Automatic detection and conversion of common chat dataset format with multi-turn conversation support

**New in v1.2:**
- **SFT Mode**: Train with only chosen responses (rejected field not required)
- Dynamic UI that adapts based on selected training mode

**New in v1.1:**
- Persistent job storage with SQLite database
- Real-time WebSocket updates during training
- Pre-flight validation to catch errors before training
- Detailed metrics tracking and history
- Support for both HuggingFace model IDs and local model directories
- Private/public repository control for HuggingFace Hub uploads

## Project Spirit

Merlina is designed to make LLM fine-tuning approachable and delightful, not intimidating. The entire project is themed around a cute anime magician girl who guides you through the training process with charm and style.

The philosophy is simple: **powerful ML training shouldn't require arcane command-line incantations or PhD-level expertise**. Whether you're a researcher experimenting with preference optimization or a hobbyist trying to make your first fine-tuned model, Merlina welcomes you with a friendly interface, clear feedback, and magical animations.

Behind the whimsical wizard theme is a carefully architected system that handles the complex stuff (dataset formatting, memory management, quantization, GPU optimization) so you can focus on what matters: creating better models. The UI is playful, but the engineering is serious.

**Key principles:**
- **Approachable**: No config files to edit, no command-line arguments to memorize
- **Visual**: See your datasets, preview formatting, watch training progress in real-time
- **Safe**: Pre-flight checks catch errors before wasting GPU time
- **Flexible**: Support for multiple dataset sources, formats, and model types
- **Delightful**: Smooth animations, clear status updates, and a touch of magic

When working on Merlina, preserve this spirit. Keep the code clean and well-documented, but remember that the end goal is to make someone smile while they train their model.

## Development Commands

### Running the Application
```bash
python merlina.py
# Access UI at http://localhost:8000
# API docs at http://localhost:8000/api/docs
```

### Testing
```bash
# Run all tests
python -m pytest tests/

# Run individual test files
python tests/test_tokenizer_formatter.py
python tests/test_dataset_loaders.py
python tests/test_pipeline.py
python tests/test_api_endpoints.py
```

### Installing Dependencies
```bash
pip install -r requirements.txt
```

### Version Management

Merlina uses [Semantic Versioning](https://semver.org/) (SemVer) for version control: `MAJOR.MINOR.PATCH`

**Version Files:**
- `version.py` - Single source of truth for version information
- `CHANGELOG.md` - Detailed change history following [Keep a Changelog](https://keepachangelog.com/)

**Bumping Versions:**
```bash
# Preview changes without applying
python bump_version.py [major|minor|patch] --dry-run

# Bump patch version (1.2.0 -> 1.2.1) for bug fixes
python bump_version.py patch

# Bump minor version (1.2.0 -> 1.3.0) for new features
python bump_version.py minor

# Bump major version (1.2.0 -> 2.0.0) for breaking changes
python bump_version.py major

# Add a release name
python bump_version.py minor --release-name "Feature Name"
```

**What the bump script does:**
1. Updates `version.py` with new version and release date
2. Adds new version section to `CHANGELOG.md`
3. Creates git tag (e.g., `v1.3.0`)
4. Provides next steps for committing and pushing

**Semantic Versioning Rules:**
- **MAJOR (X.0.0)**: Breaking changes, incompatible API changes
  - Example: Removing endpoints, changing data formats, incompatible config changes
- **MINOR (1.X.0)**: New features, backwards-compatible additions
  - Example: New training modes, additional API endpoints, new UI features
- **PATCH (1.2.X)**: Bug fixes, backwards-compatible fixes
  - Example: Bug fixes, performance improvements, documentation updates

**Version Visibility:**
- API exposes version at `GET /version` endpoint
- Frontend displays version in footer (auto-loaded on page load)
- FastAPI docs show version in OpenAPI spec
- Logs include version on startup

**Release Process:**
1. Ensure all changes are committed
2. Run `python bump_version.py [type]` to bump version
3. Edit `CHANGELOG.md` to fill in the changes for the new version
4. Commit: `git commit -am "Bump version to X.Y.Z"`
5. Push: `git push origin <branch>`
6. Push tag: `git push origin vX.Y.Z`
7. Create GitHub release from tag with CHANGELOG excerpt

**Current Version:** See `version.py` or run `python -c "from version import __version__; print(__version__)"`

## Code Architecture

### Core Components

**Backend (merlina.py)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Schneewolf-Labs/Merlina](https://github.com/Schneewolf-Labs/Merlina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
