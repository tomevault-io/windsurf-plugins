---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**granite-switch** implements **Granite Switch**, a system for building and deploying Granite models with embedded LoRA adapters. The system is a single unified Python package (`granite_switch`) with optional extras for different backends.

1. **Building models with embedded adapters** - Combine a base Granite model with multiple LoRA adapters into a single checkpoint
2. **Automatic adapter control** - Activate adapters via special control tokens or chat templates
3. **Fast inference** - Deploy with vLLM for speedup over standard HuggingFace inference
4. **Optional trainable switching** - Train a router to automatically select adapters per-token

## Project Structure

```
granite-switch/
├── pyproject.toml                       # Single package definition with optional extras
├── src/
│   └── granite_switch/                  # Unified package
│       ├── __init__.py                  # Core exports (GraniteSwitchConfig, __version__)
│       ├── config.py                    # Unified GraniteSwitchConfig
│       │
│       ├── composer/                    # Compose system (requires [compose] extra)
│       │   ├── __init__.py
│       │   ├── adapter_discovery.py     # Adapter discovery and resolution
│       │   ├── adapter_loader.py        # Adapter weight loading
│       │   ├── arch.py                  # Architecture definitions
│       │   ├── compose_granite_switch.py  # Main compose script (CLI entry point)
│       │   ├── compose_utils.py           # GraniteSwitchComposer class
│       │   ├── tokenizer_setup.py       # Tokenizer configuration for control tokens
│       │   ├── validator.py             # Compose validation checks
│       │   ├── weight_remapper.py       # Adapter name remapping (AdapterRemapper)
│       │   ├── weight_transfer.py       # Base model weight transfer
│       │   └── reporting/               # Compose reporting utilities
│       │       ├── __init__.py
│       │       ├── adapter_analysis.py
│       │       ├── compose_report.py
│       │       ├── hiding_constant_report.py
│       │       ├── model_card.py
│       │       └── population_table.py
│       │
│       ├── hf/                          # HuggingFace backend (requires [hf] extra)
│       │   ├── __init__.py              # Registers with transformers AutoConfig/AutoModel
│       │   ├── modeling_granite_switch.py
│       │   ├── core/
│       │   │   ├── __init__.py
│       │   │   └── lora.py              # SwitchedLoRALinear, MergedSwitchedLoRALinear
│       │   └── switch/
│       │       ├── __init__.py
│       │       └── single.py            # SingleSwitch (HF attention backends)
│       │
│       └── vllm/                        # vLLM backend (requires [vllm] extra)
│           ├── __init__.py              # register() for vLLM plugin system
│           ├── granite_switch_model.py
│           ├── core/
│           │   ├── __init__.py
│           │   ├── lora.py              # SwitchedLoRALinear (Punica kernels)
│           │   ├── lora_kernel_meta.py
│           │   └── decoder.py           # Decoder layers
│           └── switch/
│               ├── __init__.py
│               └── single.py            # SingleSwitch (vLLM Attention)
│
├── tests/                               # All tests
│   ├── unit/                            # Unit tests (fastest, CPU)
│   ├── hf/                              # HuggingFace-specific tests
│   ├── vllm/                            # vLLM-specific tests
│   ├── composer/                        # Compose system tests
│   ├── integration/                     # Cross-backend integration tests
│   ├── regression/                      # Regression tests (hf/, vllm/, integration/, shared/, tools/)
│   └── shared/                          # Shared test utilities and parametrized cases
│
├── scratch/                             # Throwaway debug/diagnostic scripts (gitignored)
├── docs/                                # Documentation
├── tutorials/                           # Tutorials and how-to guides
├── CLAUDE.md                            # This file
└── README.md
```

## Installation

```bash
# Core package only (config)
pip install -e .

# With HuggingFace backend
pip install -e ".[hf]"

# With vLLM backend
pip install -e ".[vllm]"

# With compose tools
pip install -e ".[compose]"

# Everything (development)
pip install -e ".[dev]"
```

## Import Paths

```python
# Config (shared by all backends)
from granite_switch import GraniteSwitchConfig
from granite_switch.config import GraniteSwitchConfig  # equivalent

# HuggingFace backend
from granite_switch.hf import GraniteSwitchForCausalLM
from granite_switch.hf.core.lora import SwitchedLoRALinear
from granite_switch.hf.switch.single import SingleSwitch

# vLLM backend (auto-registered via plugin entry point)
from granite_switch.vllm import register

# Compose system
from granite_switch.composer import GraniteSwitchComposer
```

## File Organization Convention

**IMPORTANT:** Keep the repository organized by placing files in their designated directories.

### Documentation Files (Markdown)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [generative-computing/granite-switch](https://github.com/generative-computing/granite-switch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
