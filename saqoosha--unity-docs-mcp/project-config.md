---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚨 IMPORTANT: Documentation Update Rule 🚨

**When making ANY code changes, you MUST update ALL related documentation:**

1. **Code Change → Documentation Update Checklist:**
   - [ ] README.md - If it affects installation or basic usage
   - [ ] docs/DETAILED_GUIDE.md - If it affects detailed usage or configuration
   - [ ] docs/ARCHITECTURE.md - If it changes technical design
   - [ ] docs/CHANGELOG.md - ALWAYS add an entry for changes
   - [ ] CLAUDE.md - If it affects development workflow

2. **Never commit code changes without updating docs**

3. **Documentation lives in:**
   - Root: User-facing (README.md, CLAUDE.md, LICENSE)
   - docs/: Technical and detailed documentation

## Unity Docs MCP Server v0.2.2 - Development Guide

### Commands

**Testing & Development**:
```bash
# Start MCP Inspector for interactive testing
./start_inspector.sh

# Run full test suite with coverage
python run_tests.py

# Direct test without Inspector
source venv/bin/activate && python test_mcp_tools.py

# Validate project structure
python validate_structure.py

# Install development dependencies
pip install -e .
```

**Running the Server**:
```bash
# Via entry point (after installation) - shows startup info
unity-docs-mcp
# Output:
# 🚀 Unity Docs MCP Server v0.2.1
# 📚 Supporting Unity versions 2019.1 - 6000.2
# 💾 Advanced caching enabled (6h API + 24h search index)
# 🔌 Starting MCP server...

# IMPORTANT: For Claude Desktop, use the full path to avoid "module not found" errors:
# /path/to/unity-docs-mcp/venv/bin/unity-docs-mcp

# Direct module execution
python -m unity_docs_mcp.server
```

### Architecture Overview

The project consists of four main modules that process Unity documentation:

1. **server.py** - MCP server implementation providing 4 tools:
   - `list_unity_versions` - Lists supported Unity versions
   - `suggest_unity_classes` - Provides class name suggestions  
   - `get_unity_api_doc` - Fetches API documentation for **exact version only**
   - `search_unity_docs` - Searches Unity documentation in specified version

2. **scraper.py** - Handles web requests to Unity's documentation site:
   - Implements rate limiting (0.5s between requests)
   - Builds URLs for API docs
   - Validates Unity versions
   - **Advanced caching system for API availability checks (6-hour cache)**
   - **Uses search_index.py for search instead of web scraping**

3. **parser.py** - Critical HTML processing pipeline:
   - **MUST remove `<a>` tags BEFORE Trafilatura** (prevents bracket issues)
   - Removes Unity UI elements (feedback forms, etc.)
   - Converts to clean Markdown

4. **search_index.py** - Local search index implementation:
   - Downloads and caches Unity's JavaScript search index per version
   - Each version has its own index file and cache (24-hour cache)
   - Implements client-side search logic in Python
   - Provides fast, reliable search without JavaScript execution

### Version-Specific Behavior

**Important**: The MCP server provides intelligent version handling:

#### Version Normalization
- **Automatic normalization**: `6000.0.29f1` → `6000.0`, `2022.3.45f1` → `2022.3`
- **Full version support**: Accepts any Unity version format (alpha, beta, final)
- **Transparent conversion**: Shows original and normalized versions in results

#### Dynamic Latest Version Detection
- **Smart defaults**: When no version specified, automatically uses latest available
- **Unity redirect detection**: Leverages Unity's own version redirection
- **No maintenance required**: Always up-to-date with Unity releases

#### Version Availability Information
- **404 with context**: When API not found, shows which versions have it
- **Cross-version checking**: Fast HEAD requests to determine availability with intelligent caching
- **Helpful suggestions**: "Available in versions: 6000.0, 2022.3" for upgrade decisions
- **Performance optimized**: API availability checks are cached for 6 hours to minimize web requests

#### Strict Version Accuracy
- **No fallback**: Searches only in the exact specified (normalized) version
- **Clear error messages**: Precise information about version availability
- **User control**: Developers get docs for their exact Unity version

### Critical Implementation Details

1. **HTML Link Removal is CRUCIAL** - Must remove `<a>` tags BEFORE Trafilatura
2. **Processing Pipeline Order**: HTML → Remove Links → Remove UI → Trafilatura → Clean
3. **Trafilatura's `include_links=False` is NOT enough** - it leaves `[text]` brackets
4. **Search Algorithm**: Implements Unity's exact scoring system for 100% accuracy
5. **Namespace Resolution**: Dynamic discovery using search index, no hardcoding
6. **Pre-commit Testing**: Basic functionality tests run automatically before commits

### Common Issues & Solutions

- **Brackets in code**: `[GameObject]` → Remove `<a>` tags at HTML level
- **UI elements**: "Leave feedback" → Remove with `_remove_unity_ui_elements()`
- **Bold text**: `**text**` → Remove `<strong>`, `<b>` tags and markdown formatting
- **Markdown links**: `[ComputeBuffer](ComputeBuffer.html)` → Strip with regex

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Saqoosha/unity-docs-mcp](https://github.com/Saqoosha/unity-docs-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
