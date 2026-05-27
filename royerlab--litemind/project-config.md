---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Makefile Commands

The project includes a Makefile with common development tasks. Run `make help` to see all available commands:

```bash
make setup         # Install project with all development dependencies
make install       # Install project with minimal dependencies
make test          # Run all tests
make test-cov      # Run tests with coverage report
make system-deps   # Install external system dependencies (ffmpeg, etc.)
make check         # Run all code checks (format + lint + typecheck)
make format        # Format code with black and isort
make lint          # Run flake8 linter
make typecheck     # Run mypy type checker
make build         # Build package
make clean         # Clean build artifacts
make publish       # Bump version, commit, tag, and push (triggers PyPI release)
make publish-patch # Publish patch version (same day increment)
```

## Build & Development Commands

```bash
# Install for development (with all optional dependencies)
pip install -e ".[dev,rag,whisper,documents,tables,videos,audio,remote,tasks]"

# Format code
isort .
black .

# Lint and type check
flake8 .
mypy src/litemind

# Run all tests
pytest src/

# Run a single test file
pytest src/litemind/apis/tests/test_openai_api.py

# Run tests matching a pattern
pytest -k "tools" src/

# Run tests with coverage report
pytest --cov=src --cov-report=html:reports/coverage src/

# Build package
hatch clean && hatch build
```

## Publishing to PyPI

Publishing is handled via GitHub Actions triggered by git tags. Use the Makefile commands:

```bash
# For a new day's release (bumps version to YYYY.M.D format)
make publish

# For same-day patch releases (bumps to YYYY.M.D.N format)
make publish-patch
```

These commands will:
1. Update the version in `src/litemind/__init__.py`
2. Commit the version bump
3. Create a git tag (e.g., `v2026.2.4`)
4. Push to origin with tags
5. GitHub Actions then automatically publishes to PyPI via OIDC trusted publishing

## CLI Tools

```bash
# Export repository to single file
litemind export --folder-path . --output-file exported.txt

# Validate model registry against live APIs
litemind validate --api openai gemini --models gpt-4o models/gemini-1.5-pro

# Discover features for unknown models
litemind discover --api openai --models new-model-name
```

## Architecture Overview

Litemind is a unified API wrapper and agentic AI framework supporting multiple LLM providers (OpenAI, Anthropic, Google Gemini, Ollama).

### Core Layers

**API Layer** (`src/litemind/apis/`):
- `BaseApi` → `DefaultApi` → Provider implementations (OpenAIApi, AnthropicApi, GeminiApi, OllamaApi)
- `CombinedApi`: Chains multiple APIs for fallback
- Feature-based model selection via `ModelFeatures` enum and `get_best_model(features=[...])`
- Provider-specific code lives in `providers/<provider>/` with standard methods: `convert_messages`, `format_tools`, `process_response`

**Agent Layer** (`src/litemind/agent/`):
- `Agent`: Main orchestrator managing conversation state, tool execution, and augmentations
- `Conversation`: Manages system and user messages
- `Message`/`MessageBlock`: Multimodal message containers
- `ToolSet`: Collection of tools (FunctionTool, AgentTool, BuiltinTool)
- `AugmentationSet`: RAG augmentations including vector databases

**Media Layer** (`src/litemind/media/`):
- `MediaBase` subclasses: `Text`, `Code`, `Json`, `Image`, `Audio`, `Video`, `Table`, `Document`, `NdImage`
- `MediaConverter`: Automatic format conversion pipeline for model compatibility
- Conversion utilities in `media/conversion/`

**Augmentations** (`src/litemind/agent/augmentations/`):
- `BaseVectorDB` → `InMemoryVectorDatabase`, `QdrantVectorDatabase`
- `Information`: Knowledge units with metadata for RAG

### Key Patterns

- Abstract base classes define interfaces (`BaseApi`, `BaseTool`, `AugmentationBase`, `MediaBase`)
- Callback system throughout (`ApiCallbackManager`, `ToolCallbackManager`)
- Feature matrices describe model capabilities for intelligent selection
- Tests mirror source structure (e.g., `apis/tests/`, `agent/tests/`, `media/tests/`)

### Model Registry Architecture

Model feature data is curated in YAML files (`src/litemind/apis/model_registry/*.yaml`) rather than auto-discovered via live API calls. This design decision was made because:
- Auto-discovery (the old scanner approach) produced false negatives and brittle tests
- Curated YAML provides accurate feature data, token limits, aliases, and metadata
- Each provider has a registry file: `registry_openai.yaml`, `registry_anthropic.yaml`, `registry_gemini.yaml`, `registry_ollama.yaml`
- Use `ModelRegistry.supports_feature()`, `get_model_info()`, and `resolve_alias()` for feature lookups
- The `litemind validate` CLI can validate registry accuracy against live APIs when needed

### Data Flow

```
User Message → Agent → Augmentation retrieval → Message formatting →
API call → Tool execution (if needed) → Response → Conversation history
```

## Environment Variables

- `OPENAI_API_KEY` - OpenAI API
- `ANTHROPIC_API_KEY` - Anthropic/Claude API
- `GOOGLE_GEMINI_API_KEY` - Google Gemini API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [royerlab/litemind](https://github.com/royerlab/litemind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
