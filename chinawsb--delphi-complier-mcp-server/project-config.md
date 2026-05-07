---
trigger: always_on
description: This file provides guidelines for agentic coding agents operating in this repository.
---

# AGENTS.md - Agent Coding Guidelines

This file provides guidelines for agentic coding agents operating in this repository.

## Project Overview

This is a **Delphi MCP Server** - a Model Context Protocol server that provides Delphi project compilation capabilities and knowledge base querying for AI assistants (Claude Desktop, CodeArts Agent, etc.).

- **Language**: Python 3.10-3.14
- **Platform**: Windows
- **Test Framework**: pytest (with pytest-asyncio for async tests)
- **Key Dependencies**: mcp>=0.9.0, pydantic>=2.0.0, beautifulsoup4, lxml, requests
- **Optional Dependencies**: PyMuPDF (recommended for PDF), pdfplumber (fallback for PDF), python-docx

## Project Structure

```
delphi-complier-mcp-server/
├── src/                      # Main source code
│   ├── server.py             # MCP Server entry point
│   ├── tools/               # MCP tool implementations
│   ├── services/            # Business logic services
│   ├── models/              # Data models (Pydantic/dataclasses)
│   └── utils/               # Utility functions
├── tests/                   # Test files
├── config/                  # Configuration files
├── data/                    # Knowledge base data
├── docs/                    # Documentation
├── build_kb.py             # Knowledge base builder (full source)
├── build_kb_fmx.py         # Knowledge base builder (FMX only)
└── pyproject.toml          # Project configuration
```

---

## Build, Lint, and Test Commands

### Environment Setup

```bash
# Create and activate virtual environment (Windows)
python -m venv venv
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
pip install -e ".[dev]"  # Includes pytest, pytest-asyncio, pytest-cov

# Install optional PDF processing libraries
pip install PyMuPDF  # Recommended for PDF processing (better performance)
# OR
pip install pdfplumber  # Alternative for PDF processing (pure Python)
```

### Windows Encoding Settings

**IMPORTANT**: On Windows, always set UTF-8 encoding:

```bash
# Option 1: Per command
PYTHONIOENCODING=utf-8 python your_script.py

# Option 2: For session
set PYTHONIOENCODING=utf-8
python your_script.py
```

### Development Setup

```bash
# Install all dependencies (including dev)
pip install -e ".[dev]"

# Install in development mode with editable source
pip install -e .
```

### Testing Commands

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_knowledge_base.py

# Run tests with verbose output
pytest -v

# Run a single test by name
pytest tests/test_knowledge_base.py::test_search_by_class_name -v

# Run tests with coverage report
pytest --cov=src --cov-report=html

# Run tests and generate XML coverage report for CI
pytest --cov=src --cov-report=xml

# Run tests with specific markers (if any are defined)
pytest -m "not slow"

# Run tests and stop after first failure
pytest -x
```

### Code Quality & Linting

```bash
# Type checking with mypy (if configured)
mypy src/

# Format code with black (if installed)
black src/ tests/

# Sort imports with isort (if installed)
isort src/ tests/

# Lint with flake8 (if installed)
flake8 src/ tests/
```

### Running the Server

```bash
# Run the MCP server
python src/server.py

# Run with specific configuration
python src/server.py --config config/config.json
```

### Building Knowledge Bases

```bash
# Build FMX only knowledge base (~30s, 311 files)
python build_kb_fmx.py

# Build full source knowledge base (~65s, 2768 files, 3.5M lines)
python build_kb.py

# Build project knowledge base (via MCP tool)
# delphi_kb(action="build", kb_type="project", project_path="path/to/project.dproj")

# Build Delphi help document knowledge base (via MCP tool, async)
# delphi_kb(action="build", kb_type="document", directory="C:\Program Files (x86)\Embarcadero\Studio\22.0\Help\Doc", extensions=[".chm"], async_mode=true)

# Run with custom paths
python build_kb.py --source C:\Delphi\Source --output data/delphi-kb/
```

---

## Delphi Knowledge Base Quick Reference

### Entity Types (Two-Letter Codes)

| Code | Type | Description |
|------|------|-------------|
| **TC** | Type/Class | class |
| **TR** | Type/Record | record |
| **TI** | Type/Interface | interface |
| **TE** | Type/Enum | enum |
| **TS** | Type/Set | set of |
| **TY** | Type | type alias |
| **FF** | Function | function |
| **FP** | Function | procedure |
| **CC** | Constant | const |
| **CR** | Constant | resourcestring |

### Search Functions

- Use `search_by_name()` for generic searches (covers all types)
- Use `search_by_class_name()` specifically for class types (TC)
- Prefer generic naming over single-letter codes for clarity

### Knowledge Base Types (via `delphi_kb` tool)

| kb_type | Description | Build Command |
|---------|------------|---------------|
| `delphi` | Delphi 官方源码 (RTL/VCL/FMX 等) | `action=build, version=<ver>` |
| `project` | 项目级知识库 (项目源码 + 三方库) | `action=build, project_path=<.dproj>` |
| `thirdparty` | 全局共享第三方库知识库 | `action=build, version=<ver>` |
| `document` | 通用文档 (txt/md/html/docx/pdf/epub/hlp/chm/网页) | `action=build` + `directory`/`url`/`urls` |

### Source Scanner File Extensions

The `DelphiSourceScanner` scans these extensions:
- `.pas`, `.dpr`, `.dpk`, `.dfm`, `.inc`

### Incremental Build Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chinawsb/delphi-complier-mcp-server](https://github.com/chinawsb/delphi-complier-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
