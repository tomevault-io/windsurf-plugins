---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP PDF is a FastMCP server that provides comprehensive PDF processing capabilities including text extraction, table extraction, OCR, image extraction, and format conversion. The server is built on the FastMCP framework and provides intelligent method selection with automatic fallbacks.

## Development Commands

### Environment Setup
```bash
# Install with development dependencies
uv sync --dev

# Install system dependencies (Ubuntu/Debian)
sudo apt-get install tesseract-ocr tesseract-ocr-eng poppler-utils ghostscript python3-tk default-jre-headless
```

### Testing
```bash
# Run all tests
uv run pytest

# Run with coverage
uv run pytest --cov=mcp_pdf_tools

# Run specific test file
uv run pytest tests/test_server.py

# Run specific test
uv run pytest tests/test_server.py::TestTextExtraction::test_extract_text_success
```

### Code Quality
```bash
# Format code
uv run black src/ tests/ examples/

# Lint code
uv run ruff check src/ tests/ examples/

# Type checking
uv run mypy src/
```

### Security Scanning
```bash
# Check for known vulnerabilities in dependencies
uv run safety check

# Audit Python packages for known vulnerabilities
uv run pip-audit

# Run comprehensive security scan
uv run safety check --json && uv run pip-audit --format=json
```

### Running the Server
```bash
# Run MCP server directly
uv run mcp-pdf

# Verify installation
uv run python examples/verify_installation.py

# Test with sample PDF
uv run python examples/test_pdf_tools.py /path/to/test.pdf
```

### Building and Distribution
```bash
# Build package
uv build

# Upload to PyPI (requires credentials)
uv publish
```

## Architecture

### Core Components

- **`src/mcp_pdf_tools/server.py`**: Main server implementation with all PDF processing tools
- **FastMCP Framework**: Uses FastMCP for MCP protocol implementation
- **Multi-library approach**: Integrates PyMuPDF, pdfplumber, pypdf, Camelot, Tabula, and Tesseract

### Tool Categories

1. **Text Extraction**: `extract_text` - Writes extracted text to a .txt file by default, returns path + preview. Set `inline=True` for full text in response.
2. **Table Extraction**: `extract_tables` - Auto-fallback through Camelot → pdfplumber → Tabula
3. **OCR Processing**: `ocr_pdf` - Tesseract with preprocessing options
4. **Document Analysis**: `is_scanned_pdf`, `get_document_structure`, `extract_metadata`
5. **Format Conversion**: `pdf_to_markdown` - Writes markdown + extracted raster images and vector graphics (SVG) to disk by default, returns path + preview. Images use relative `./images/` paths, vectors use `./vectors/` paths. Set `inline=True` for full markdown in response. Set `include_vectors=False` to skip vector extraction. Use `output_filename` to override the default .md filename. When `include_vectors=True`, returns `vector_diagnostics` showing which pages had drawings below the complexity threshold. Set `vector_fallback_raster=True` to render those sub-threshold pages as full-page raster images (PNG at 150 DPI) instead of skipping them. `markdown_to_pdf` - Reverse direction: converts a `.md` file or inline markdown text to PDF using pandoc. Auto-detects available PDF engines (xelatex, pdflatex, tectonic, weasyprint, wkhtmltopdf) and picks the best one on PATH. Pass `pdf_engine` to override or `extra_args` for raw pandoc options. Requires `pip install mcp-pdf[markdown]` and the pandoc binary.
6. **Image Processing**: `extract_images` - Extract images with custom output paths and clean summary output
7. **Link Extraction**: `extract_links` - Extract all hyperlinks with page filtering and type categorization
8. **PDF Forms**: `extract_form_data`, `create_form_pdf`, `fill_form_pdf`, `add_form_fields` - Complete form lifecycle management. **`extract_form_data` aligned to a six-term portable vocabulary** (text/checkbox/radio/dropdown/date/signature, plus button/unknown) so callers see the same field-type strings whether the form is AcroForm or XFA. **XFA detection wired in**: dynamic XFA forms now return a diagnostic error with `hint: "extract_xfa_fields"` instead of cryptic "document closed".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rsp2k/mcp-pdf](https://github.com/rsp2k/mcp-pdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
