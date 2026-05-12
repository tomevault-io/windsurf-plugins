---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🎯 Project Overview

**Skill Seekers** is a Python tool that converts documentation websites, GitHub repositories, and PDFs into LLM skills. It supports 4 platforms: Claude AI, Google Gemini, OpenAI ChatGPT, and Generic Markdown.

**Current Version:** v2.5.1
**Python Version:** 3.10+ required
**Status:** Production-ready, published on PyPI

## 🏗️ Architecture

### Core Design Pattern: Platform Adaptors

The codebase uses the **Strategy Pattern** with a factory method to support multiple LLM platforms:

```
src/skill_seekers/cli/adaptors/
├── __init__.py          # Factory: get_adaptor(target)
├── base_adaptor.py      # Abstract base class
├── claude_adaptor.py    # Claude AI (ZIP + YAML)
├── gemini_adaptor.py    # Google Gemini (tar.gz)
├── openai_adaptor.py    # OpenAI ChatGPT (ZIP + Vector Store)
└── markdown_adaptor.py  # Generic Markdown (ZIP)
```

**Key Methods:**
- `package(skill_dir, output_path)` - Platform-specific packaging
- `upload(package_path, api_key)` - Platform-specific upload
- `enhance(skill_dir, mode)` - AI enhancement with platform-specific models

### Data Flow (5 Phases)

1. **Scrape Phase** (`doc_scraper.py:scrape_all()`)
   - BFS traversal from base_url
   - Output: `output/{name}_data/pages/*.json`

2. **Build Phase** (`doc_scraper.py:build_skill()`)
   - Load pages → Categorize → Extract patterns
   - Output: `output/{name}/SKILL.md` + `references/*.md`

3. **Enhancement Phase** (optional, `enhance_skill_local.py`)
   - LLM analyzes references → Rewrites SKILL.md
   - Platform-specific models (Sonnet 4, Gemini 2.0, GPT-4o)

4. **Package Phase** (`package_skill.py` → adaptor)
   - Platform adaptor packages in appropriate format
   - Output: `.zip` or `.tar.gz`

5. **Upload Phase** (optional, `upload_skill.py` → adaptor)
   - Upload via platform API

### File Structure (src/ layout)

```
src/skill_seekers/
├── cli/                        # CLI tools
│   ├── main.py                 # Git-style CLI dispatcher
│   ├── doc_scraper.py          # Main scraper (~790 lines)
│   ├── github_scraper.py       # GitHub repo analysis
│   ├── pdf_scraper.py          # PDF extraction
│   ├── unified_scraper.py      # Multi-source scraping
│   ├── enhance_skill_local.py  # AI enhancement (local)
│   ├── package_skill.py        # Skill packager
│   ├── upload_skill.py         # Upload to platforms
│   ├── install_skill.py        # Complete workflow automation
│   ├── install_agent.py        # Install to AI agent directories
│   └── adaptors/               # Platform adaptor architecture
│       ├── __init__.py
│       ├── base_adaptor.py
│       ├── claude_adaptor.py
│       ├── gemini_adaptor.py
│       ├── openai_adaptor.py
│       └── markdown_adaptor.py
└── mcp/                        # MCP server integration
    ├── server.py               # FastMCP server (stdio + HTTP)
    └── tools/                  # 18 MCP tool implementations
```

## 🛠️ Development Commands

### Setup

```bash
# Install in editable mode (required before tests due to src/ layout)
pip install -e .

# Install with all platform dependencies
pip install -e ".[all-llms]"

# Install specific platforms
pip install -e ".[gemini]"   # Google Gemini
pip install -e ".[openai]"   # OpenAI ChatGPT
```

### Running Tests

**CRITICAL: Never skip tests** - User requires all tests to pass before commits.

```bash
# All tests (must run pip install -e . first!)
pytest tests/ -v

# Specific test file
pytest tests/test_scraper_features.py -v

# Multi-platform tests
pytest tests/test_install_multiplatform.py -v

# With coverage
pytest tests/ --cov=src/skill_seekers --cov-report=term --cov-report=html

# Single test
pytest tests/test_scraper_features.py::test_detect_language -v

# MCP server tests
pytest tests/test_mcp_fastmcp.py -v
```

**Test Architecture:**
- 46 test files covering all features
- CI Matrix: Ubuntu + macOS, Python 3.10-3.13
- 700+ tests passing
- Must run `pip install -e .` before tests (src/ layout requirement)

### Building & Publishing

```bash
# Build package (using uv - recommended)
uv build

# Or using build
python -m build

# Publish to PyPI
uv publish

# Or using twine
python -m twine upload dist/*
```

### Testing CLI Commands

```bash
# Test scraping (dry run)
skill-seekers scrape --config configs/react.json --dry-run

# Test multi-platform packaging
skill-seekers package output/react/ --target gemini --dry-run

# Test MCP server (stdio mode)
python -m skill_seekers.mcp.server

# Test MCP server (HTTP mode)
python -m skill_seekers.mcp.server --transport http --port 8765
```

## 🔧 Key Implementation Details

### CLI Architecture (Git-style)

**Entry point:** `src/skill_seekers/cli/main.py`

The unified CLI modifies `sys.argv` and calls existing `main()` functions to maintain backward compatibility:

```python
# Example: skill-seekers scrape --config react.json
# Transforms to: doc_scraper.main() with modified sys.argv
```

**Subcommands:** scrape, github, pdf, unified, enhance, package, upload, estimate, install

### Platform Adaptor Usage

```python
from skill_seekers.cli.adaptors import get_adaptor

# Get platform-specific adaptor

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [18701745572/-SkillSeeker-](https://github.com/18701745572/-SkillSeeker-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
