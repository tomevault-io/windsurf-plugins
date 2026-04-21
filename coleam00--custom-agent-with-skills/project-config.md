---
trigger: always_on
description: Framework-agnostic skill system for AI agents that extracts the progressive disclosure pattern from Claude Skills and implements it as a native feature in Pydantic AI. This system enables developers to build AI agents with modular, reusable skills that load instructions and resources on-demand, eliminating context window constraints while maintaining type safety and testability.
---

# Custom Skill-Based Pydantic AI Agent Development Instructions

## Project Overview

Framework-agnostic skill system for AI agents that extracts the progressive disclosure pattern from Claude Skills and implements it as a native feature in Pydantic AI. This system enables developers to build AI agents with modular, reusable skills that load instructions and resources on-demand, eliminating context window constraints while maintaining type safety and testability.

**Key Innovation**: Unlike Claude Skills which are locked to the Claude ecosystem, this implementation makes advanced skill capabilities available to ANY AI agent framework. Demonstrates how to take successful patterns from proprietary systems and apply them to open frameworks.

## Core Principles

1. **FRAMEWORK AGNOSTIC**
   - Skills work with any AI framework, not just Claude
   - No vendor lock-in to Claude Desktop/Code
   - Portable skills across different agent implementations

2. **PROGRESSIVE DISCLOSURE IS KEY**
   - Level 1: Metadata in system prompt (~100 tokens/skill)
   - Level 2: Full instructions loaded via tool call
   - Level 3: Resources loaded via tool call only when referenced
   - Never consume unnecessary context

3. **TYPE SAFETY IS NON-NEGOTIABLE**
   - All functions, methods, and variables MUST have type annotations
   - Use Pydantic models for all data structures (skills, metadata)
   - No `Any` types without explicit justification
   - Pydantic Settings for configuration

4. **KISS** (Keep It Simple, Stupid)
   - Prefer simple, readable solutions over clever abstractions
   - Don't build fallback mechanisms unless absolutely necessary
   - Trust the progressive disclosure pattern - don't over-engineer

5. **YAGNI** (You Aren't Gonna Need It)
   - Don't build features until they're actually needed
   - MVP first, enhancements later
   - Focus on demonstrating the core pattern

6. **REFERENCE THE EXAMPLES FOLDER**
   - `examples/` contains production-quality MongoDB RAG agent
   - **DO NOT MODIFY** files in `examples/` - they are reference only
   - Copy patterns and code from examples, adapt for skills
   - Maintain same architecture and design patterns

**Architecture:**

```
custom-skill-agent/
├── src/                      # Skill-based agent implementation
│   ├── agent.py              # Pydantic AI agent with skills
│   ├── skill_loader.py       # Skill discovery and metadata
│   ├── skill_tools.py        # Progressive disclosure tools
│   ├── dependencies.py       # AgentDependencies with SkillLoader
│   ├── providers.py          # LLM provider configuration
│   ├── settings.py           # Pydantic Settings
│   ├── prompts.py            # Skill-aware system prompts
│   └── cli.py                # Rich-based CLI
├── skills/                   # Skill library
│   ├── weather/              # Simple skill demo
│   └── code_review/          # Advanced skill demo
├── examples/                 # MongoDB RAG reference (DO NOT MODIFY)
└── tests/                    # Test suite
```

---

## Documentation Style

**Use Google-style docstrings** for all functions, classes, and modules:

```python
async def load_skill(
    ctx: RunContext[AgentDependencies],
    skill_name: str
) -> str:
    """
    Load the full instructions for a skill (Level 2 progressive disclosure).

    Args:
        ctx: Agent runtime context with dependencies
        skill_name: Name of the skill to load

    Returns:
        Full skill instructions from SKILL.md

    Raises:
        ValueError: If skill not found
    """
```

---

## Development Workflow

**Setup environment:**
```bash
# Create virtual environment
python -m venv .venv

# Activate environment
source .venv/bin/activate  # Unix
.venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt
# Or if using uv:
uv pip install -e .
```

**Run CLI agent:**
```bash
python -m src.cli
```

**Run tests:**
```bash
pytest tests/ -v
```

---

## Configuration Management

### Environment Variables

**ALL configuration in .env file:**
```bash
# LLM Provider
LLM_PROVIDER=openrouter
LLM_API_KEY=sk-or-v1-...
LLM_MODEL=anthropic/claude-sonnet-4.5
LLM_BASE_URL=https://openrouter.ai/api/v1

# Skills Configuration
SKILLS_DIR=skills

# Application Settings
APP_ENV=development
LOG_LEVEL=INFO
```

### Pydantic Settings

**Use Pydantic Settings for type-safe configuration:**
```python
from pydantic_settings import BaseSettings
from pydantic import Field, ConfigDict
from pathlib import Path

class Settings(BaseSettings):
    """Application settings with environment variable support."""

    model_config = ConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False
    )

    skills_dir: Path = Field(default=Path("skills"))
    llm_api_key: str = Field(..., description="LLM provider API key")
    llm_model: str = Field(default="anthropic/claude-sonnet-4.5")
```

---

## Progressive Disclosure Pattern

### The Three Levels


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coleam00/custom-agent-with-skills](https://github.com/coleam00/custom-agent-with-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
