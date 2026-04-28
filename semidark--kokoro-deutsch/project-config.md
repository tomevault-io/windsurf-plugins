---
trigger: always_on
description: kokoro-deutsch is a training recipe for fine-tuning [Kokoro TTS](https://github.com/hexgrad/kokoro) (82M parameters, based on StyleTTS 2) for German. The project contains:
---

# AGENTS.md — kokoro-deutsch

## Project Overview

kokoro-deutsch is a training recipe for fine-tuning [Kokoro TTS](https://github.com/hexgrad/kokoro) (82M parameters, based on StyleTTS 2) for German. The project contains:

- A forked `kokoro/` inference package as a git submodule (`semidark/kokoro`, branch `main`), with German language code support
- A patched fork of `StyleTTS2/` as a git submodule (`semidark/StyleTTS2`, branch `main`)
- Original scripts for dataset preparation, voicepack extraction, and inference testing
- Training and troubleshooting docs split by purpose

- **Primary language:** Python 3.10–3.13
- **Package manager:** `uv` (lockfile: `uv.lock`)
- **Build backend:** hatchling
- **License:** Apache 2.0
- **Repository:** `https://github.com/semidark/kokoro-deutsch`

## Build & Install

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/semidark/kokoro-deutsch
cd kokoro-deutsch

# Install Python dependencies (use uv, not pip directly)
uv sync

# Install the package in editable mode
uv pip install -e .

# System dependency required for German G2P
# macOS: brew install espeak-ng
# Linux: apt-get install espeak-ng
```

## Running Tests

No repository-level test suite is currently maintained in this repo.

To validate training/inference changes, use:

- Smoke training runs in `StyleTTS2/`
- `scripts/test_inference.py` for checkpoint conversion + inference checks

## CLI Usage

```bash
# Generate speech from text
uv run kokoro --text "Hallo Welt" -o output.wav -l d --voice dm_daniel

# From file
uv run kokoro -i input.txt -o output.wav -l d --voice dm_daniel
```

## Code Style Guidelines

### Python

No linter or formatter is configured. Follow the existing conventions observed in the codebase:

#### Imports

- **Order:** Relative imports first, then third-party, then stdlib — but the codebase is inconsistent. Prefer: relative imports, third-party (alphabetical), stdlib (alphabetical).
- **Style:** Use `from x import Y` for specific items; use `import x` for modules used with qualification (e.g., `import torch`, `import re`).
- Common aliases: `import torch.nn as nn`, `import torch.nn.functional as F`, `import numpy as np`.
- Conditional imports are used for optional dependencies (e.g., `misaki[ja]`, `misaki[zh]`) — wrap in try/except with `logger.error()` before re-raising.
- Use `TYPE_CHECKING` guard for imports only needed for type hints (see `__main__.py`).

#### Formatting

- **Quotes:** Single quotes for strings (e.g., `'ab'`, `'cpu'`). Double quotes acceptable in docstrings.
- **Line length:** No hard limit configured; lines up to ~150 chars exist. Keep reasonable.
- **Indentation:** 4 spaces (standard Python).
- **Trailing commas:** Used in multiline constructs (function args, lists).
- **Blank lines:** Two blank lines between top-level classes/functions, one blank line between methods.

#### Types & Type Hints

- Use `typing` module imports: `Optional`, `Union`, `List`, `Tuple`, `Generator`, `Callable`.
- Modern Python syntax (`tuple[X, Y]` lowercase) appears in `model.py` — both styles coexist.
- Annotate public method signatures. Internal/private methods may omit hints.
- Use `@dataclass` for simple data containers (see `KPipeline.Result`, `KModel.Output`).

#### Naming Conventions

- **Classes:** PascalCase with `K` prefix for public API (`KModel`, `KPipeline`). Neural network modules use standard PyTorch naming (`CustomAlbert`, `ProsodyPredictor`, `TextEncoder`).
- **Functions/methods:** snake_case (`load_voice`, `forward_with_tokens`).
- **Constants:** UPPER_SNAKE_CASE (`ALIASES`, `LANG_CODES`, `MODEL_NAMES`, `MAGIC_DIVISOR`).
- **Private methods:** Prefixed with underscore (`_f02uv`, `_shortcut`, `_residual`, `_build_weights`).
- **Variables:** Short names acceptable in math-heavy code (`x`, `s`, `d`, `F0`, `N`). Use descriptive names elsewhere.
- **Files:** snake_case (`custom_stft.py`, `istftnet.py`).

#### Error Handling

- Use `assert` for internal invariants and input validation (e.g., `assert lang_code in LANG_CODES`).
- Raise `ValueError` for invalid user inputs with descriptive messages.
- Raise `RuntimeError` for environment issues (CUDA/MPS unavailability).
- Use `try/except` with `logger.error()` for optional dependency failures, then re-raise.
- Bare `except:` exists in `model.py:73` for state_dict loading fallback — avoid this pattern in new code; catch specific exceptions.

#### Logging

- Use `loguru` logger (imported from `loguru import logger`), NOT `print()` for diagnostics.
- `print()` is acceptable in `scripts/` CLI tools for direct user output.
- Log levels: `logger.debug()` for tracing, `logger.warning()` for non-fatal issues, `logger.error()` for failures.
- Logger is disabled by default in `__init__.py` (`logger.disable("kokoro")`).

#### Classes & Architecture

- **KModel** (`model.py`): `torch.nn.Module` subclass. Language-blind. Handles weight loading and forward pass. Use `@torch.no_grad()` for inference methods.
- **KPipeline** (`pipeline.py`): Language-aware orchestrator. Handles G2P, voice loading, chunking, and inference. Designed as one instance per language; reuse `KModel` across pipelines.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [semidark/kokoro-deutsch](https://github.com/semidark/kokoro-deutsch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
