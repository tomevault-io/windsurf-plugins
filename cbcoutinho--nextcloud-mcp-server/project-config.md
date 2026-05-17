---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Coding Conventions

### async/await Patterns
- **Use anyio for all async operations** - Provides structured concurrency
  - pytest runs in `anyio` mode (`anyio_mode = "auto"` in pyproject.toml)
  - Use `anyio.create_task_group()` for concurrent execution (NOT `asyncio.gather()`)
  - Use `anyio.Lock()` for synchronization primitives (NOT `asyncio.Lock()`)
  - Use `anyio.run()` for entry points (NOT `asyncio.run()`)
  - Prefer standard async/await syntax without explicit library imports when possible
  - Examples: app.py, search/hybrid.py, search/verification.py, auth/token_broker.py

### Type Hints
- **Use Python 3.10+ union syntax**: `str | None` instead of `Optional[str]`
- **Use lowercase generics**: `dict[str, Any]` instead of `Dict[str, Any]`
- **Type all function signatures** - Parameters and return types
- **Type checker**: `ty` is configured for static type checking
  ```bash
  uv run ty check -- nextcloud_mcp_server
  ```

### Code Quality
- **Before committing or pushing, invoke the `pre-push-review` skill**:
  - Runs `ruff check`, `ruff format --check`, `ty check`, and unit tests, then audits the
    branch diff against this repo's recurring PR-review patterns (mined from PRs #733–#750).
  - Output is a labelled punch list (🔴 blocking / 🟡 important / 🟢 nit). The main loop
    fixes; the skill reports.
  - Skill location: `.claude/skills/pre-push-review/SKILL.md`. Invoke via the `Skill` tool
    with `skill="pre-push-review"`, or when the user types `/pre-push-review`.
  - Skip only for tiny diffs (typo, README tweak, single-line dependency bump) or when the
    user explicitly says "just push it".
- **Manual fallback** (if the skill is unavailable):
  ```bash
  uv run ruff check
  uv run ruff format
  uv run ty check -- nextcloud_mcp_server
  uv run pytest tests/unit/ -x -q
  ```
- **Ruff configuration** in pyproject.toml (extends select: ["I"] for import sorting)

### Error Handling
- **Use custom decorators**: `@retry_on_429` for rate limiting (see base_client.py)
- **Standard exceptions**: `HTTPStatusError` from httpx, `McpError` for MCP-specific errors
- **Logging patterns**:
  - `logger.debug()` for expected 404s and normal operations
  - `logger.warning()` for retries and non-critical issues
  - `logger.error()` for actual errors

### Testing Patterns
- **Use existing fixtures** from `tests/conftest.py` (2888 lines of test infrastructure)
- **Session-scoped fixtures** handle anyio/pytest-asyncio incompatibility
- **Mocked unit tests** use `mocker.AsyncMock(spec=httpx.AsyncClient)`
- **pytest-timeout**: 180s default per test
- **Mark tests appropriately**: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.oauth`, `@pytest.mark.smoke`

### Architectural Patterns
- **Base classes**: `BaseNextcloudClient` for all API clients
- **Pydantic responses**: All MCP tools return Pydantic models inheriting from `BaseResponse`
- **Decorators**: `@require_scopes`, `@require_provisioning` for access control
- **Context pattern**: `await get_client(ctx)` to access authenticated NextcloudClient (async!)
- **FastMCP decorators**: `@mcp.tool()`, `@mcp.resource()`
- **Token acquisition**: `get_client()` resolves credentials per deployment mode (see Deployment Modes below)

### MCP Tool Annotations (ADR-017)

**All tools MUST include annotations** following these patterns:

```python
from mcp.types import ToolAnnotations

# Read-only tools (list, search, get)
@mcp.tool(
    title="Human Readable Name",
    annotations=ToolAnnotations(
        readOnlyHint=True,
        openWorldHint=True,  # Nextcloud is external to MCP server
    ),
)

# Create operations
@mcp.tool(
    title="Create Resource",
    annotations=ToolAnnotations(
        idempotentHint=False,  # Creates new resources each time
        openWorldHint=True,
    ),
)

# Update operations (with etag/version control)
@mcp.tool(
    title="Update Resource",
    annotations=ToolAnnotations(
        idempotentHint=False,  # ETag changes = different inputs
        openWorldHint=True,
    ),
)

# Delete operations
@mcp.tool(
    title="Delete Resource",
    annotations=ToolAnnotations(
        destructiveHint=True,   # Permanently deletes data
        idempotentHint=True,    # Same end state if called repeatedly
        openWorldHint=True,
    ),
)

# HTTP PUT without version control (special case)
@mcp.tool(
    title="Write File",
    annotations=ToolAnnotations(
        idempotentHint=True,  # Same content = same end state
        openWorldHint=True,
    ),
)
```

**Key Principles**:
- **Idempotency**: Same inputs → same result. ETags change after updates, making them non-idempotent
- **Destructive**: Operations that permanently delete/overwrite data
- **Open World**: All Nextcloud tools access external service (openWorldHint=True)
- **Titles**: Use human-readable names, not snake_case function names

**See**: `docs/ADR-017-mcp-tool-annotations.md` for detailed rationale and examples

### Project Structure
- `nextcloud_mcp_server/client/` - HTTP clients for Nextcloud APIs
- `nextcloud_mcp_server/server/` - MCP tool/resource definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cbcoutinho/nextcloud-mcp-server](https://github.com/cbcoutinho/nextcloud-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
