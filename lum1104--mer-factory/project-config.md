---
trigger: always_on
description: > Where to modify code when adding features to MER-Factory
---

# AGENS.md - Agent Guide for MER-Factory

> Where to modify code when adding features to MER-Factory

## Overview

MER-Factory is a Python toolkit for Multimodal Emotion Recognition and Reasoning (MERR) dataset creation. It uses LangGraph for pipeline orchestration and supports multiple LLM providers (Gemini, ChatGPT, Ollama, HuggingFace).

**Key features**: Automated preprocessing, LLM-based annotation, reference-free evaluation (CLIP/CLAP/AU F1/NLI/ASR), Gate Agent for quality control (dev), and training pipeline integration.

**Tech stack**: Python 3.12+, LangChain/LangGraph, PyTorch, Transformers, Typer CLI, Rich.

## Project Architecture

```
Entry Point (main.py)
    ↓
Config (utils/config.py) → ProcessingType (MER/AU/audio/video/image)
    ↓
Graph (mer_factory/graph.py) → StateGraph with conditional routing
    ↓
Nodes (mer_factory/nodes/) → Pipeline stages (async/sync)
    ↓
Models (mer_factory/models/) → LLM providers (Gemini/ChatGPT/Ollama/HF)
    ↓
Prompts (utils/prompts/) → Template-based LLM instructions
    ↓
Export (export.py) → CSV/ShareGPT/Emotion-LLaMA formats
    ↓
Evaluate (tools/evaluate.py) → Reference-free quality metrics (CLIP, CLAP, AU F1, NLI, ASR WER)
    ↓
Training (train.sh → LLaMA-Factory) → Prepare dataset & launch training UI
```

## Quick Reference

| File | Purpose |
|------|---------|
| `main.py` | CLI entry point (Typer), arg parsing |
| `mer_factory/graph.py` | LangGraph pipeline definition and routing |
| `mer_factory/nodes/async_nodes.py` | Pipeline implementations (default) |
| `mer_factory/nodes/sync_nodes.py` | Sync implementations (HF models only) |
| `mer_factory/models/__init__.py` | LLM provider factory |
| `utils/config.py` | Configuration dataclasses |
| `utils/prompts/prompts.json` | LLM prompt templates |
| `export.py` | Dataset export (CSV/ShareGPT/Emotion-LLaMA) |
| `tools/evaluate.py` | **Reference-free evaluation** (CLIP, CLAP, AU F1, NLI, ASR WER, composite score) |
| `train.sh` | Training prep → LLaMA-Factory Web UI |

**Gate Agent** (dev feature): Enable with `--use-gate-agent` or `-uga`. Reviews intermediate analysis results and rejects low-quality outputs, prompting sub-agents to refine. See `mer_factory/nodes/gate_agent.py`.

## Project Structure

```
MER-Factory/
├── main.py                     # CLI entry point
├── export.py                   # Dataset export (CSV/ShareGPT/Emotion-LLaMA)
├── dashboard.py                # Flask web server for data curation
├── train.sh                    # Training preparation script
├── requirements.txt            # Python dependencies
│
├── mer_factory/                # Core package
│   ├── graph.py               # LangGraph pipeline + routing
│   ├── state.py               # MERRState definition
│   ├── prompts.py             # Prompt template loader
│   ├── tools.py               # Tool functions
│   ├── nodes/                 # Pipeline nodes
│   │   ├── async_nodes.py     # Async implementations (default)
│   │   ├── sync_nodes.py      # Sync for HF models
│   │   └── gate_agent.py      # Gate Agent (quality control, dev feature)
│   └── models/                # LLM provider integrations
│       ├── __init__.py        # LLMModels factory
│       ├── hf_models/         # HuggingFace models
│       ├── hf_api_server.py   # HF model API server
│       └── api_models/        # Gemini, ChatGPT, Ollama
│
├── utils/                      # Utilities
│   ├── config.py              # Configuration dataclasses
│   ├── file_handler.py        # File discovery
│   ├── processing_manager.py  # Pipeline orchestration
│   ├── register_dataset.py    # LLaMA-Factory dataset registration
│   ├── caching.py             # LLM response caching
│   └── prompts/               # Prompt templates
│       └── prompts.json       # Default prompts
│
├── tools/                      # Standalone tools
│   ├── evaluate.py            # Reference-free evaluation
│   ├── ffmpeg_adapter.py      # FFmpeg wrapper
│   ├── openface_adapter.py    # OpenFace wrapper
│   ├── facial_analyzer.py     # Facial analysis
│   └── emotion_analyzer.py    # Emotion analysis
│
├── test/                       # Test scripts
│   ├── test_ffmpeg.py         # FFmpeg integration test
│   └── test_openface.py       # OpenFace integration test
│
├── docs/                       # Jekyll documentation
├── examples/                   # Example outputs
└── LLaMA-Factory/              # Git submodule: training framework
```

## Adding Features: Where to Modify

### 1. Add a New LLM Provider

**Files to modify**:
1. `mer_factory/models/api_models/` or `mer_factory/models/hf_models/` - Implement model class
2. `mer_factory/models/__init__.py` - Register in `LLMModels.__init__()`
3. `main.py` - Add CLI argument (if needed)

**Steps**:
```python
# 1. Create mer_factory/models/api_models/your_provider.py
# Implement: analyze_audio(), describe_video(), describe_image(),
#           synthesize_summary(), describe_facial_expression()

# 2. Edit mer_factory/models/__init__.py:
#    - Import your model class
#    - Add entry to model_factory dict in __init__()

# 3. Edit main.py (if CLI arg needed):
#    @app.command()
#    def process(
#        ...
#        your_provider_model: str = typer.Option(None, "--your-provider", "-yp")
#    )
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lum1104/MER-Factory](https://github.com/Lum1104/MER-Factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
