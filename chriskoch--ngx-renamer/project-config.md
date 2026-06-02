---
trigger: always_on
description: AI coding agent for ngx-renamer, a Paperless NGX document title generator
---


# AGENTS.md

You are an expert Python developer working on **ngx-renamer**, an AI-powered document title generator for Paperless NGX. Your role is to maintain, test, and improve the codebase while following established conventions.

## Project Overview

**Tech Stack:**
- Python 3.8+
- OpenAI API (GPT-4o, GPT-4o-mini) with structured outputs
- Anthropic Claude API (Claude 3.5 Sonnet) with tool calling
- Ollama API (local LLM support) with JSON schema validation
- PyYAML for configuration
- pytest for testing
- Docker for deployment

**Project Structure:**
```
ngx-renamer/
├── change_title.py          # Main entry point
├── modules/
│   ├── base_llm_provider.py    # Abstract base class for LLM providers
│   ├── openai_titles.py        # OpenAI integration
│   ├── claude_titles.py        # Anthropic Claude integration
│   ├── ollama_titles.py        # Ollama integration
│   ├── paperless_ai_titles.py  # Paperless API orchestrator
│   ├── logger.py               # Centralized logging configuration
│   ├── constants.py            # Constants and schemas
│   ├── exceptions.py           # Custom exception hierarchy
│   ├── llm_factory.py          # Provider factory with registry pattern
│   ├── paperless_client.py     # Paperless NGX API client
│   └── providers/              # Provider plugin directory
│       └── __init__.py         # Provider registry and auto-discovery
├── scripts/
│   ├── init-and-start.sh           # Docker entrypoint
│   ├── setup-venv-if-needed.sh     # Venv initialization
│   └── post_consume_wrapper.sh     # Post-consume hook
├── tests/
│   ├── conftest.py                 # Test fixtures
│   ├── fixtures/                   # Test data (YAML configs)
│   └── integration/                # Integration tests
├── settings.yaml            # Configuration file
├── requirements.txt         # Runtime dependencies
└── requirements-dev.txt     # Development dependencies
```

## Commands

### Development Setup
```bash
# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt requirements-dev.txt

# Configure environment
cp .env.example .env
# Edit .env with your API keys
```

### Testing
```bash
# Run all tests
pytest tests/

# Run specific test categories
pytest -m smoke          # Critical smoke tests
pytest -m integration    # All integration tests
pytest -m openai        # OpenAI API tests (costs money, requires OPENAI_API_KEY)
pytest -m claude        # Claude API tests (costs money, requires CLAUDE_API_KEY)
pytest -m ollama        # Ollama tests (requires Ollama running on localhost:11434)

# Run specific test files
pytest tests/integration/test_openai_integration.py
pytest tests/integration/test_claude_integration.py
pytest tests/integration/test_ollama_integration.py
pytest tests/integration/test_llm_provider_selection.py
pytest tests/integration/test_paperless_integration.py
pytest tests/integration/test_end_to_end.py

# Coverage reporting
pytest --cov=modules --cov-report=html
pytest --cov=modules --cov-report=term-missing

# Skip expensive API tests
pytest -m "not openai and not ollama"
```

### Manual Testing
```bash
# Test title generation with sample text
python3 test_title.py

# Test with your own PDF
python3 ./test_pdf.py path/to/your/ocr-ed/pdf/file
```

### Docker Integration
```bash
# Check logs in Paperless container
docker compose logs webserver | grep ngx-renamer

# Force rebuild venv
docker compose exec webserver rm /usr/src/ngx-renamer-venv/.initialized
docker compose restart webserver

# Test API connectivity
docker compose exec webserver curl http://webserver:8000/api/
docker compose exec webserver curl http://host.docker.internal:11434/api/version
```

## Code Style & Conventions

### Python Style
- **Follow PEP 8** for all Python code
- Use **type hints** where appropriate
- Keep functions focused and under 50 lines when possible
- Use descriptive variable names (no single letters except loop counters)

### Example: Good Code Style
```python
def generate_title_from_text(self, text: str) -> str:
    """
    Generate an AI-powered title from OCR text.

    Args:
        text: OCR-extracted document content

    Returns:
        Generated title string (max 127 characters)
    """
    prompt = self._build_prompt(text)
    response = self._call_llm_api(prompt)
    return self._extract_title(response)
```

### Example: Bad Code Style
```python
# ❌ Avoid this
def gen(t):  # Unclear name, no docstring, no type hints
    p = self.make_p(t)
    r = self.call(p)
    return r.strip()
```

### Naming Conventions
- **Classes**: `PascalCase` (e.g., `PaperlessAITitles`, `OpenAITitles`)
- **Functions/Methods**: `snake_case` (e.g., `generate_title`, `_build_prompt`)
- **Private methods**: Prefix with `_` (e.g., `_call_openai_api`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `DEFAULT_MODEL`, `MAX_TITLE_LENGTH`)
- **Test files**: `test_*.py` (e.g., `test_openai_integration.py`)
- **Test functions**: `test_*` (e.g., `test_ollama_title_generation`)

### LLM Provider Pattern (v1.3.0+)
All LLM providers inherit from `BaseLLMProvider` and are registered in the provider registry:

```python
from typing import Optional

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chriskoch/ngx-renamer](https://github.com/chriskoch/ngx-renamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
