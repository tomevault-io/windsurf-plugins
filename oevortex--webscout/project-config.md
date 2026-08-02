---
trigger: always_on
description: LLM4Free is a comprehensive Python toolkit providing unified access to 40+ AI providers, search engines, and digital utilities. This document guides AI agents for productive development in this codebase.
---

# LLM4Free AI Agent Instructions

## Overview
LLM4Free is a comprehensive Python toolkit providing unified access to 40+ AI providers, search engines, and digital utilities. This document guides AI agents for productive development in this codebase.

## Quick Start

### Development Environment
```bash
# Recommended: Use uv for modern Python package management
uv add llm4free
uv run llm4free --help

# For development with all features
uv add "llm4free[dev,api]"

# Run tests
uv run pytest

# Live testing (requires API keys)
uv run python tests/live_test.py --list
uv run python tests/live_test.py --provider OpenAI --api-key YOUR_KEY
```

### Core Concepts
- **OpenAI Compatibility**: All providers implement identical `chat.completions.create()` interface
- **Dynamic Loading**: Providers loaded at runtime via `_load_providers()`
- **Unified Client**: Use `llm4free.client.Client` for all provider interactions
- **Modular Architecture**: Clear separation between providers, search engines, and utilities

## Essential Commands

### Provider Discovery & Usage
```python
from llm4free import Client

# Initialize unified client
client = Client()

# List available providers
providers = client.list_providers()

# Chat with auto-failover
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)
```

### Search Operations
```python
from llm4free import DuckDuckGoSearch, BingSearch, BraveSearch

# Search with any engine
search = DuckDuckGoSearch()
results = search.search("Python web scraping")
```

### CLI Usage
```bash
# List available commands
llm4free --help

# Search with DuckDuckGo
llm4free ddg "Python tutorials"

# Weather information
llm4free weather "New York"
```

## Key Files & Patterns

### Core Interface Files
- `llm4free/client.py` - Unified client for all providers
- `llm4free/cli.py` - Rich-powered command-line interface
- `llm4free/server/` - OpenAI-compatible API server

### Provider Architecture
- `llm4free/Provider/llm/` - OpenAI-compatible providers
- `llm4free/Provider/OPENAI/` - OpenAI-compatible wrappers
- `llm4free/Provider/TTI/` - Text-to-image providers
- `llm4free/Provider/TTS/` - Text-to-speech providers

### Search Module
- `llm4free/search/` - Search engine implementations
- `llm4free/search/engines/` - Provider-specific engines

### Base Classes
- `llm4free/Provider/llm/base.py` - `OpenAICompatibleProvider` abstract base
- `llm4free/Provider/llm/utils.py` - Response utilities

## Development Best Practices

### Code Quality
- **Type hints**: Use for all public functions
- **Google-style docstrings**: Comprehensive documentation
- **Imports**: Standard lib → third-party → local
- **Error handling**: Comprehensive with provider fallbacks

### Testing Strategy
```bash
# Unit tests (mocked)
uv run pytest tests/providers/

# Live testing (requires API keys)
uv run python tests/live_test.py --test-all --api-keys-file keys.json
```

### Adding New Functionality

**To add a new provider:**
1. Implement in `llm4free/Provider/llm/` (or appropriate subdirectory)
2. Update `Provider.md` with provider matrix
3. Consider adding to `llm4free/models.py` for registry

**To add a CLI command:**
1. Update `llm4free/cli.py` with command group
2. Add corresponding search engine/provider
3. Update `docs/cli.md`

**To add server capability:**
1. Update `llm4free/server/` with new routes
2. Document in `docs/openai-api-server.md`
3. Ensure CLI/Client can hit new endpoints

## Common Patterns

### Provider Authentication
```python
# Some providers require auth (marked with required_auth = True)
# Keys can be passed via:
client = Client(api_key="your-key")
# or via JSON file
client = Client(api_keys_file="keys.json")
```

### Model Resolution
```python
# Client handles model name mapping across providers
response = client.chat.completions.create(
    model="gpt-4o",  # Maps to available provider/model
    messages=[...]
)
```

### Tool Support
```python
# Providers support OpenAI-style tool calling
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    tools=[
        {
            "type": "function",
            "function": {
                "name": "get_weather",
                "description": "Get weather information",
                "parameters": {...}
            }
        }
    ]
)
```

## Troubleshooting

### Common Issues
1. **Provider not found**: Use `client.list_providers()` to see available options
2. **Authentication errors**: Check `required_auth` attribute and provide API keys
3. **Model not available**: Client performs fuzzy matching across providers
4. **Import errors**: Use dynamic loading via `llm4free.client.load_openai_providers()`

### Error Handling
```python
try:
    response = client.chat.completions.create(...)
except Exception as e:
    # Client provides detailed error messages
    print(f"Error: {e}")
    # Try fallback provider
    response = client.chat.completions.create(
        model="alternative-model",
        messages=...
    )
```

## Integration Points

### When to Use Which Component

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OEvortex/Webscout](https://github.com/OEvortex/Webscout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
