---
trigger: always_on
description: This file provides guidance for AI coding agents working with the clippy-code codebase.
---

# AGENTS.md

This file provides guidance for AI coding agents working with the clippy-code codebase.

## Essential Commands

```bash
make dev              # Install with dev dependencies
make test             # Run pytest
make check            # Run format, lint, and type-check
make run              # Launch interactive mode through the Makefile
make format           # Autofix and format code with ruff
make lint             # Static analysis with ruff
make type-check       # Run mypy against src/clippy
make run              # Launch clippy-code in interactive mode through Makefile
uv run python -m clippy       # Run in interactive mode
```

## Development Workflow Tips

- Prefer the `make` targets above for consistent formatting, linting, and type checks.
- Run `make check` and `make test` before finishing a task to catch regressions early.
- Use `make format` if a change requires ruff autofixes prior to committing or submitting.
- Reference `README.md` for installation guidance and `CONTRIBUTING.md` for contributor workflow details.
### Commit Message Standards

- **Always use conventional commits**: Follow the `type(scope): description` format for consistency and tooling compatibility
- **Use appropriate scopes**: Common scopes include `agent`, `cli`, `tools`, `mcp`, `subagent`, `docs`, `test`, `fix`, `refactor`
- **Types**: Use `feat` for new features, `fix` for bug fixes, `docs` for documentation, `refactor` for code refactoring, `test` for tests, `chore` for maintenance
- **Examples**:
  - `feat(subagent): add grepper specialized information gathering agent`
  - `fix(tools): resolve file encoding issue in read_file tool`
  - `docs(agent): update subagent configuration examples`
  - `refactor(cli): simplify command parsing logic`

## Agent Behavior Guidelines

When working as an AI coding agent with clippy-code, follow these behavior guidelines:

### Scope Management

- **Stay within user requests**: Only perform tasks explicitly requested by the user. Don't add features, refactor code, or make improvements that weren't asked for.
- **Ask for clarification**: If a request is ambiguous or incomplete, ask clarifying questions before proceeding.
- **Minimal changes**: Make only the changes necessary to complete the requested task. Avoid unnecessary refactoring or "improvements" unless specifically requested.
- **Respect project structure**: Don't reorganize files or directories unless explicitly asked to do so.

### Tool Usage

- **Read before editing**: Always use `read_file` to examine files before modifying them.
- **Use appropriate tools**: Choose the right tool for the job (e.g., `edit_file` for precise edits, `find_replace` for multi-file changes).
- **Test changes**: After making changes, verify they work correctly by running tests or checking the modified code.
- **Be cautious with destructive operations**: Use `execute_command`, `delete_file`, and other potentially destructive tools with care.

### Communication

- **Explain reasoning**: Before taking significant actions, explain your approach and reasoning to the user.
- **Provide progress updates**: When working on complex tasks, provide regular updates on progress.
- **Handle errors gracefully**: If something goes wrong, explain what happened and suggest alternatives.


## Project Structure

```
src/clippy/
├── agent/
│   ├── core.py                 # Core agent implementation
│   ├── loop.py                 # Agent loop logic
│   ├── conversation.py         # Conversation utilities
│   ├── tool_handler.py         # Tool calling handler
│   ├── subagent.py             # Subagent implementation
│   ├── subagent_manager.py     # Subagent lifecycle management
│   ├── subagent_types.py       # Subagent type configurations
│   ├── subagent_cache.py       # Result caching system
│   ├── subagent_chainer.py     # Hierarchical execution chaining
│   ├── subagent_config_manager.py # Subagent configuration management
│   ├── utils.py                # Agent helper utilities
│   └── errors.py               # Agent-specific exceptions
├── cli/
│   ├── completion.py           # Command completion utilities
│   ├── commands.py             # High-level CLI commands
│   ├── main.py                 # Main entry point
│   ├── oneshot.py              # One-shot mode implementation
│   ├── parser.py               # Argument parsing
│   ├── repl.py                 # Interactive REPL mode
│   └── setup.py                # Initial setup helpers
├── tools/
│   ├── __init__.py             # Tool registrations
│   ├── catalog.py              # Tool catalog for built-in and MCP tools
│   ├── create_directory.py
│   ├── delete_file.py
│   ├── delegate_to_subagent.py
│   ├── edit_file.py
│   ├── execute_command.py
│   ├── get_file_info.py
│   ├── grep.py
│   ├── list_directory.py
│   ├── read_file.py
│   ├── read_files.py
│   ├── run_parallel_subagents.py
│   ├── search_files.py
│   └── write_file.py
├── mcp/
│   ├── config.py               # MCP configuration loading
│   ├── errors.py               # MCP error handling
│   ├── manager.py              # MCP server connection manager
│   ├── naming.py               # MCP tool naming utilities
│   ├── schema.py               # MCP schema conversion

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thomwebb/clippy-code](https://github.com/thomwebb/clippy-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
