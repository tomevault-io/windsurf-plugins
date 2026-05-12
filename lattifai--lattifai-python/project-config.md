---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LattifAI is a Python SDK for precision audio-text forced alignment powered by the Lattice-1 model. The project provides:
- **Forced alignment**: Word-level and segment-level audio-text synchronization
- **Multi-model transcription**: Gemini (100+ languages), Parakeet (24 languages), SenseVoice (5 languages)
- **Speaker diarization**: Multi-speaker identification with label preservation
- **Universal format support**: 30+ caption/subtitle formats
- **Multiple interfaces**: CLI and Python SDK

## Essential Commands

### Development Setup

```bash
# Using uv (recommended - 10-100x faster)
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync
source .venv/bin/activate

# Or using pip
pip install -e ".[dev]"

# Install pre-commit hooks
pre-commit install
```

### Testing

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src

# Run specific test file
pytest tests/test_basic.py

# Run specific test
pytest tests/caption/test_formats.py::test_format_name
```

### Code Quality

```bash
# Run all pre-commit hooks
pre-commit run --all-files

# Format code with black (line length: 120)
black src/ tests/ --line-length=120

# Sort imports with isort
isort src/ tests/ --profile=black --line-length=120

# Lint with flake8
flake8 src/ tests/ --config=.flake8 --ignore=E203
```

### CLI Commands

The project uses `nemo_run` for CLI entry points. All CLI commands are in `src/lattifai/cli/`:

```bash
# Alignment
lai-align <audio> <caption> <output>

# Transcription
lai-transcribe <input> <output>

# Caption utilities
laicap-convert <input> <output>
laicap-shift <input> <output> <offset_seconds>

# Diarization
lai-diarize <audio> [--num-speakers N]

# Translation
lai-translate <input> <output> --target-lang <lang>

# YouTube
lai-youtube <url>

# Server
lai-serve --port 8001
```

## Architecture

### Core Design Pattern: Config-Driven

LattifAI uses a **config-driven architecture** with `nemo_run` for maximum flexibility and reproducibility:

```
Configuration Layer (nemo_run Config objects)
├── ClientConfig       # API client settings
├── AlignmentConfig    # Lattice-1 model & device
├── CaptionConfig      # I/O formats & processing
├── TranscriptionConfig # ASR model selection
├── DiarizationConfig  # Speaker detection
└── MediaConfig        # Audio/video loading
```

### Module Structure

```
src/lattifai/
├── client.py              # Main LattifAI client (config-driven)
├── mixin.py               # Mixins for transcribe/save workflows
├── audio2.py              # Audio loading and resampling
├── errors.py              # Custom exception hierarchy
├── utils.py               # Shared utilities
├── languages.py           # Language code definitions
├── config/                # Configuration classes (all inherit from nemo_run.Config)
│   ├── client.py          # API settings
│   ├── alignment.py       # Model & device config
│   ├── caption.py         # I/O format config
│   ├── transcription.py   # ASR model config
│   ├── diarization.py     # Speaker detection config
│   ├── translation.py     # Translation config
│   ├── event.py           # Event tracking config
│   └── media.py           # Audio loading config
├── alignment/             # Forced alignment engine
│   ├── lattice1_aligner.py    # Main aligner using Lattice-1 model
│   ├── lattice1_worker.py     # Low-level alignment worker
│   ├── tokenizer.py           # Text preprocessing & normalization
│   ├── sentence_splitter.py   # Smart sentence splitting (wtpsplit)
│   ├── segmenter.py           # Audio segmentation
│   ├── text_align.py          # Supervision-transcription alignment & duplicate detection
│   ├── phonemizer.py          # G2P phoneme conversion
│   └── punctuation.py         # Punctuation handling
├── caption/               # Caption I/O and data structures
│   ├── caption.py         # Caption dataclass (container for all results)
│   ├── supervision.py     # Supervision segment dataclass
│   ├── formats/           # Format handlers (30+ formats)
│   │   ├── base.py        # Base reader/writer classes
│   │   ├── pysubs2.py     # SRT, VTT, ASS, SSA via pysubs2
│   │   ├── gemini.py      # Gemini API format
│   │   ├── textgrid.py    # Praat TextGrid
│   │   ├── tabular.py     # TSV, CSV, AUD
│   │   └── nle/           # NLE formats (Premiere, FCPXML, etc.)
│   └── parsers/           # Text parsing utilities
├── transcription/         # Multi-model transcription
│   ├── base.py            # BaseTranscriber interface
│   ├── gemini.py          # Gemini API (100+ languages)
│   ├── lattifai.py        # Local models (Parakeet, SenseVoice)
│   ├── vllm.py            # vLLM/SGLang inference backend
│   └── __init__.py        # create_transcriber() factory
├── translation/           # Caption translation pipeline
│   ├── base.py            # BaseTranslator interface
│   ├── analyzer.py        # Language/terminology analysis
│   ├── glossary.py        # Terminology management
│   ├── reviewer.py        # Translation review
│   └── prompts.py         # Translation prompts
├── diarization/           # Speaker diarization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lattifai/lattifai-python](https://github.com/lattifai/lattifai-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
