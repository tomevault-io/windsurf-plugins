---
trigger: always_on
description: This is a Model Context Protocol (MCP) server that provides tools for querying and analyzing data from the Konflux DevLake database. The server exposes tools for database operations, incident analysis, deployment tracking, and PR retest analysis.
---

# Konflux DevLake MCP Server - Cursor Rules

## Project Overview
This is a Model Context Protocol (MCP) server that provides tools for querying and analyzing data from the Konflux DevLake database. The server exposes tools for database operations, incident analysis, deployment tracking, and PR retest analysis.

## Code Style & Formatting

### Python Code Style
- **Formatter**: Use `black` with line length 100
  - Command: `black --line-length 100 .`
  - Always format code before committing
- **Linter**: Use `flake8` for code quality
  - Command: `flake8 .`
  - Fix all linting errors before committing
- **Type Hints**: Use type hints for all function parameters and return types
- **Docstrings**: Use Google-style docstrings for all classes and functions

### YAML Code Style
- **Linter**: Use `yamllint` for YAML file validation
  - Command: `yamllint .`
  - Configuration file: `.yamllint` (extends default, max line length 200)
  - Fix all linting errors before committing
  - Applies to: `.github/workflows/*.yaml`, `k8s/*.yaml`, and all other YAML files

### Code Formatting Rules
- Maximum line length: 100 characters
- Use double quotes for strings (black default)
- Use trailing commas in multi-line collections
- Use f-strings for string formatting (not `.format()` or `%`)
- Use `async`/`await` for all database operations and tool calls
- **NO EMOJIS**: Do not use any emojis in code, comments, docstrings, or any project files

## Architecture Patterns

### Tool Creation Pattern
All tools must:
1. Inherit from `BaseTool` in `tools/base/base_tool.py`
2. Implement `get_tools()` method returning a list of `Tool` objects
3. Implement `call_tool(name, arguments)` method for async tool execution
4. Use `log_tool_call()` for all tool invocations
5. Return TOON-encoded strings for token efficiency (not JSON)

### Tool Structure
```python
class MyTool(BaseTool):
    def __init__(self, db_connection):
        super().__init__(db_connection)
        self.logger = get_logger(f"{__name__}.MyTool")
    
    def get_tools(self) -> List[Tool]:
        return [Tool(name="...", description="...", inputSchema={...})]
    
    async def call_tool(self, name: str, arguments: Dict[str, Any]) -> str:
        # Use toon_encode for output
        return toon_encode(result, {"delimiter": ",", "indent": 2, "lengthMarker": ""})
```

### Database Query Patterns
- Always use parameterized queries or string formatting with validation
- Use `await self.db_connection.execute_query(query, limit)`
- Handle MySQL type conversions (Decimal/string to int/float)
- Always validate date fields before using in queries
- Use CTEs (WITH clauses) for complex queries with deduplication

### Error Handling
- Always wrap tool execution in try-except blocks
- Log errors with `self.logger.error()` including context
- Return structured error responses: `{"success": False, "error": str(e)}`
- Use `log_tool_call(..., success=False, error=...)` for failed calls
- Handle `ClosedResourceError` and `CancelledError` gracefully (don't log as errors)

### Logging
- Use `get_logger(__name__)` for module-level loggers
- Use appropriate log levels:
  - `DEBUG`: Detailed diagnostic information
  - `INFO`: General informational messages
  - `WARNING`: Warning messages
  - `ERROR`: Error conditions
- Never log sensitive information (passwords, tokens, etc.)
- Suppress noisy library errors (e.g., ClosedResourceError from MCP library)

## File Organization

### Directory Structure
```
tools/
  base/           # Base tool classes
  devlake/        # DevLake-specific tools (incidents, deployments, PR retests)
  database_tools.py  # Database operation tools
  tools_manager.py   # Tool registry and routing

server/
  core/           # Core MCP server implementation
  factory/        # Server factory for different transports
  handlers/       # Request handlers
  transport/      # Transport implementations (HTTP, stdio)

utils/
  config.py       # Configuration management
  db.py           # Database connection and utilities
  logger.py       # Logging setup
  security.py     # Security validation
```

### Naming Conventions
- Files: `snake_case.py`
- Classes: `PascalCase`
- Functions/methods: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Private methods: `_leading_underscore`

## Testing Requirements

### Unit Tests
- All tools must have unit tests in `tests/unit/`
- Test files: `test_<module_name>.py`
- Use `pytest` with `pytest-asyncio` for async tests
- Mock database connections in tests
- Test both success and error cases

### Running Tests
- Unit tests: `pytest tests/unit/ -v`
- All tests: `pytest`
- Use `run_tests.py` script for convenience

## Database Patterns

### DevLake Database
- Main database: `lake`
- Key tables:
  - `incidents`: Incident tracking
  - `cicd_deployments`, `cicd_deployment_commits`: Deployment data
  - `pull_requests`, `pull_request_comments`: PR data
  - `repos`: Repository information
  - `project_mapping`: Project-to-resource mapping

### Query Best Practices
- Always join with `project_mapping` when filtering by project

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [konflux-ci/konflux-devlake-mcp](https://github.com/konflux-ci/konflux-devlake-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
