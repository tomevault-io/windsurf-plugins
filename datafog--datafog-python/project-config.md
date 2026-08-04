---
trigger: always_on
description: **DataFog** is an open-source Python library for PII detection and anonymization with a focus on speed and lightweight architecture.
---

# DataFog - Agent Development Guide

## Project Overview

**DataFog** is an open-source Python library for PII detection and anonymization with a focus on speed and lightweight architecture.

## Core Value Proposition

- **Ultra-Fast Performance**: 100x+ faster than spaCy/Presidio for structured PII (reproduce: `python benchmarks/run.py`), 32x faster with GLiNER
- **Lightweight Core**: <2MB package with optional ML extras
- **Modern Engine Options**: Regex, GLiNER, spaCy, and smart cascading
- **Production Ready**: Comprehensive testing, CI/CD, and performance validation

## Current Project Status

**Stable version: 4.8.0**

**Development version: 4.8.0**

**Next major target: 5.0.0**

### ✅ Recently Completed (Latest)

- **Redaction Correctness**: Document-order token numbering, overlap-safe span handling, and stable mask mappings
- **Allowlists and Typing**: Exact and regex allowlists, Presidio-style aliases, and `py.typed`
- **Agent & Gateway Firewall**: Claude Code hook and LiteLLM guardrail adapters
- **GLiNER Integration**: Modern NER engine with PII-specialized models
- **Performance Validation**: 100x+ regex speedup reproducible via `benchmarks/`; 32x GLiNER

## Quick Development Setup

```bash
# 1. Clone and setup environment
git clone https://github.com/datafog/datafog-python.git
cd datafog-python
python -m venv .venv && source .venv/bin/activate

# 2. Install with dev dependencies
pip install -e ".[dev]" && pip install -r requirements-dev.txt
pre-commit install

# 3. Install ML extras for advanced features
pip install -e ".[nlp]"           # spaCy
pip install -e ".[nlp-advanced]"  # GLiNER
pip install -e ".[all]"           # Everything

# 4. Verify installation
python -c "from datafog.services.text_service import TextService; print('✅ All engines:', ['regex', 'gliner', 'spacy', 'smart', 'auto'])"
```

## Architecture Overview

### Engine Ecosystem (Updated with GLiNER)

```python
from datafog.services.text_service import TextService

# Core engines (always available)
regex_service = TextService(engine="regex")      # 100x+ faster, structured PII

# ML engines (require extras)
gliner_service = TextService(engine="gliner")    # 32x faster, modern NER
spacy_service = TextService(engine="spacy")      # Comprehensive NLP

# Smart combinations
smart_service = TextService(engine="smart")      # Cascading: regex→GLiNER→spaCy
auto_service = TextService(engine="auto")        # Legacy: regex→spaCy
```

### Performance Comparison (Validated)

| Engine   | Speed vs spaCy   | Accuracy          | Use Case                    | Install          |
| -------- | ---------------- | ----------------- | --------------------------- | ---------------- |
| `regex`  | **100x+ faster** | High (structured) | Emails, phones, SSNs        | Core only        |
| `gliner` | **32x faster**   | Very High         | Modern NER, custom entities | `[nlp-advanced]` |
| `spacy`  | 1x (baseline)    | Good              | Traditional NLP             | `[nlp]`          |
| `smart`  | **60x faster**   | Highest           | Best balance                | `[nlp-advanced]` |

### Dependency Strategy

```python
# Lightweight core (<2MB)
pip install datafog

# Optional ML engines
pip install datafog[nlp]           # spaCy (traditional NLP)
pip install datafog[nlp-advanced]  # GLiNER (modern NER)
pip install datafog[ocr]           # Image processing
pip install datafog[all]           # Everything
```

## GLiNER Integration

### Overview

GLiNER (Generalist Model for Named Entity Recognition) provides modern, accurate NER capabilities optimized for PII detection.

### Key Features

- **PII-Specialized Models**: `urchade/gliner_multi_pii-v1` trained specifically for PII
- **Custom Entity Types**: Configurable entity detection beyond default PII types
- **Smart Cascading**: Automatically tries regex first, GLiNER second, spaCy last
- **CLI Management**: Download and manage GLiNER models via CLI

### Usage Examples

```python
# GLiNER engine
from datafog.services.text_service import TextService
service = TextService(engine="gliner", gliner_model="urchade/gliner_multi_pii-v1")
result = service.annotate_text_sync("Dr. John Doe at john@hospital.org")
# Detects: PERSON, EMAIL, and more

# Smart cascading (recommended)
smart_service = TextService(engine="smart")
result = smart_service.annotate_text_sync(text)
# Uses regex for speed, GLiNER for accuracy, spaCy as fallback

# CLI model management
subprocess.run(["datafog", "download-model", "urchade/gliner_base", "--engine", "gliner"])
subprocess.run(["datafog", "list-models", "--engine", "gliner"])
```

### Available GLiNER Models

- `urchade/gliner_multi_pii-v1` - PII-specialized (recommended)
- `urchade/gliner_base` - General purpose starter
- `urchade/gliner_large-v2` - Higher accuracy
- `knowledgator/modern-gliner-bi-large-v1.0` - 4x more efficient

## Development Workflow

### Git Branch Strategy

- **main**: Production releases only
- **dev**: Main development branch (use this)
- **feature/\***: New features from dev
- **fix/\***: Bug fixes from dev

### Making Changes

```bash
# Start from dev
git checkout dev && git pull origin dev

# Create feature branch
git checkout -b feature/your-change

# Make changes, test, commit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataFog/datafog-python](https://github.com/DataFog/datafog-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
