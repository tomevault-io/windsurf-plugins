---
trigger: always_on
description: These rules codify the design decisions and architectural patterns established during development. Follow these rules when making changes to the codebase.
---

# Project Rules - chatguru Agent

These rules codify the design decisions and architectural patterns established during development. Follow these rules when making changes to the codebase.

## Core Architecture Principles

### 1. Async-First Design
- **Rule**: All agent operations MUST be async-only. Never add synchronous methods to the Agent class.
- **Rationale**: Prevents blocking operations, aligns with FastAPI's async nature, ensures consistent API surface.
- **Implementation**: Use `async def` and `AsyncIterator` for all streaming operations.
- **Example**: ✅ `async def astream(messages: list[dict[str, str]], ...) -> AsyncIterator[str]` | ❌ `def run(...) -> str`

### 2. WebSocket for Real-Time Communication
- **Rule**: Use WebSocket endpoints for streaming responses. HTTP POST endpoints are deprecated for chat functionality.
- **Rationale**: Enables true real-time streaming, better UX, lower latency.
- **Implementation**: All streaming endpoints use `@router.websocket("/ws")` pattern.
- **Exception**: Health checks and static content can use HTTP GET.

### 3. Session ID Handling
- **Rule**: ALWAYS use `is not None` checks for `session_id` values, never truthiness checks.
- **Rationale**: Preserves empty string values which are valid but falsy.
- **Example**:
  ```python
  # ✅ Correct
  session_id = value if value is not None else "default"

  # ❌ Wrong - treats empty string as falsy
  session_id = value or "default"
  ```

### 4. API Contract Consistency
- **Rule**: ALL WebSocket responses (success, error, token, end) MUST include `session_id` field.
- **Rationale**: Maintains API contract, enables client-side error recovery, better debugging.
- **Implementation**: Extract `session_id` from raw JSON before validation fails, use "unknown" as fallback.

### 5. Comprehensive Error Handling
- **Rule**: All exceptions MUST be caught and converted to proper WebSocket error messages with `session_id`.
- **Rationale**: Prevents unhandled exceptions, maintains connection, enables error recovery.
- **Implementation**:
  - Agent initialization inside try-except block
  - Separate handlers for `json.JSONDecodeError`, `ValidationError`, `TypeError`
  - All error responses follow: `{"type": "error", "content": "...", "session_id": "..."}`

### 6. Type Validation Before Pydantic
- **Rule**: Validate JSON structure (dict vs array/string) BEFORE passing to Pydantic models.
- **Rationale**: Prevents `TypeError` from non-dict JSON, better error messages.
- **Implementation**: Check `isinstance(message_data, dict)` before `ChatMessage(**message_data)`.

### 7. Environment Variable Naming
- **Rule**: Use `LLM_*` prefix for model/key/version settings. The endpoint uses `OPENAI_ENDPOINT` (and `OPENAI_EMBEDDINGS_ENDPOINT`) because the application targets any OpenAI-compatible API, not just Azure.
- **Rationale**: Generic naming allows provider switching (true OpenAI, Azure APIM, Azure direct, etc.) without changing code.
- **Allowed**: `OPENAI_ENDPOINT`, `LLM_API_KEY`, `LLM_DEPLOYMENT_NAME`, `LLM_API_VERSION`, `OPENAI_EMBEDDINGS_ENDPOINT`, `OPENAI_EMBEDDINGS_API_KEY`
- **Deprecated**: `LLM_ENDPOINT` (replaced by `OPENAI_ENDPOINT`), `AZURE_OPENAI_*`, `BRAND_NAME` (use `APP_NAME`)

### 8. WebSocket Message Protocol
- **Rule**: Use explicit message types (`token`, `end`, `error`) in JSON format for all WebSocket messages.
- **Rationale**: Clear message boundaries, easy parsing, extensible.
- **Format**:
  ```json
  {"type": "token" | "end" | "error", "content": "...", "session_id": "..."}
  ```
- **Future**: Can extend with new types like `tool_call`, `metadata` without breaking changes.
- **Client request (incoming JSON)**: Required `messages` array — full conversation for the request; **last entry must be the current user turn** (`role: user`). Optional `session_id`, `visitor_id`. No separate top-level `message` field.

## Code Quality Rules

### 9. Streaming Configuration
- **Rule**: Enable streaming by default in LLM initialization (`streaming=True`).
- **Rationale**: Streaming is the primary use case, consistent behavior.
- **Implementation**: `AzureChatOpenAI(..., streaming=True)`

### 10. Test Client Selection
- **Rule**: Use FastAPI's `TestClient` for WebSocket testing, not `httpx.AsyncClient`.
- **Rationale**: Simpler test code, synchronous WebSocket handling, consistent patterns.
- **Implementation**: `TestClient(create_app())` for WebSocket tests.

### 11. Docker Healthchecks
- **Rule**: Include `curl` in Docker images for healthcheck functionality.
- **Rationale**: Enables Docker healthchecks without additional dependencies.
- **Implementation**: Install curl in Dockerfile, configure healthcheck in docker-compose.yml.

## File Organization Rules

### 12. HTML Templates Location
- **Rule**: Serve HTML templates from `src/api/templates/` directory.
- **Rationale**: Keeps templates with API code, simple deployment.
- **Implementation**: `Path(__file__).parent / "templates" / "index.html"`

### 13. Single Page Application at Root
- **Rule**: Serve web chat interface at root route (`/`) for simplicity.
- **Rationale**: Single application deployment, no CORS issues, easier development.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netguru/chatguru](https://github.com/netguru/chatguru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
