---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**mlxterp** is a mechanistic interpretability library for Apple Silicon, leveraging the MLX framework. The goal is to provide researchers with tools similar to nnsight and nnterp but optimized for Apple Silicon Macs.

**Core Philosophy:**
- Simplicity first - make mechanistic interpretability accessible to many researchers
- Lean codebase with minimal boilerplate
- Clean architecture and good maintainability
- Functional-first API leveraging MLX's strengths

## Architecture Design

The library follows a **functional-first paradigm** that aligns with MLX's design:

### Core Components

1. **core/** - Computation graph tracing, intervention system, activation caching, hook registration
   - Generic model wrapping that works with ANY MLX model (not model-specific implementations)
   - Automatic layer detection and naming
2. **analysis/** - Attention patterns, activation analysis, circuit discovery, attribution methods
3. **interpretability/** - Logit lens, patching utilities, probing tools, sparse autoencoder support
4. **visualization/** - Attention/activation visualization and interactive dashboard
5. **utils/** - Data utilities, metrics, Metal-specific optimizations

### Key Design Decisions

**1. Context Manager Pattern for Tracing (nnterp style)**
- Use context managers for clean, intuitive tracing: `with model.trace("text"):`
- Direct attribute access to layer outputs: `model.layers[3].attn.output`
- `.save()` method to capture values for later use

**2. Direct Attribute Access with Proxy Objects**
- Access layers and modules naturally: `model.layers[i].attn.output`
- No need for verbose method calls like `.get_activation("layer_3.attn")`
- Proxy objects handle the wrapping transparently

**3. Lazy Evaluation with Explicit Materialization**
- Leverage MLX's lazy evaluation for efficiency
- Use `.save()` to capture values when needed

**5. Unified Memory Exploitation**
- Take advantage of MLX's unified memory model for zero-copy operations
- Design for efficient caching without device transfers

**6. Generic Model Wrapping (nnsight principle)**
- Works with ANY MLX model, not specific model implementations
- Automatically discovers and wraps model layers at runtime
- No need for model-specific subclasses (unlike TransformerLens's HookedTransformer approach)

## Development Guidelines

### Repository Cleanliness (CRITICAL)

**ALWAYS keep the repository clean and professional:**

**DO:**
- Place all examples in `examples/` directory
- Place all tests in `tests/` directory
- Place all documentation in `docs/` directory
- Use temporary variables in code instead of creating debug files
- Clean up any temporary files immediately after debugging
- Add generated files (`.mlx/`, `wandb/`, etc.) to `.gitignore`

**DON'T:**
- Create debug scripts in root directory (e.g., `debug_*.py`, `test_*.py`, `check_*.py`)
- Create temporary documentation files in root (e.g., `*_SUMMARY.md`, `*_FIX.md`, `*_GUIDE.md`)
- Leave test artifacts (`.mlx` model files, cache directories)
- Scatter analysis scripts across the repository
- Create multiple redundant documentation files

**Root directory should ONLY contain:**
- `CLAUDE.md` - This file (development guidelines)
- `README.md` - Main documentation
- `SAE_ROADMAP.md` - SAE development phases
- `TROUBLESHOOTING.md` - User troubleshooting guide
- Configuration files (`pyproject.toml`, `mkdocs.yml`, etc.)
- Essential project files (`.gitignore`, `LICENSE`, etc.)

**For debugging:**
- Use Python debugger or print statements
- Create temporary functions, not files
- If you must create a debug script, delete it immediately after use
- Use notebooks for interactive exploration (but clean them up)

**For documentation:**
- Add to `docs/` directory and update `mkdocs.yml`
- Don't create summary files for internal communication
- Consolidate related information instead of creating many small files

### When Creating New Components

**DO:**
- Use functional APIs that return new values
- Leverage MLX's lazy evaluation by default
- Add type hints to all function signatures
- Design for Metal optimization opportunities
- Keep the API simple and composable
- Keep the repository clean and organized

**DON'T:**
- Create model-specific implementations (e.g., HookedLlama, HookedMistral) - use generic wrapping
- Add stateful hooks without strong justification
- Create unnecessary abstractions or boilerplate
- Duplicate functionality that exists in MLX
- Add features that compromise the "simple and accessible" goal
- Scatter debug files and temporary documentation across the repo

### Code Style Preferences

- **Direct attribute access** pattern inspired by nnterp (not verbose method calls)
- Context managers for tracing (`with model.trace():`)
- Proxy objects for intuitive layer access (`model.layers[i].attn.output`)
- Simple utility functions over complex class hierarchies
- Use dataclasses for configuration when needed

### MLX-Specific Considerations

- **Unified Memory**: Arrays live in shared memory - no CPU/GPU transfers needed
- **Lazy Evaluation**: Computations aren't executed until `mx.eval()` is called

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coairesearch/mlxterp](https://github.com/coairesearch/mlxterp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
