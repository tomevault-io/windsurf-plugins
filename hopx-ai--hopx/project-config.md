---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Workflow Requirements

**CRITICAL: Always Test Changes and Provide Evidence**

When making any code changes:
1. **Write comprehensive tests** that verify the fix/feature works correctly
2. **Run the tests** and capture the full output
3. **Provide evidence** in the form of test output showing success
4. **Update documentation**:
   - README.md (customer-facing, no fluff, clear and concise)
   - CLAUDE.md (internal, detailed technical implementation notes)
   - CHANGELOG.md (version history with clear descriptions)
   - Version bumps in `pyproject.toml` and `hopx_ai/__init__.py`

**Documentation Standards**:
- README.md: Enterprise-quality, customer-focused, easy to follow, no unnecessary words
- CLAUDE.md: Complete technical details, implementation notes, internal behaviors
- Test evidence must be included for all significant changes

**Writing Guidelines for All Documentation**:

Apply these principles to all writing (documentation, comments, commit messages, changelogs):

1. **Conciseness**: Use clear, direct sentences. Remove unnecessary words.
2. **Clarity**: Write for a wide audience. Explain technical terms when needed.
3. **Objectivity**: Maintain neutral tone. Avoid subjective adjectives and adverbs.
4. **Customer Focus**: Explain why information matters. Show the benefit.
5. **No Buzzwords**: Avoid marketing language and vague terms that obscure meaning.
6. **Simplicity**: Use simple words. Avoid jargon when plain language works.
7. **Readability**: Use short sentences. Avoid complex sentence structures.
8. **Action-Oriented**: Use subject-verb-object structure. Make doers and actions clear.
9. **No Clutter**: Remove words that don't contribute to the main point.
10. **Professional Tone**: Be warm and human while maintaining professionalism.

**Examples**:
- Bad: "This amazing fix is super fast and works perfectly!"
- Good: "The fix reduces wait time from 15 seconds to 6 seconds."

- Bad: "We leverage cutting-edge technology to provide world-class solutions."
- Good: "The SDK polls template status every 3 seconds."

- Bad: "You'll love how easy it is to use our incredible API!"
- Good: "The API requires two parameters: template name and API key."

**Code Example Testing Policy**:

All code snippets in customer-facing documentation (README.md) must be:
1. **Tested**: Run the exact snippet to verify it works
2. **Complete**: Include all necessary imports and setup
3. **Executable**: Users can copy-paste and run immediately
4. **Stored**: Save tested examples in `examples/` directory
5. **Referenced**: Link to the example file from documentation

When adding code examples:
- Create a standalone file in `examples/` directory
- Run the example and capture output
- Only add the example to documentation after successful test
- Include the file path reference for users to find complete code

Example reference format:
```
See `examples/preview_url_basic.py` for a complete working example.
```

## Overview

This is the **Hopx Python SDK** - the official Python client for Hopx.ai's cloud sandbox service. Hopx provides secure, isolated cloud sandboxes (lightweight VMs) that spin up in seconds for AI agents, code execution, testing, and data processing.

Version: 0.3.0
Python Support: 3.8+
License: MIT

## Development Commands

### Setup
```bash
# Install package in development mode with dependencies
pip install -e .

# Install with dev dependencies (pytest, black, ruff, mypy)
pip install -e ".[dev]"
```

### Testing
```bash
# Run OpenAPI compliance test suite
export HOPX_API_KEY="your_api_key_here"
python test_openapi_compliance.py

# Run template building test (comprehensive end-to-end test)
export HOPX_API_KEY="your_api_key_here"
python examples/test_template_building.py

# Note: There are no pytest tests in this repo currently
# Testing is done via test scripts and example scripts
```

### Code Quality
```bash
# Format code with black (line-length: 100)
black hopx_ai/

# Lint with ruff (line-length: 100, target: py38)
ruff check hopx_ai/

# Type check with mypy (strict mode, target: py38)
mypy hopx_ai/
```

### Running Examples
```bash
# Set API key first
export HOPX_API_KEY="your_api_key"

# Run any example
python examples/quick_start.py
python examples/async_quick_start.py
python examples/template_build.py
```

## OpenAPI Specification Compliance

**Current Status**: 100% compliant with OpenAPI spec version 2025-10-21

The SDK fully implements all Public API endpoints as defined in the OpenAPI specification:

### Implemented Endpoints (16/16)

**System (1/1)**:
- `GET /health` - Health check (no auth required)

**Sandboxes (8/8)**:
- `GET /v1/sandboxes` - List sandboxes with filters
- `POST /v1/sandboxes` - Create sandbox
- `GET /v1/sandboxes/{id}` - Get sandbox details
- `DELETE /v1/sandboxes/{id}` - Delete sandbox
- `POST /v1/sandboxes/{id}/pause` - Pause sandbox
- `POST /v1/sandboxes/{id}/resume` - Resume paused sandbox
- `PUT /v1/sandboxes/{id}/timeout` - Set auto-kill timeout
- `POST /v1/sandboxes/{id}/token/refresh` - Refresh JWT token

**Templates (7/7)**:
- `GET /v1/templates` - List all templates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hopx-ai/hopx](https://github.com/hopx-ai/hopx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
