---
trigger: always_on
description: SimpleLLMFunc is a lightweight LLM application framework enabling developers to write LLM-powered workflows and agents using Python decorators with DocString-as-Prompt philosophy. Core idea: **"Everything is Function, Prompt is Code"**.
---

# SimpleLLMFunc - AI Coding Agent Instructions

## 📋 Project Overview

SimpleLLMFunc is a lightweight LLM application framework enabling developers to write LLM-powered workflows and agents using Python decorators with DocString-as-Prompt philosophy. Core idea: **"Everything is Function, Prompt is Code"**.

## 🏗 Architecture & Key Components

### 1. **Core Design Pattern: Prompt as DocString**
- Function behavior is defined in the DocString, not the function body (which stays empty with `pass`)
- Decorators (`@llm_function`, `@llm_chat`, `@tool`) intercept function calls and delegate to LLM
- Type hints on parameters and return types ensure type safety and automatic validation
- Example from `/SimpleLLMFunc/llm_decorator/llm_function_decorator.py`:
  ```python
  @llm_function(llm_interface=my_llm)
  async def analyze_product_review(product_name: str, review_text: str) -> ProductReview:
      """You are a product review expert analyzing the following review...
      
      Args:
          product_name: Name of the product
          review_text: User review content
      
      Returns:
          Structured ProductReview object with rating, pros, cons, summary
      """
      pass  # Prompt as Code - LLM handles the actual execution
  ```

### 2. **Async-First Architecture**
- **All LLM decorators ONLY support `async def` functions** (non-negotiable requirement)
- Use `await` when calling decorated functions or `asyncio.run()` at top level
- Example from examples/event_stream_chatbot.py: consume `ResponseYield` / `EventYield` in an async loop
- Enables high-concurrency API calls via native asyncio integration

### 3. **Three-Tier Component Stack**

#### **Tier 1: Decorators** (`llm_decorator/`)
- **`@llm_function`**: Stateless single-call transformations (request → response)
- **`@llm_chat`**: Multi-turn conversations with history management
- **`@tool`**: Tool definitions for use by agents (async functions only)
- All accept `toolkit` parameter for tool composition
- Support `_template_params` for dynamic DocString templating (v0.2.14+)

#### **Tier 2: LLM Interface** (`interface/`)
- **`LLM_Interface` (abstract)**: Base contract for all LLM implementations
- **`OpenAICompatible`**: Universal implementation for any OpenAI-compatible API
  - Loads config from JSON files via `load_from_json_file()` method
  - Auto-routes to correct provider/model from hierarchical config
- **`APIKeyPool`**: Min-heap based load balancing for multiple API keys
- **`TokenBucket`**: Rate limiting to prevent API throttling

#### **Tier 3: Base Modules** (`base/`)
- **`messages/`**: Constructs system/user prompts + handles multimodal content
- **`post_process.py`**: Deserializes LLM responses to target types
- **`type_resolve/`**: Analyzes function signatures for type information
- **`ReAct.py`**: Orchestrates LLM calls with tool execution loops (max 5 by default)
- **`tool_call/`**: Tool extraction, invocation, and result validation

### 4. **Configuration & Providers** 
- `.env` file: `LOG_LEVEL=DEBUG` setting
- `provider.json`: Hierarchical structure with vendor → model → credentials
  ```json
  {
    "volc_engine": {
      "deepseek-v3-250324": {
        "api_keys": ["key1", "key2"],
        "base_url": "https://api.volc.example.com/v1",
        "model": "deepseek-chat",
        "rate_limit_capacity": 10
      }
    }
  }
  ```
- Load via: `OpenAICompatible.load_from_json_file("provider.json")["volc_engine"]["deepseek-v3-250324"]`

### 5. **Logging & Tracing System** (`logger/`)
- Console-only output (no file persistence in v0.2.13+)
- Auto-generated `trace_id` in format `{func_name}_{uuid}` for correlated logging
- Context manager: `async_log_context(trace_id=..., function_name=...)`
- Functions: `app_log()`, `push_error()`, `push_debug()`, `push_warning()`
- Structured context inheritance across async calls
- See `logger/core.py` for logger setup

## 🔄 Execution Data Flow

### Standard `llm_function` Flow:
1. **Call Capture** → Decorator intercepts `async def my_func(...)`
2. **Argument Binding** → Map args/kwargs to function signature
3. **Prompt Construction**:
   - System prompt = function's DocString + custom template (if provided)
   - User prompt = formatted arguments + type descriptions
4. **LLM Invocation** → Send to `llm_interface.chat(messages=[...])`
5. **Tool Loop** (if toolkit provided):
   - Check if LLM invoked tools via `tool_calls` field
   - Execute tools via `base/ReAct.py` (max 5 iterations)
   - Include tool results in next LLM message
6. **Response Deserialization** → Convert LLM text output to return type
   - If return type is Pydantic model: auto-parse JSON
   - If return type is primitive: direct conversion

### `llm_chat` Flow (Multi-turn):
1. Accepts `history: List[Dict[str, str]]` parameter
2. Maintains conversation state across calls
3. Yields tuples of `(content_chunk, updated_history)` in stream mode
4. Returns mode (`text` vs `raw`) affects tool call visibility in history
5. Final yield contains only `("", updated_history)` to signal completion

## 🛠 Tool System Patterns

### Function Decorator Way (Recommended):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NiJingzhe/SimpleLLMFunc](https://github.com/NiJingzhe/SimpleLLMFunc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
