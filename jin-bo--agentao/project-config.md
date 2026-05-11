---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Management

**Always use `uv` for package management**, not pip:

```bash
# Install dependencies
uv sync

# Add a new dependency
uv add package-name

# Run Python scripts
uv run python script.py

# Run the CLI
uv run agentao
# or
uv run python main.py
```

## Running and Testing

### Start the Agent

```bash
# Quick start
./run.sh

# Or directly
uv run agentao

# Or via Python
uv run python main.py
```

### Run Tests

```bash
# Run all tests with pytest
uv run python -m pytest tests/

# Run a specific test file
uv run python tests/test_imports.py
uv run python tests/test_tool_confirmation.py
uv run python tests/test_readchar_confirmation.py
uv run python tests/test_date_in_prompt.py

# All test files are in tests/ directory
```

### Configuration

Copy and edit `.env` from `.env.example`:
```bash
cp .env.example .env
# Edit .env with your API key and settings
```

> **Reference for all config files** (`.env`, `.agentao/settings.json`, `permissions.json`, `mcp.json`, `acp.json`, `skills_config.json`, `AGENTAO.md`, memory DBs): see [docs/CONFIGURATION.md](docs/CONFIGURATION.md) for paths, schema, defaults, and precedence rules.

Required: `OPENAI_API_KEY`
Optional: `OPENAI_BASE_URL`, `OPENAI_MODEL`

## Architecture

### Three-Layer Design

Agentao uses a **Tool-Agent-CLI** architecture:

1. **CLI Layer** (`cli.py`): User interface with Rich, handles commands, manages session state (like `allow_all_tools`)
2. **Agent Layer** (`agent.py`): Orchestrates LLM, tools, skills, and conversation history
3. **Tool Layer** (`tools/`): Individual tool implementations following the Tool base class

```
User → CLI → Agent → LLM + Tools
                  ↓
            SkillManager (loads from skills/)
```

### Tool System

All tools inherit from `Tool` base class (`tools/base.py`):

```python
class MyTool(Tool):
    @property
    def name(self) -> str:
        return "my_tool"

    @property
    def description(self) -> str:
        return "Description for LLM"

    @property
    def parameters(self) -> Dict[str, Any]:
        return {...}  # JSON Schema

    @property
    def requires_confirmation(self) -> bool:
        return False  # True for dangerous operations

    def execute(self, **kwargs) -> str:
        return "Result"
```

**Tool Registration**: Tools are registered in `agent.py::_register_tools()`. The `ToolRegistry` converts them to OpenAI function calling format.

**Tool Confirmation**: Tools with `requires_confirmation=True` (Shell, Web, File Writing) pause execution and prompt user via `confirmation_callback` passed from CLI.

**Tools requiring confirmation:**
- `run_shell_command` - Shell command execution (allowlist for safe read-only commands)
- `web_fetch` - Fetch web content (domain-tiered: allowlist/blocklist/ask)
- `web_search` - Web search
- `write_file` - File writing/overwriting (prevents data loss)

**Domain-Based Permissions** (`web_fetch`): The `PermissionEngine` supports `"domain"` rules with allowlist/blocklist matching. Default presets auto-allow trusted docs sites (`.github.com`, `.docs.python.org`, etc.) and auto-deny SSRF targets (`localhost`, `127.0.0.1`, `169.254.169.254`, etc.). Customizable via `.agentao/permissions.json`. See `docs/features/TOOL_CONFIRMATION_FEATURE.md` for details.

### Skills System

**Dynamic Loading**: Skills are auto-discovered from `skills/` directory. Each subdirectory contains:
- `SKILL.md` - Main file with YAML frontmatter (`name:`, `description:`)
- `reference/*.md` (optional) - Additional documentation loaded on-demand

**Skill Manager** (`skills/manager.py`):
- Parses YAML frontmatter from SKILL.md files
- Maintains `available_skills` dict (all skills)
- Maintains `active_skills` dict (currently activated)
- Injects active skill context into system prompt

**Activation**: Use `activate_skill` tool or `/skills` command. Active skills add their documentation to the system prompt.

### System Prompt Composition

The system prompt is dynamically built in `agent.py::_build_system_prompt()`:

1. **AGENTAO.md** (if exists in cwd) - Project-specific instructions
2. **Agent Instructions** - Base Agentao capabilities
3. **Current Date/Time** - Auto-injected: `YYYY-MM-DD HH:MM:SS (Day)`
4. **Available Skills** - List with descriptions
5. **Active Skills Context** - Full documentation of activated skills

This composition happens on every `chat()` call to keep skills context fresh.

### Conversation Flow

```python
# agent.py::chat()
1. User message added to self.messages
2. System prompt built (includes AGENTAO.md, date, skills)
3. LLM called with messages + tools
4. Loop (max 100 iterations):
   a. If tool_calls: execute each tool
      - Check requires_confirmation
      - Call confirmation_callback if needed
      - Execute tool or cancel based on response
   b. Add tool results to messages
   c. Call LLM again with updated messages
   d. If no tool_calls: return final response
```

### Logging System

**Complete LLM interaction logging** to `agentao.log`:
- Every request/response (full content, no truncation)
- All tool calls with formatted JSON arguments
- Tool results
- Token usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jin-bo/agentao](https://github.com/jin-bo/agentao) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
