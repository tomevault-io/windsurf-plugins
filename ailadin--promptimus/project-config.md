---
trigger: always_on
description: promptimus python library for building AI agents
---

# Promptimus Skill Reference

This is the sole reference for AI coding agents building agents with Promptimus.

---

## 1. Import Map

```python
import promptimus as pm
```

### Top-level exports (`pm.*`)

| Symbol          | Type     |
|-----------------|----------|
| `pm.Module`     | ABC base class for all agents |
| `pm.Parameter`  | Serializable config container |
| `pm.Prompt`     | Single-shot LLM call with template |
| `pm.Message`    | Pydantic model for chat messages |
| `pm.MessageRole`| StrEnum: `USER`, `SYSTEM`, `ASSISTANT`, `TOOL` |
| `pm.ImageContent`| Pydantic model for image payloads |
| `pm.Usage`      | Pydantic model for LLM token usage |

### Modules (`pm.modules.*`)

| Symbol | Description |
|--------|-------------|
| `pm.modules.MemoryModule` | Multi-turn conversation with rolling history |
| `pm.modules.StructuralOutput` | Pydantic model extraction from LLM |
| `pm.modules.Tool` | Function/module wrapper for agent tools |
| `pm.modules.ToolCallingAgent` | ReACT text-based tool loop |
| `pm.modules.OpenaiToolCallingAgent` | Native OpenAI function calling loop |
| `pm.modules.RetrievalModule` | Hybrid search (vector + text + reranking) |
| `pm.modules.RAGModule` | Retrieval + memory composition |
| `pm.modules.ResetMemoryContext` | Context manager to clear all memories |
| `pm.modules.SupportsHandoff` | Protocol for handoff-capable modules |

### LLMs (`pm.llms.*`)

| Symbol | Description |
|--------|-------------|
| `pm.llms.OpenAILike` | OpenAI-compatible LLM client |
| `pm.llms.OllamaProvider` | Convenience wrapper for local Ollama |

### Embedders (`pm.embedders.*`)

| Symbol | Description |
|--------|-------------|
| `pm.embedders.OpenAILikeEmbedder` | OpenAI-compatible embedding client |

### Rerankers (`pm.rerankers.*`)

| Symbol | Description |
|--------|-------------|
| `pm.rerankers.RerankerProtocol` | Protocol for query/result-list rerankers |
| `pm.rerankers.RRFReranker` | Reciprocal Rank Fusion reranker (default fallback) |
| `pm.rerankers.OpenAILikeReranker` | API reranker for OpenAI-compatible `/rerank` endpoints (e.g. Cohere via OpenRouter) |

### Not re-exported (direct import required)

```python
from promptimus.llms.dummy import DummyLLm        # Testing stub
from promptimus.modules.memory import Memory       # Raw shared state buffer
```

---

## 2. Core Rules

1. **Every agent is a `pm.Module` with `async def forward()`.** Always call `super().__init__()` in your `__init__`.
2. **Assign submodules and Parameters as instance attributes in `__init__`.** They are auto-registered via `__setattr__` for hierarchy tracking, serialization, and digest computation.
3. **Configure providers once on the root module.** Call `.with_llm()`, `.with_embedder()`, and `.with_reranker()` on the root -- they propagate recursively to all submodules. Stores (vector_store, text_store) are passed directly as constructor arguments to `RetrievalModule` / `RAGModule`. Reranker has a built-in default (`RRFReranker(k=60)`), so `.with_reranker()` is only needed to override it.

---

## 3. Data Types

### `pm.Message`

Pydantic model. Fields:
- `role: MessageRole | str` -- the message role
- `content: str` -- text content
- `images: list[ImageContent]` -- image attachments (default `[]`)
- `tool_calls: list[ToolRequest] | None` -- OpenAI-style tool calls (default `None`)
- `tool_call_id: str | None` -- for TOOL role responses
- `reasoning: str | None` -- reasoning content from supported models
- `usage: Usage | None` -- token usage from LLM response (populated by `OpenAILike`)

### `pm.MessageRole`

StrEnum with values: `USER = "user"`, `SYSTEM = "system"`, `ASSISTANT = "assistant"`, `TOOL = "tool"`.

### `pm.ImageContent`

- `ImageContent(url="https://...")` -- direct URL
- `ImageContent.from_buffer(buffer: BytesIO, mimetype: str)` -- base64-encodes a buffer

### `pm.Usage`

Pydantic model for token usage. Populated automatically by `OpenAILike` from `response.usage`.
- `prompt_tokens: int`
- `completion_tokens: int`
- `total_tokens: int`
- `cached_tokens: int | None` -- from `prompt_tokens_details.cached_tokens`
- `reasoning_tokens: int | None` -- from `completion_tokens_details.reasoning_tokens`

### Forward convention

Most modules accept `list[Message] | Message | str` and return `Message`. Exceptions noted per block.

---

## 4. Provider Setup

### OpenAILike

```python
llm = pm.llms.OpenAILike(
    model_name="gpt-4.1-nano",
    # All extra kwargs pass to AsyncOpenAI():
    # api_key="...", base_url="...", etc.
)
```

API key is auto loaded from env for OpenAI, no need to load it explicitly.

Constructor: `OpenAILike(model_name: str, call_kwargs: dict | None = None, max_concurrency: int = 10, n_retries: int = 5, base_wait: float = 3.0, **client_kwargs)`

`client_kwargs` are forwarded to `AsyncOpenAI(...)`.

### OllamaProvider

```python
llm = pm.llms.OllamaProvider(
    model_name="gemma3:4b",
    base_url="http://localhost:11434/v1",
)
```

Constructor: `OllamaProvider(model_name: str, base_url: str)`. Wraps `OpenAILike` internally with `api_key="DUMMY"`.

### OpenAILikeEmbedder

```python
embedder = pm.embedders.OpenAILikeEmbedder(
    model_name="text-embedding-3-small",
    # api_key="...", base_url="...", etc.
)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AIladin/promptimus](https://github.com/AIladin/promptimus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
