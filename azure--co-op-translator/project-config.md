---
trigger: always_on
description: Co‑op Translator is a Python command‑line tool and GitHub Actions workflow that translates Markdown files, Jupyter notebooks, and image text into multiple languages. It organizes outputs under language‑specific folders and keeps translations synchronized with source content. The project is structured as a Poetry‑managed library with CLI entry points.
---

# AGENTS.md

## Project Overview

Co‑op Translator is a Python command‑line tool and GitHub Actions workflow that translates Markdown files, Jupyter notebooks, and image text into multiple languages. It organizes outputs under language‑specific folders and keeps translations synchronized with source content. The project is structured as a Poetry‑managed library with CLI entry points.

### Architecture overview

- CLI entry points (`translate`, `migrate-links`, `evaluate`) invoke a unified CLI that dispatches to translation, link migration, and evaluation flows.
- Configuration loader reads `.env` and auto‑detects the LLM provider (Azure OpenAI or OpenAI) and, if requested, the vision provider (Azure AI Service) for image text extraction.
- Translation core handles Markdown and notebooks; the vision pipeline extracts text from images when `-img` is used.
- Outputs are organized into `translations/<lang>/` for text and `translated_images/` for images, preserving original structure.

### Key technologies and frameworks

- Python 3.10–3.12, Poetry for packaging
- CLI: `click`
- LLM/AI SDKs: Azure OpenAI, OpenAI
- Vision: Azure AI Service (Computer Vision)
- HTTP and data: `httpx`, `pydantic`
- Imaging: `pillow`, `opencv-python`, `matplotlib`
- Tooling: `pytest`, `black`, `ruff`

## Setup Commands

### Prerequisites

- Python 3.10–3.12
- Azure subscription (optional, for Azure AI services)
- Internet access for LLM/Vision APIs (e.g., Azure OpenAI/OpenAI, Azure AI Vision)

### Option A: Poetry (recommended)

```bash
# From repository root
pip install poetry
poetry install

# Run any command via Poetry
poetry run translate --help
```

### Option B: pip/venv

```bash
# Create & activate virtual environment
python -m venv .venv
# Windows
.venv\\Scripts\\activate
# Linux/macOS
# source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# (Optional) Editable install for local development
pip install -e .
```

## End User Usage

### Docker (published image)

```bash
# Pull public image from GHCR
docker pull ghcr.io/azure/co-op-translator:latest

# Run with current folder mounted and .env provided (Bash/Zsh)
docker run --rm -it --env-file .env -v "${PWD}:/work" ghcr.io/azure/co-op-translator:latest -l "fr es" -md

# PowerShell
docker run --rm -it --env-file .env -v ${PWD}:/work ghcr.io/azure/co-op-translator:latest -l "fr es" -md
```

Notes:
- Default entrypoint is `translate`. Override with `--entrypoint migrate-links` for link migration.
- Ensure GHCR package visibility is Public for anonymous pulls.

### CLI (pip)

```bash
pip install co-op-translator
translate -l "fr es" -md
```

### Environment configuration

Create a `.env` file at the repository root and provide credentials/endpoints for your chosen language model and (optionally) vision service. For provider‑specific setup, see `getting_started/set-up-azure-ai.md`.

### Required Environment Variables

At least one LLM provider must be configured. For image translation, Azure AI Service must also be configured.

- Azure OpenAI (text translation):
  - `AZURE_OPENAI_API_KEY`
  - `AZURE_OPENAI_ENDPOINT`
  - `AZURE_OPENAI_MODEL_NAME`
  - `AZURE_OPENAI_CHAT_DEPLOYMENT_NAME`
  - `AZURE_OPENAI_API_VERSION`

- OpenAI (text translation alternative):
  - `OPENAI_API_KEY`
  - `OPENAI_ORG_ID` (optional)
  - `OPENAI_CHAT_MODEL_ID` (required when using OpenAI provider)
  - `OPENAI_BASE_URL` (optional; defaults to `https://api.openai.com/v1`)

- Azure AI Service for image text extraction (required when using `-img`):
  - `AZURE_AI_SERVICE_API_KEY` (preferred) or legacy `AZURE_SUBSCRIPTION_KEY`
  - `AZURE_AI_SERVICE_ENDPOINT`

Example `.env` snippet:

```bash
# Azure AI Service (for image translation)
AZURE_AI_SERVICE_API_KEY="..."
AZURE_AI_SERVICE_ENDPOINT="https://<your-ai-service>.cognitiveservices.azure.com/"

# Azure OpenAI (primary option)
AZURE_OPENAI_API_KEY="..."
AZURE_OPENAI_ENDPOINT="https://<your-azure-openai>.openai.azure.com/"
AZURE_OPENAI_MODEL_NAME="gpt-4o"
AZURE_OPENAI_CHAT_DEPLOYMENT_NAME="<your-deployment>"
AZURE_OPENAI_API_VERSION="2024-12-01-preview"

# OpenAI (alternative option)
OPENAI_API_KEY="..."
OPENAI_ORG_ID="..."            # optional
OPENAI_CHAT_MODEL_ID="gpt-4o"   # required when using OpenAI
OPENAI_BASE_URL="https://api.openai.com/v1" # optional
```

Notes:

- The tool auto-detects the available LLM provider; configure either Azure OpenAI or OpenAI.
- Image translation requires both `AZURE_AI_SERVICE_API_KEY` and `AZURE_AI_SERVICE_ENDPOINT`.
- The CLI will raise a clear error if required variables are missing.

## Development Workflow

- Source code lives under `src/co_op_translator`; tests under `tests/`.
- Primary CLIs (installed via entry points):

```bash
# Translate content to one or more languages (space‑separated codes)
translate -l "fr es de"

# Restrict by content type
translate -l "ja" -md            # only Markdown
translate -l "ko" -nb            # only notebooks
translate -l "zh" -md -img       # Markdown + images


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/co-op-translator](https://github.com/Azure/co-op-translator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
