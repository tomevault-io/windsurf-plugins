---
trigger: always_on
description: **Purpose:** Technical reference for ALICE development
---

# AGENTS.md

**Version:** 2.0  
**Date:** 2026-02-22  
**Purpose:** Technical reference for ALICE development

---

## Project Overview

**ALICE** (Artificial Latent Image Composition Engine) is a remote Stable Diffusion service built for privacy, performance, and simplicity.

- **Language:** Python 3.10+
- **Architecture:** FastAPI web service with PyTorch/Diffusers backend
- **Philosophy:** Privacy-first, local-first AI image generation
- **Part of:** Synthetic Autonomic Mind ecosystem

**Key Technologies:**
- FastAPI 0.104+ for web server
- PyTorch 2.6.0 for model inference
- Diffusers (latest) for Stable Diffusion pipelines
- stable-diffusion.cpp (sd.cpp) for Vulkan-based inference
- Uvicorn for ASGI server
- Pydantic for data validation

**Supported Model Types:**
- **SD 1.5** - Stable Diffusion 1.5 (.safetensors)
- **SDXL** - Stable Diffusion XL (.safetensors or diffusers)
- **FLUX** - Black Forest Labs FLUX (.safetensors or diffusers)
- **Qwen Image Edit** - Qwen2.5-based image editing (.gguf, multi-component)

**Supported Generation Modes:**
- **Text-to-Image** - Generate images from text prompts
- **Image-to-Image** - Edit/transform existing images with text instructions

---

## Quick Setup

```bash
# Clone repository
git clone https://github.com/SyntheticAutonomicMind/ALICE.git
cd ALICE

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install PyTorch (CRITICAL: platform-specific)
# For NVIDIA CUDA:
pip install torch==2.6.0 torchvision==0.21.0 --index-url https://download.pytorch.org/whl/cu124

# For AMD ROCm:
pip install torch==2.6.0 torchvision==0.21.0 --index-url https://download.pytorch.org/whl/rocm6.2

# For macOS (Apple Silicon):
pip install torch==2.6.0 torchvision==0.21.0

# Install dependencies
pip install -r requirements.txt

# Create required directories
mkdir -p models images logs

# Run development server
python -m src.main

# Or use Makefile
make install
make run
```

---

## Architecture

```
FastAPI Application (src/main.py)
    |
    +-- Authentication Layer (src/auth.py)
    |   - API key management
    |   - Session handling
    |   - User roles (admin/user)
    |
    +-- Model Management
    |   +-- ModelRegistry (src/model_registry.py)
    |   |   - Model detection (SD 1.5, SDXL, FLUX)
    |   |   - Model listing and metadata
    |   |
    |   +-- ModelCache (src/model_cache.py)
    |   |   - Memory caching of loaded models
    |   |   - Auto-unload on timeout
    |   |
    |   +-- DownloadManager (src/downloader.py)
    |       - CivitAI integration
    |       - HuggingFace integration
    |       - Async download tasks
    |
    +-- Generation Pipeline
    |   +-- GeneratorService (src/generator.py)
    |   |   - Request coordination
    |   |   - Job queue management
    |   |   - Text-to-image and image-to-image routing
    |   |
    |   +-- JobQueue (src/job_queue.py)
    |   |   - Concurrent request management
    |   |   - Request timeout handling
    |   |
    |   +-- Backend System (src/backends/)
    |       +-- PyTorchBackend (pytorch_backend.py)
    |       |   - GPU-accelerated generation (ROCm/CUDA/MPS)
    |       |   - SD 1.5, SDXL, FLUX, Qwen support
    |       |   - Text-to-image and image-to-image
    |       |   - Memory optimization
    |       |
    |       +-- SDCppBackend (sdcpp_backend.py)
    |           - Vulkan-based generation
    |           - Universal GPU support
    |           - GGUF model format
    |           - Multi-component model configs (YAML)
    |           - Text-to-image and image-to-image
    |
    +-- Gallery Management (src/gallery.py)
    |   - Image metadata storage
    |   - Privacy controls (public/private)
    |   - Batch operations
    |
    +-- Web Interface (web/)
        - Dashboard (index.html)
        - Generation UI (generate.html)
        - Gallery (gallery.html)
        - Model management (download.html)
        - Admin panel (admin.html)
```

---

## Directory Structure

| Path | Purpose |
|------|---------|
| `src/` | Main Python application code |
| `src/backends/` | Generation backend implementations |
| `web/` | Static web UI files |
| `scripts/` | Installation and utility scripts |
| `docker/` | Docker configuration files |
| `docs/` | Technical documentation |
| `tests/` | Unit and integration tests |
| `models/` | Stable Diffusion models (gitignored) |
| `images/` | Generated images (gitignored) |
| `data/` | Database and metadata (gitignored) |
| `logs/` | Application logs (gitignored) |
| `venv/` | Python virtual environment (gitignored) |
| `.clio/` | CLIO agent configuration |
| `.github/workflows/` | GitHub Actions (CI/CD, release, triage) |
| `ai-assisted/` | Session handoff documents (gitignored) |
| `scratch/` | Working documents (gitignored) |

**Key Files:**

- `src/main.py` - FastAPI application entry point
- `src/__init__.py` - **Version single source of truth** (`__version__`)
- `src/updater.py` - Self-update system (GitHub Releases integration)
- `src/config_migration.py` - Config migration for cross-version upgrades
- `src/backends/pytorch_backend.py` - PyTorch generation backend (text2img + img2img)
- `src/backends/sdcpp_backend.py` - sd.cpp Vulkan backend (text2img + img2img, GGUF models)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SyntheticAutonomicMind/ALICE](https://github.com/SyntheticAutonomicMind/ALICE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
