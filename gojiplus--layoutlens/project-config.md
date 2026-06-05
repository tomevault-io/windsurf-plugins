---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the LayoutLens codebase.
---

# CLAUDE.md - LayoutLens v1.4.0

This file provides guidance to Claude Code (claude.ai/code) when working with the LayoutLens codebase.

## Project Overview

 LayoutLens is a production-ready AI-powered UI testing framework that enables natural language visual testing. It captures screenshots using Playwright and analyzes them with OpenAI's GPT-4o Vision API to validate layouts, accessibility, responsive design, and visual consistency.

**Current Version:** v1.5.0 (includes major API simplification, architectural cleanup, and enhanced pathlib support)

## Quick Start Commands

### Installation
```bash
pip install layoutlens>=1.4.0
playwright install chromium
```

### Basic Usage
```bash
# Set API key
export OPENAI_API_KEY="your_key_here"

# Basic analysis
python -c "
from layoutlens import LayoutLens
lens = LayoutLens()
result = lens.analyze('https://example.com', 'Is the navigation user-friendly?')
print(f'Answer: {result.answer}')
print(f'Confidence: {result.confidence:.1%}')
"
```

### CLI Usage (v1.4.0 - Async-by-Default)
```bash
# Show system info and check setup
layoutlens info

# Analyze a single page with concurrent processing
layoutlens test --page https://example.com --queries "Is this page accessible?,Is the design professional?"

# Test with multiple viewports concurrently
layoutlens test --page mysite.com --queries "Good mobile UX?" --viewports "mobile_portrait,desktop"

# Compare two pages with async processing
layoutlens compare page1.html page2.html --query "Which design is better?"

# Batch process multiple sources efficiently
layoutlens batch --sources "site1.com,site2.com" --queries "Is it accessible?"

# Start interactive session with Rich formatting
layoutlens interactive

# Generate configuration file
layoutlens generate config --output my_config.yaml

# Validate configuration
layoutlens validate --config my_config.yaml
```

## Current API Structure (v1.4.0)

### Core LayoutLens Class
```python
from layoutlens import LayoutLens

# Initialize with LiteLLM unified provider support
lens = LayoutLens(
    api_key="your-key",        # Optional if OPENAI_API_KEY env var set
    model="gpt-4o-mini",       # Model to use (LiteLLM naming)
    provider="openai",         # "openai", "anthropic", "google", "gemini", "litellm"
    output_dir="custom_dir",   # Output directory for screenshots
    cache_enabled=True,        # Enable result caching for performance
    cache_type="memory"        # "memory" or "file"
)
```

### Main API Methods
```python
# Single page analysis
result = lens.analyze(
    source="https://example.com",  # URL or file path
    query="Is this page user-friendly?",
    viewport="desktop",            # "desktop", "mobile_portrait", "tablet_landscape"
    context={"user_type": "elderly"}  # Optional context
)

# Compare multiple sources
result = lens.compare(
    sources=["page1.html", "page2.html"],
    query="Which layout is better?",
    context={"focus": "accessibility"}
)

# Batch analysis (sync)
results = lens.analyze_batch(
    sources=["page1.html", "page2.html"],
    queries=["Is it accessible?", "Is it mobile-friendly?"],
    viewport="desktop"
)

# Async batch analysis for better performance
results = await lens.analyze_batch_async(
    sources=["page1.html", "page2.html"],
    queries=["Is it accessible?", "Is it mobile-friendly?"],
    max_concurrent=5
)

# Built-in checks
result = lens.check_accessibility("https://example.com")
result = lens.check_mobile_friendly("https://example.com")
result = lens.check_conversion_optimization("https://example.com")
```

### Result Objects
All analysis methods return objects with these properties:
```python
result.answer      # String: Natural language answer
result.confidence  # Float: Confidence score (0.0-1.0)
result.reasoning   # String: Detailed explanation
result.metadata    # Dict: Additional information
```

## Package Structure (v1.4.0)

```
layoutlens/
├── __init__.py           # Main exports
├── api/
│   ├── __init__.py
│   ├── core.py          # LayoutLens class with async support
│   └── test_suite.py    # Test suite execution
├── vision/
│   ├── __init__.py
│   ├── capture.py       # URLCapture class
│   ├── comparator.py    # LayoutComparator class
│   └── types.py         # VisionAnalysisRequest/Response dataclasses
├── providers/
│   ├── __init__.py
│   └── provider.py      # Simplified LayoutLensProvider (LiteLLM unified)
├── integrations/
│   ├── __init__.py
│   └── github.py        # GitHub Actions integration
├── cli.py               # Main CLI entry point
├── cli_commands.py      # Unified async command implementations
├── cli_interactive.py   # Interactive mode with Rich formatting
├── config.py            # Configuration management
├── cache.py             # Result caching system
├── logger.py            # Structured logging
└── exceptions.py        # Custom exception classes
```

## CLI Commands (v1.4.0 - Async-by-Default)

### test command
```bash
# Analyze single page with custom queries (concurrent processing)
layoutlens test --page https://example.com --queries "Is it accessible?,Is it responsive?"

# Analyze with multiple viewports concurrently

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gojiplus/layoutlens](https://github.com/gojiplus/layoutlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
