---
trigger: always_on
description: **Goal**: Maintain code quality, consistency, and documentation across rossum-agents packages (rossum-mcp, rossum-agent, rossum-agent-client).
---

# Development Guidelines

**Goal**: Maintain code quality, consistency, and documentation across rossum-agents packages (rossum-mcp, rossum-agent, rossum-agent-client).

## Critical Constraints

- **No auto-commits** - Only `git commit`/`git push` when explicitly instructed
- **Simplicity first** - Design the simplest solution that works. Fewer abstractions, fewer indirections, fewer layers. If a reader needs to jump through hoops to understand the code, it's too complex.
- **YAGNI** - Don't add functionality until needed. Remove unused code proactively.
- **Tests required** - New features and bug fixes must include tests
- **Docs in sync** - Tool changes require documentation updates
- **Changelog PR references** - When adding changelog entries, include the PR hyperlink (e.g. `[#123](https://github.com/rossumai/rossum-agents/pull/123)`) if available

## Commands

| Task | Command |
|------|---------|
| Setup | `uv sync` or `uv pip install -e .` |
| Server | `rossum-mcp` (installed) or `python -m rossum_mcp.server` (dev) |
| Tests | `pytest` or `pytest path/to/test.py` |
| Lint | `pre-commit run --all-files` |
| TUI lint | `cd rossum-agent-tui && npm run lint && npm run format:check && npm run typecheck` |

## Architecture

- **rossum-mcp**: FastMCP server in `rossum_mcp/server.py`; tools registered from `rossum_mcp/tools/` modules
- **rossum-agent**: AI agent with system prompt in `rossum_agent/system_prompt.py`, skills in `rossum_agent/skills/`
- **rossum-agent-tui**: Development test-bed TUI for rossum-agent. Not production code — no tests required. Always use `/ink-tui-best-practices` skill when working on this package.
- **New skills**: Create `rossum_agent/skills/<slug>.md` — auto-discovered by `SkillRegistry`

## Prompt Engineering (rossum-agent)

**rossum-agent uses Opus 4.6** - optimize prompts in `rossum_agent/system_prompt.py` and `rossum_agent/skills/` accordingly:

| Principle | Implementation |
|-----------|----------------|
| Goals over procedures | "Goal: Deploy safely" not step-by-step instructions |
| Constraints over explanations | "Never mix credentials" - Opus infers consequences |
| Tables for structure | More token-efficient than prose lists |
| No redundancy | Don't explain what Opus can infer |
| Facts not warnings | State rules directly, skip "IMPORTANT" preambles |

## Code Style

| Rule | Example |
|------|---------|
| Python 3.12+ | Modern syntax required |
| Type hints | `str \| None` not `Optional[str]`, `list[str]` not `List[str]` |
| No `Any` | Use specific types |
| Imports | Standard library first, `from pathlib import Path` |
| No lazy imports | All imports at module level. No `import` inside functions/methods. |
| Comments | Explain why, not what |
| No trailing commas | Follow ruff-format output |
| Logging | f-strings in `logger.*()` calls are fine — prefer `logger.info(f"...")` over `%s` style |
| Noqa comments | Always explain: `# noqa: TC003 - reason` |
| No hardcoded `/tmp` | Use `tempfile` module or `/mock/...` paths in tests — CodeFactor runs Bandit (B108) |

## FastMCP Tools (rossum-mcp)

**Constraint**: Don't duplicate info between `description` and docstring.

```python
@mcp.tool(description="List users. Filter by username/email. Returns URLs usable as token_owner.")
async def list_users(
    username: str | None = None,
    email: str | None = None,
) -> list[User]:
    # No docstring - description + type hints sufficient
    ...
```

Add docstring only when: non-obvious formats, complex filtering, unclear defaults.

Import return types at module level (not TYPE_CHECKING) for FastMCP serialization.

### Adding New MCP Tools

| Step | Action |
|------|--------|
| Install latest SDK | Run `uv add rossum-api@latest` to get the newest `rossum-api` package |
| Leverage SDK | Check if `rossum-api` already provides models, dataclasses, type literals, or helper methods for the feature before writing custom code |
| Private SDK access | Usage of private/internal APIs in `rossum-api` is allowed — we control both packages |
| Use typed constructs | Prefer dataclasses, `Literal` types, enums, and typed models from `rossum-api` over plain strings or untyped dicts |
| API docs fallback | If the SDK doesn't cover the needed functionality, consult https://rossum.app/api/docs for the raw API spec |

### Adding New Rossum Capabilities (formula fields, reasoning fields, etc.)

When implementing support for Rossum-specific features, research them first:

| Source | URL | Purpose |
|--------|-----|---------|
| Knowledge Base | https://knowledge-base.rossum.ai/ | Feature concepts, configuration, and usage guides |
| API Docs | https://rossum.app/api/docs | API endpoints, request/response schemas |

## Documentation Updates

When adding/modifying tools, update:

| Tool Type | Files to Update |
|-----------|-----------------|
| MCP tools | `rossum-mcp/README.md`, `docs/source/rossum-mcp/tools.rst`, `docs/source/rossum-mcp/configuration.rst` |
| Agent tools | `rossum-agent/README.md`, `docs/source/rossum-agent/tools.rst`, `docs/source/rossum-agent/skills.rst`, `docs/source/rossum-agent/subagents.rst` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rossumai/rossum-agents](https://github.com/rossumai/rossum-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
