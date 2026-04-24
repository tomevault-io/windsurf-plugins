---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

see [AGENTS.md](AGENTS.md)

## Project Overview

This is a **Multi-Agent Lab** project built with **LangChain + Ollama**. The project is structured as a learning-focused framework for building production-ready multi-agent systems with multiple business domains.

**Architecture**: Domain-driven multi-agent system with hierarchical agent structure
**Execution Patterns**:
- **Static Execution Plan**: Predefined sequential workflow using Skill Cards
- **Dynamic Agent**: LLM selects tools contextually based on user queries

**Current Status**: Multi-Agent Lab structure migration completed. LangGraph Supervisor pattern implemented with TodoAgent.

## Development Commands

### Running Examples

```bash
# Basic demos (learning progression)
uv run python -m src.examples.01_basic_agent        # Basic Agent
uv run python -m src.examples.02_file_agent         # File operations
uv run python -m src.examples.03_schedule_agent     # Schedule manager
uv run python -m src.examples.04_middleware_demo    # Middleware patterns

# Skill Card system
uv run python -m src.examples.05_skill_card_demo    # Skill Card demo
uv run python -m src.examples.07_executor_demo      # Executor demo

# Advanced integration (most important)
uv run python -m src.examples.08_real_tools_demo    # LLM + DB + Logic tools (verbose debugging)
uv run python -m src.examples.09_dynamic_agent      # Dynamic tool selection
uv run python -m src.examples.10_langgraph_supervisor  # LangGraph Multi-Agent Supervisor

# Infrastructure demos
uv run python -m src.examples.13_elasticsearch_demo # Elasticsearch integration
uv run python -m src.examples.14_redis_demo         # Redis integration

# Quick test script
uv run python quick_test.py
```

### Testing

```bash
# Run all tests
uv run pytest

# Run with verbose output
uv run pytest -v

# Run specific test file
uv run pytest tests/core/test_skill_card_manager.py

# Run specific test
uv run pytest tests/core/test_skill_card_manager.py::test_skill_card_load
```

### Code Quality

```bash
# Lint with ruff
uv run ruff check .

# Auto-fix issues
uv run ruff check --fix .

# Format code
uv run ruff format .
```

### Environment Setup

```bash
# Install dependencies
uv sync

# Check Ollama model availability
ollama list

# Start Ollama server (if not already running)
ollama serve

# Pull required model
ollama pull gpt-oss:20b
```

## Architecture

### Core Pattern: Skill Card System

The **Skill Card** is a JSON-based metadata pattern that defines Agent behavior:

```
User Query → SkillCardManager → SkillCardExecutor → Tools → Result
```

**Key Components:**

1. **SkillCard** (`src/multi_agent_lab/platform/skill_card/schema.py`): Pydantic schema defining agent metadata
2. **SkillCardManager** (`src/multi_agent_lab/platform/skill_card/manager.py`): Loads JSON files
3. **SkillCardExecutor** (`src/multi_agent_lab/platform/skill_card/executor.py`): Executes the plan

**Execution Flow:**

```python
# 1. Load Skill Card from JSON
manager = SkillCardManager()
card = manager.get("SC_SCHEDULE_001")

# 2. Register tools
executor = SkillCardExecutor(card, verbose=True)
executor.register_tool("parse_event_info", parse_event_info)
executor.register_tool("create_event", create_event)

# 3. Execute
result = executor.execute(
    user_query="내일 오후 2시에 팀 회의",
    context={"user_id": "user123"}
)
```

### Variable Substitution Pattern

Skill Cards use `${variable}` syntax for data flow between steps:

```json
{
  "step": 2,
  "action": "create_event",
  "input": {
    "title": "${event_data.title}",           // From step 1
    "start_time": "${free_slots.best_slot.start}"  // Nested access
  },
  "output_to": "created_event"
}
```

Implementation: `SkillCardExecutor._substitute_variables()` uses regex pattern `\$\{([^}]+)\}` to replace variables with actual values from `ExecutionContext.variables`.

### Tool Types

**1. LLM Tools** (Structured Output with Pydantic):
```python
from pydantic import BaseModel, Field

class EventInfo(BaseModel):
    title: str = Field(description="일정 제목")
    date: str = Field(description="날짜 (YYYY-MM-DD)")

@tool
def parse_event_info(query: str, verbose: bool = False) -> dict:
    llm = ChatOllama(model="gpt-oss:20b", temperature=0.0)
    structured_llm = llm.with_structured_output(EventInfo)
    result: EventInfo = structured_llm.invoke(prompt)
    return result.model_dump()
```

**2. DB Tools** (Database operations):
```python
@tool
def create_event(title: str, start_time: str) -> dict:
    event = {...}
    db.add_event(event)  # In-memory DB
    return {"success": True, "event": event}
```

**3. Logic Tools** (Business logic):
```python
@tool
def find_free_time(date: str, duration: int = 60) -> dict:
    # Complex scheduling logic
    busy_slots = get_busy_slots(date)
    available_slots = calculate_free_time(busy_slots)
    return {"available_slots": available_slots}
```

### Verbose Debugging System

The project uses a **two-level debugging** approach:

**Level 1: SkillCardExecutor verbose mode**
```python
executor = SkillCardExecutor(card, verbose=True)
# Prints step-by-step execution, variable substitution, timing
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sskim91) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
