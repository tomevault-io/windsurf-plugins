---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

The project uses `uv` for package management and Make for common tasks:

- **Setup environment**: `make setup` (installs all dependencies with `uv sync --all-extras`)
- **Run tests**: `make check-tests` or `uv run pytest` (uses VCR cassettes by default)
- **Type checking**: `make check-types` or `uv run pyright`
- **Linting/formatting**: `make check-format` (check) or `make format` (fix)
- **Full checks**: `make check` (runs format, type, and test checks)
- **Build package**: `make build` (creates dist/ with built package)
- **Run single test**: `uv run pytest tests/test_specific_file.py::TestClass::test_method -v`
- **Update snapshots**: `make update-snaps` (for syrupy snapshot tests)
- **Update VCR cassettes**: `make update-snaps-vcr` (re-records HTTP interactions, requires API keys)
- **Check VCR secrets**: `make check-vcr-secrets` (scans cassettes for leaked credentials)
- **Documentation**: `make docs` (build) or `make docs-preview` (serve locally)

## Project Architecture

### Core Components

**Chat System**: The main `Chat` class in `_chat.py` manages conversation state and provider interactions. It's a generic class that works with different providers through the `Provider` abstract base class.

**Provider Pattern**: All LLM providers (OpenAI, Anthropic, Google, etc.) inherit from `Provider` in `_provider.py`. Each provider (e.g., `_provider_openai.py`) implements:
- Model-specific parameter handling 
- API client configuration
- Request/response transformation
- Tool calling integration

**Content System**: The `_content.py` module defines structured content types:
- `ContentText`: Plain text messages
- `ContentImage`: Image content (inline, remote, or file-based)
- `ContentToolRequest`/`ContentToolResult`: Tool interaction messages
- `ContentJson`: Structured data responses

**Tool System**: Tools are defined in `_tools.py` and allow LLMs to call Python functions. The system supports:
- Function registration with automatic schema generation
- Tool approval workflows
- MCP (Model Context Protocol) server integration via `_mcp_manager.py`

**Turn Management**: `Turn` objects in `_turn.py` represent individual conversation exchanges, containing sequences of `Content` objects.

### Key Patterns

1. **Provider Abstraction**: All providers implement the same interface but handle model-specific details internally
2. **Generic Typing**: Heavy use of TypeVars and generics for type safety across providers
3. **Streaming Support**: Both sync and async streaming responses via `ChatResponse`/`ChatResponseAsync`
4. **Content-Based Messaging**: All communication uses structured `Content` objects rather than raw strings
5. **Tool Integration**: Seamless function calling with automatic JSON schema generation from Python type hints

### Typing Best Practices

This project prioritizes strong typing that leverages provider SDK types directly:

- **Use provider SDK types**: Import and use types from `openai.types`, `anthropic.types`, `google.genai.types`, etc. rather than creating custom TypedDicts or dataclasses that mirror them. This ensures compatibility with SDK updates and provides better IDE support.
- **Use `@overload` for provider-specific returns**: When a method returns different types based on a provider argument, use `@overload` with `Literal` types to give callers precise return type information.
- **Explore SDK types interactively**: Use `python -c "from <sdk>.types import <Type>; print(<Type>.__annotations__)"` to inspect available fields and nested types when implementing provider-specific features.

### Testing Structure

- Tests are organized by component (e.g., `test_provider_openai.py`, `test_tools.py`)
- Snapshot testing with `syrupy` for response validation
- MCP server tests use local test servers in `tests/mcp_servers/`
- Async tests configured via `pytest.ini` with `asyncio_mode=strict`

### VCR Testing (HTTP Recording/Replay)

Tests use [pytest-recording](https://github.com/kiwicom/pytest-recording) (wrapping vcrpy) to record and replay HTTP interactions:

- **Cassettes**: YAML files stored in `tests/_vcr/` organized by test module
- **Default mode**: Tests replay cassettes without making live API calls
- **Recording**: Use `make update-snaps-vcr` or `uv run pytest --record-mode=rewrite` (requires real API keys)
- **Dummy credentials**: Auto-set by `conftest.py` when env vars are missing, enabling VCR replay without secrets

**Adding VCR to tests**:
```python
from .conftest import make_vcr_config, VCR_MATCH_ON_WITHOUT_BODY

# Most tests use default config (matches on request body)
@pytest.mark.vcr
def test_provider_simple():
    ...

# For tests with dynamic request bodies (temp files, generated IDs)
@pytest.fixture(scope="module")
def vcr_config():
    return make_vcr_config(match_on=VCR_MATCH_ON_WITHOUT_BODY)
```

**Tests requiring live API** (skip in VCR mode):
```python
from .conftest import is_dummy_credential

@pytest.mark.skipif(
    is_dummy_credential("ANTHROPIC_API_KEY"),
    reason="This test requires live API calls",
)
def test_multi_sample():
    ...
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [posit-dev/chatlas](https://github.com/posit-dev/chatlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
