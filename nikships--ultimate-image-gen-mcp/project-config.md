---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Gemini 3 Pro Image MCP Server** - A production-ready FastMCP server for Google's Gemini 3 Pro Image, featuring state-of-the-art image generation with advanced reasoning, high-resolution output (1K-4K), reference images (up to 14), Google Search grounding, and thinking mode.

**Uses Official Google GenAI SDK** - Built on `google-genai` package for clean, maintainable code with automatic handling of thought signatures and response parsing.

## Development Commands

### Setup and Installation
```bash
# Install dependencies (required before development)
uv sync --all-extras

# Install from source
uv sync

# Run locally with dev mode (hot-reload enabled)
fastmcp dev src.server:create_app

# Or run directly
python -m src.server
```

### Testing and Quality
```bash
# Run all tests
pytest

# Run with coverage report
pytest --cov=src --cov-report=html

# Type checking
mypy src/

# Linting (check only)
ruff check src/

# Auto-format code
ruff format src/

# Run all quality checks
ruff check src/ && ruff format src/ && mypy src/ && pytest
```

### Environment Setup
```bash
# Required: Set API key
export GEMINI_API_KEY=your_key_here

# Optional: Enable debug logging
export LOG_LEVEL=DEBUG

# Optional: Change output directory
export OUTPUT_DIR=/path/to/output

# Optional: Default image size
export DEFAULT_IMAGE_SIZE=4K

# Optional: Enable Google Search by default
export ENABLE_GOOGLE_SEARCH=true
```

## Architecture

### Core Design Pattern: Gemini 3 Pro Image with Official SDK

The server uses the **official Google GenAI SDK** for clean, type-safe image generation:

```python
from google import genai
from google.genai import types

# Initialize client
client = genai.Client(api_key=api_key)

# Generate with reference images
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[image1, image2, prompt],
    config=types.GenerateContentConfig(
        response_modalities=["TEXT", "IMAGE"],
        image_config=types.ImageConfig(
            aspect_ratio="16:9",
            image_size="4K"
        ),
        tools=[{"google_search": {}}]
    )
)

# Extract images using SDK methods
image = response.parts[0].as_image()
```

**Key insight:** The SDK handles thought signatures automatically, provides built-in image conversion methods, and manages API authentication/errors cleanly.

### Module Responsibilities

**`config/`** - Settings and constants
- `constants.py`: Model names, limits, aspect ratios (single source of truth)
- `settings.py`: Pydantic settings with environment variable binding

**`core/`** - Framework-agnostic utilities
- `exceptions.py`: Custom exception hierarchy for error categorization
- `validation.py`: Input validation functions (called before API requests)

**`services/`** - Business logic layer
- `gemini_client.py`: Gemini API using official `google-genai` SDK
- `prompt_enhancer.py`: Uses Gemini Flash to enhance prompts
- `image_service.py`: High-level orchestrator for image generation

**`tools/`** - MCP tool definitions
- `generate_image.py`: Main tool, handles all Gemini 3 parameters
- `batch_generate.py`: Parallel processing wrapper

**`server.py`** - FastMCP initialization
- Creates app via `create_app()` factory function
- Registers tools and resources
- Entry point for both `python -m src.server` and `uvx`

### Data Flow for Image Generation

1. **MCP Tool** (`generate_image`) receives user request with parameters
2. **Validation** (`core/validation.py`) checks all inputs
3. **(Optional) PromptEnhancer** improves prompt using Gemini Flash
4. **ImageService** orchestrates generation flow
5. **GeminiClient** calls Google API using official SDK
   - Converts reference images from base64 to PIL Images
   - Builds typed config objects (`GenerateContentConfig`, `ImageConfig`)
   - Handles async execution via `run_in_executor` (SDK is sync)
6. **Response Processing** extracts images, text, and thoughts
   - Uses SDK's `.as_image()` method for automatic PIL conversion
   - Separates thought parts from final output
   - Extracts grounding metadata if Google Search was used
7. **ImageResult** objects created with metadata
8. Images saved to disk with descriptive filenames
9. JSON response returned to MCP client

### Gemini 3 Pro Image Features

**Resolution Options:**
- 1K: Fast generation
- 2K (default): High quality for professional use
- 4K: Maximum resolution for production assets

**Reference Images (up to 14 total):**
- Up to 6 object images for high-fidelity inclusion
- Up to 5 human images for character consistency
- Passed as PIL Image objects to SDK

**Google Search Grounding:**
```python
config_args["tools"] = [{"google_search": {}}]
```
- Real-time weather, stocks, events, maps
- Returns `grounding_metadata` with search sources

**Thinking Mode:**
- Enabled by default for Gemini 3 Pro Image
- Model generates interim thought images to refine composition
- Thoughts extracted via `part.thought` attribute
- Final image is highest quality after reasoning

**Response Modalities:**
- `["TEXT", "IMAGE"]` (default): Get both explanation and image
- `["IMAGE"]`: Image only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikships/ultimate-image-gen-mcp](https://github.com/nikships/ultimate-image-gen-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
