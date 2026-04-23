---
trigger: always_on
description: Unified interface library for working with multiple AI models (LLM, embedding, reranking, speech-to-text, text-to-speech) from different providers.
---

# Esperanto

Unified interface library for working with multiple AI models (LLM, embedding, reranking, speech-to-text, text-to-speech) from different providers.

## Core Value Proposition

Esperanto provides a **consistent, provider-agnostic interface** for AI models. Users can switch providers by changing one parameter, with identical code otherwise.

**Key principle**: Consistency across providers is the main value proposition. When adding features, maintain interface uniformity.

## Project Structure

```
esperanto/
├── src/esperanto/
│   ├── __init__.py                 # Public API exports
│   ├── factory.py                  # AIFactory for creating provider instances
│   ├── model_discovery.py          # Static model discovery system
│   ├── providers/                  # Provider implementations
│   │   ├── llm/                    # Language model providers
│   │   ├── embedding/              # Embedding providers
│   │   ├── reranker/               # Reranker providers
│   │   ├── stt/                    # Speech-to-text providers
│   │   └── tts/                    # Text-to-speech providers
│   ├── common_types/               # Shared type definitions
│   └── utils/                      # Cross-cutting utilities
└── tests/                          # Test suite

See detailed documentation in subdirectory CLAUDE.md files.
```

## Module Documentation

- **[src/esperanto/providers/](src/esperanto/providers/CLAUDE.md)**: All provider implementations
  - **[llm/](src/esperanto/providers/llm/CLAUDE.md)**: Language models (OpenAI, Anthropic, Google, etc.)
  - **[embedding/](src/esperanto/providers/embedding/CLAUDE.md)**: Embedding models (OpenAI, Jina, Voyage, etc.)
  - **[reranker/](src/esperanto/providers/reranker/CLAUDE.md)**: Reranking models (Jina, Voyage, Transformers)
  - **[stt/](src/esperanto/providers/stt/CLAUDE.md)**: Speech-to-text (OpenAI, Groq, Google, Azure)
  - **[tts/](src/esperanto/providers/tts/CLAUDE.md)**: Text-to-speech (OpenAI, ElevenLabs, Google, Azure)
- **[src/esperanto/common_types/](src/esperanto/common_types/CLAUDE.md)**: Response types and models
- **[src/esperanto/utils/](src/esperanto/utils/CLAUDE.md)**: Timeout, SSL, caching utilities

## Key Files

### factory.py

Central factory for creating provider instances:

- `AIFactory.create_language()`: Create LLM provider
- `AIFactory.create_embedding()`: Create embedding provider
- `AIFactory.create_reranker()`: Create reranker provider
- `AIFactory.create_speech_to_text()`: Create STT provider
- `AIFactory.create_text_to_speech()`: Create TTS provider
- `AIFactory.get_provider_models()`: Static model discovery (no instance needed)

**Registration**: All providers registered in `_provider_modules` dict by type and name.

### model_discovery.py

Static model discovery system:

- `PROVIDER_MODELS_REGISTRY`: Maps provider names to discovery functions
- Discovery functions return `List[Model]` without creating provider instances
- Results cached with `ModelCache` (1 hour TTL)

### __init__.py

Public API surface:

- Exports `AIFactory` (primary interface)
- Exports base classes (`LanguageModel`, `EmbeddingModel`, etc.)
- Conditionally imports provider classes (handles missing dependencies)

## Architecture Patterns

### Provider Pattern

All providers follow consistent architecture:

1. **Base class** defines interface (abstract methods)
2. **Provider implementations** inherit and implement interface
3. **Factory registration** makes provider discoverable
4. **Common response types** ensure consistency

### Configuration Priority

Three-tier configuration system (highest to lowest):

1. **Config dict**: `config={"timeout": 120}`
2. **Environment variables**: `ESPERANTO_LLM_TIMEOUT=90`
3. **Defaults**: Provider type defaults

### Mixin Composition

Providers inherit functionality via mixins:

- `TimeoutMixin`: Configurable HTTP timeouts
- `SSLMixin`: Configurable SSL verification
- Base class (e.g., `LanguageModel`): Provider-specific interface
- Provider implementation: Actual API integration

## Adding a New Provider

When building a provider:

1. **Research existing implementations**: Check base class and 2-3 sibling providers for patterns
2. **Follow the interface exactly**: Consistency is critical for user experience
3. **Implement all abstract methods**: Don't skip required methods
4. **Register in factory**: Add to `factory._provider_modules["{type}"]`
5. **Add optional import**: In `src/esperanto/__init__.py` with try/except
6. **Write tests**: Use `uv run pytest -v` to verify

**Critical pattern** - `__post_init__()`:

```python
def __post_init__(self):
    super().__post_init__()  # ALWAYS call first
    self.api_key = self.api_key or os.getenv("PROVIDER_API_KEY")
    self.base_url = self.base_url or "https://api.provider.com/v1"
    self._create_http_clients()  # ALWAYS call last
```

### Steps for New Provider

1. Identify provider type (language, embedding, reranker, stt, tts)
2. Create `{provider_name}.py` in appropriate `src/esperanto/providers/{type}/` directory
3. Import base class from `.base`
4. Implement all abstract methods
5. Follow `__post_init__()` pattern above

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lfnovo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
