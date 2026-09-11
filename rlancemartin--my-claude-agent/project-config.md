---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open Claude Agent is a minimal, open-source agent harness that reproduces key features from Anthropic's Claude Agent SDK. It implements a simple agent framework using Claude's native tool support (filesystem, bash, web search) combined with context management features.

## Development Setup

```bash
# Install dependencies using uv (required)
uv sync

# Install package in editable mode
uv pip install -e .

# Set API key
export ANTHROPIC_API_KEY=your_key_here

# Run the agent (typically in Jupyter notebook)
# See sandbox/run_agent.ipynb for examples
```

## Architecture

### Core Components

**ClaudeAgent** (`src/open_claude_agent/claude_agent.py`):
- Main agent class orchestrating tool usage and API interactions
- Automatically configures 4 built-in tools: memory, bash, text_editor, web_search
- Implements conversation loop with tool use iterations (max 15 turns by default)
- Uses context management via Claude's context editing feature (clear_tool_uses_20250919)
- System messages automatically composed: `GENERAL_TOOL_USAGE_GUIDELINES + skills_metadata + custom_message`
- Three response handling paths in the loop:
  1. Server-side tools only: Display results, continue loop
  2. Client-side tools only: Execute locally, add results to messages, continue loop
  3. Mixed server/client tools: Handle both in sequence
  4. No tools: Extract final response, display, exit loop

**Tool Handlers** (`src/open_claude_agent/tools/`):
- Each handler implements `handle(tool_input)` method returning result dict
- `MemoryToolHandler`: Operates exclusively in `./memories` directory (hardcoded, auto-created)
- `BashToolHandler`: Persistent session, command validation, timeout protection, audit logging
- `TextEditorToolHandler`: Automatic backups before modifications, path validation
- Web search is server-side (no local handler needed)

**Utilities** (`src/open_claude_agent/utils.py`):
- Rich console formatting using panels and styled text
- `format_anthropic_tool_call()`: Displays both client-side (tool_use) and server-side (server_tool_use) tool calls
  - Special visualization for skill loading: detects when bash commands are reading SKILL.md files
  - Shows "📚 Loading Skill: skill-name" with blue border instead of regular bash formatting
- `format_anthropic_tool_result()`: Shows success/error states with color coding
  - Detects skill loading results and displays "✓ Skill Loaded: skill-name" with blue styling
- `detect_skill_usage()`: Helper function to detect when bash commands are loading skills
- `display_claude_response()`: Final response display

**Prompts** (`src/open_claude_agent/prompts.py`):
- `GENERAL_TOOL_USAGE_GUIDELINES`: Core instructions automatically included in every system message

Note: Research guidance is now available through the `web-research` skill (bundled at `src/open_claude_agent/skills/web-research/SKILL.md`).

### Context Engineering

Three context management principles:

1. **Context Reduction**: Automatically clears oldest tool results when approaching token limits
   - Trigger: 100k input tokens
   - Keep: 5 most recent tool uses
   - Cleared results replaced with placeholders

2. **Context Offloading**: Memory tool saves important information before context is cleared
   - Claude receives warning when approaching clearing threshold
   - Encouraged to preserve key information to `./memories` directory

3. **Context Isolation**: Placeholder - not yet implemented

### Skills System

The agent supports modular skills that package domain-specific expertise and tools. Skills follow Anthropic's progressive disclosure architecture for efficient context management.

**Skill Loader** (`src/open_claude_agent/skill_loader.py`):
- `SkillLoader` class scans skills directory for subdirectories containing `SKILL.md`
- Parses YAML frontmatter to extract `name` and `description` metadata
- Formats metadata into system message section (Level 1: Progressive Disclosure)
- Convenience function `load_skills(skills_dir)` returns formatted context string

**Skills Directory Structure** (bundled at `src/open_claude_agent/skills/`):
```
src/open_claude_agent/skills/
└── skill-name/
    ├── SKILL.md         # Required: YAML frontmatter + documentation
    ├── reference.md     # Optional: additional docs
    └── script.py        # Optional: executable code
```

**Progressive Disclosure (3 Levels)**:
1. **Metadata**: Skill name and description loaded into system prompt at initialization
2. **Core Content**: Full `SKILL.md` read via bash tool when skill is relevant
3. **Resources**: Additional files and scripts accessed only as needed

**Integration with ClaudeAgent**:
- New parameters:
  - `skills_dir` (default: `None` - auto-resolves to `src/open_claude_agent/skills/`)
  - `enable_skills` (default: `True`)
- Path resolution: When `skills_dir=None`, automatically resolves to the bundled skills directory at `src/open_claude_agent/skills/`
- System message construction: `GENERAL_TOOL_USAGE_GUIDELINES + skills_metadata + custom_message`
- Skills loaded once during `__init__`, not on every `call()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rlancemartin/my-claude-agent](https://github.com/rlancemartin/my-claude-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
