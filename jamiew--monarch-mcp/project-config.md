---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## PII & Data Privacy

**CRITICAL**: Never commit or include personally identifiable financial data in code, docs, tests, or commit messages. This includes:
- Real account names (e.g., specific credit card names like "Main Credit Card")
- Real merchant names from the user's transaction history
- Real transaction IDs, category IDs, or account IDs from Monarch Money
- Real dollar amounts tied to specific transactions
- Any data that could identify the user's financial institutions or spending habits

Use generic, obviously-fake examples instead: "Main Credit Card", "Corner Deli", "cat_001", "txn_123". Brand names like "Starbucks" are fine as generic illustrative examples in docstrings — the distinction is between "examples of merchants" vs "data from the user's actual account."

## Development Commands

### Basic Operations
- `uv sync` - Install dependencies and create/update virtual environment
- `uv run python server.py` - Run the MCP server directly for testing
- `uv add <package>` - Add new dependencies to the project
- `uv remove <package>` - Remove dependencies from the project

### Testing & Validation
- `uv run pytest tests/ -v --tb=short` - Run all tests
- `uv run mypy server.py` - Type checking
- `uv run ruff check .` - Lint
- `uv run ruff format --check .` - Format check (use `ruff format .` to auto-fix)
- `uv run python server.py` - Test server directly (all logs to stderr)
- `MONARCH_FORCE_LOGIN=true uv run python server.py` - Force fresh login (if session expires)

### Debugging Startup Issues (Updated July 2025)
- **Session expired**: Delete `~/.monarch-mcp/session.pickle` or set `MONARCH_FORCE_LOGIN=true`
- **JSON parse errors**: Fixed - all stdout output suppressed with `contextlib.redirect_stdout()`
- **MCP protocol compliance**: All logging/warnings redirected to stderr, third-party lib output suppressed
- **AsyncIO errors**: Fixed - uses `run_stdio_async()` in async context
- **SSL warnings**: Suppressed from gql.transport.aiohttp to prevent stdout contamination
- **Date serialization errors**: Fixed - `build_date_filter()` returns ISO strings for JSON safety
- **Broken pipe errors**: Fixed - comprehensive graceful shutdown and error recovery implemented
- **Date parsing failures**: Enhanced with multi-format fallbacks and helpful error messages

### Usage Analytics & Optimization Monitoring

**View usage analytics in Claude's MCP log:**
```bash
# Monitor all analytics (tool calls, performance, errors)
tail -f /Users/jamie/Library/Logs/Claude/mcp-server-monarch-money.log | grep "\[ANALYTICS\]"

# Watch for optimization suggestions
tail -f /Users/jamie/Library/Logs/Claude/mcp-server-monarch-money.log | grep "\[OPTIMIZATION\]"

# Monitor performance (slow operations > 1 second)
tail -f /Users/jamie/Library/Logs/Claude/mcp-server-monarch-money.log | grep "\[ANALYTICS\]" | grep -E "time: [1-9][0-9]*\.[0-9]+s"

# View session summaries and top tools
tail -f /Users/jamie/Library/Logs/Claude/mcp-server-monarch-money.log | grep "session_summary"

# NEW: Debug tool calls with arguments (for optimization)
tail -f /Users/jamie/Library/Logs/Claude/mcp-server-monarch-money.log | grep "\[TOOL_CALL\]"

# NEW: Monitor result sizes for context usage optimization
tail -f /Users/jamie/Library/Logs/Claude/mcp-server-monarch-money.log | grep "\[RESULT_SIZE\]"

# NEW: Watch for large results (> 50KB) that may need optimization
tail -f /Users/jamie/Library/Logs/Claude/mcp-server-monarch-money.log | grep "\[RESULT_SIZE\]" | grep -E "[5-9][0-9]\.[0-9]+ KB|[0-9]{3,}\.[0-9]+ KB"
```

**Log Format Examples:**
- `[TOOL_CALL] get_transactions | args: {'limit': 100, 'start_date': 'last month', 'verbose': False}`
- `[ANALYTICS] tool_called: get_transactions | time: 0.234s | status: success`
- `[RESULT_SIZE] get_transactions | chars: 12,543 | size: 12.25 KB | transactions: 42 items`
- `[OPTIMIZATION] Consider using get_complete_financial_overview instead of separate get_accounts + get_transactions calls`
- `[ANALYTICS] session_summary: 15 calls | top_tool: get_transactions`

## Development Workflow & Git Guidelines

### Automated Development Process
**When no specific instructions are provided, follow this workflow:**

1. **Read Current Status**: Always start by reading the latest TODO items and status in this CLAUDE.md file
2. **Select Next Task**: Choose the highest priority pending task from the current status section
3. **Implement & Test**: Work on the task following the quality standards below
4. **Validate Before Commit**: Always run type checks and tests before committing
5. **Commit Each Feature**: Make atomic commits for individual features, fixes, or optimizations
6. **Update Status**: Periodically update CLAUDE.md status section (not every commit)

### Git Commit Standards

**Pre-push check (mirrors CI):**
```bash
uv run python scripts/ci.py
```

This runs ruff check, ruff format, mypy, and pytest — the same checks as `.github/workflows/ci.yml`. CI runs these on Python 3.10–3.13 against every PR to main.

**Commit Message Format:**
```
<type>: <concise description>

<optional body explaining why/what changed>
```

**Commit Types:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamiew/monarch-mcp](https://github.com/jamiew/monarch-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
