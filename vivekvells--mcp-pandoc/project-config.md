---
trigger: always_on
description: *Last Updated: 2025-07-13*
---

# CLAUDE.md - mcp-pandoc Development Guide

*Last Updated: 2025-07-13*

## Project Overview

**mcp-pandoc** is a Model Context Protocol (MCP) server that provides document format conversion capabilities using Pandoc. This project enables seamless bidirectional conversion between 10+ document formats including Markdown, HTML, PDF, DOCX, LaTeX, EPUB, and more.

### Core Architecture
- **MCP Server**: Implements JSON-RPC 2.0 protocol for tool-based document conversion
- **Primary Tool**: `convert-contents` - handles all format conversions with comprehensive validation
- **Backend Engine**: Pandoc with pypandoc Python wrapper
- **Special Features**: Reference document styling for DOCX, ODT and PPTX, advanced format support

## 🎯 Project Philosophy & Decision Framework

### Core Principle: "Pandoc, Perfected for MCP"

mcp-pandoc follows the "iPhone approach" - **do fewer things, but do them perfectly**. We are the definitive Pandoc MCP server for document format conversion, not a universal document processor.

### Decision Framework for New Features

When evaluating any new feature request, assess against these criteria:

**✅ Green Light Criteria:**
- Native Pandoc capability that we're not fully leveraging
- High user value with low to medium implementation complexity
- Improves reliability or performance of existing features
- Enhances the core document conversion workflow

**⚠️ Yellow Light Criteria (Requires Strong Justification):**
- Adds new dependencies but provides significant user value
- Complex implementation but addresses critical user need
- Extension of Pandoc capabilities through well-established patterns

**❌ Red Light Criteria (Default: No):**
- Requires external tools not related to Pandoc
- Significant maintenance burden for niche use cases
- Better served by specialized servers in the MCP ecosystem
- Scope creep beyond document format conversion

### Maintenance Philosophy
- **Focused Excellence > Feature Breadth**
- **Reliability > Cutting-Edge Features**
- **Pandoc-Native > External Integrations**
- **Sustainable Development > Complex Dependencies**

### Examples of This Philosophy in Action

**✅ Good Additions (Pandoc-Native):**
- Citation & bibliography support (built-in citeproc)
- Math equation processing (native Pandoc capability)
- Custom template support (core Pandoc feature)
- Enhanced metadata handling (Pandoc strength)

**❌ Avoided Complexity:**
- Native diagram support (external Node.js dependency)
- Multi-format media conversion (belongs in specialized servers)
- Complex external tool integrations (maintenance burden)
- Custom syntax extensions (scope creep)

*Note: Features outside document conversion scope should be implemented in specialized servers rather than added to mcp-pandoc.*

## Development Environment Setup

### Required Dependencies
```bash
# Core dependencies (required)
brew install pandoc uv                    # macOS
sudo apt-get install pandoc && pip install uv  # Ubuntu/Debian

# PDF support (optional but recommended)
brew install texlive                      # macOS
sudo apt-get install texlive-xetex      # Ubuntu/Debian
```

### Development Workflow
```bash
# Setup and sync dependencies
uv sync

# Run locally for testing
uv run mcp-pandoc

# Run comprehensive test suite
uv run pytest tests/test_conversions.py

# Build for distribution
uv build

# Publish to PyPI
uv publish
```

### MCP Inspector for Debugging
```bash
npx @modelcontextprotocol/inspector uv --directory $(pwd) run mcp-pandoc
```

## Project Structure & Key Files

```
/mcp-pandoc/
├── src/mcp_pandoc/
│   ├── __init__.py              # Entry point with async main()
│   └── server.py                # Core MCP server implementation
├── tests/
│   ├── fixtures/                # Test input files for all formats
│   ├── output/                  # Test output directory
│   └── test_conversions.py      # Parametrized bidirectional testing
├── demo/                        # Screenshots and demo assets
├── README.md                    # Comprehensive user documentation
├── CHEATSHEET.md               # Quick reference guide
├── pyproject.toml              # Python project configuration
└── smithery.yaml               # MCP server distribution config
```

## MCP Protocol Best Practices

### Tool Definition Standards
- **JSON Schema Validation**: All parameters use comprehensive JSON Schema with enum validation
- **Error Handling**: Descriptive error messages with actionable guidance
- **Type Safety**: Full type hints throughout codebase
- **Protocol Compliance**: Strict JSON-RPC 2.0 adherence

### Security Implementation
```python
# File path validation pattern (server.py:161-162)
if reference_doc and not os.path.exists(reference_doc):
    raise ValueError(f"Reference document not found: {reference_doc}")

# Format validation pattern (server.py:165-167)
OUTPUT_FORMATS = ("markdown", "html", "pdf", "docx", "rst", "latex", "epub", "txt", "ipynb", "odt", "pptx")
if output_format not in OUTPUT_FORMATS:
    raise ValueError(f"Unsupported output format: '{output_format}'")
```

### Tool Architecture Pattern
- **Parameter Validation**: oneOf/allOf JSON Schema constraints for required parameters
- **Conditional Requirements**: Advanced formats require output_file paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vivekVells/mcp-pandoc](https://github.com/vivekVells/mcp-pandoc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
