---
trigger: always_on
description: **Last Updated**: 2026-04-06
---

# CLAUDE.md - Haive Agent Framework

**Version**: 5.0
**Last Updated**: 2026-04-06

## Project Context

- **Directory**: `/home/will/Projects/haive`
- **Branch**: `final-refactor`
- **Structure**: Monorepo with Git submodules (7 packages)
- **Core Rules**:
  - Always use `poetry run` prefix for ALL Python commands
  - Real components only - NO MOCKS EVER in tests
  - Always use explicit imports: `from haive.core.*`
  - Be EXTREMELY careful with submodules - each is its own repo

## Guides & Documentation

### Agent Design (NEW — 2026-04-06)

- **@project_docs/guides/agent/AGENT_DESIGN_PATTERNS.md** — How to build agents around BaseGraph, state schemas, SimpleAgent/ReactAgent/MultiAgent patterns, anti-patterns
- **@project_docs/guides/agent/MULTIAGENT_STATE_DESIGN.md** — Complex state schemas for multi-agent systems, sequential/parallel/conditional patterns
- **@project_docs/guides/agent/CUSTOM_NODES_AND_GRAPHS.md** — Custom nodes, graph patterns (branching, parallel, reflection loops), NodeConfig types
- **@project_docs/guides/agent/MEMORY_AGENT_GUIDE.md** — Memory + KG integration, Neo4j Cypher, store namespaces, docker-compose
- **@project_docs/guides/agent/STATE_SCHEMA_NOTES.md** — State flow research, engine injection fix, schema hierarchy

### Architecture

- **@project_docs/active/architecture/state_schema_engine_gap.md** — How engines flow through state (FIXED)
- **@project_docs/active/architecture/multi_agent_meta_agent_memory_hub.md** — Multi-agent architecture decisions
- **@project_docs/active/architecture/agent_as_tool_pattern.md** — Agent-as-tool composition
- **@project_docs/guides/TOOL_ROUTING_REFACTOR.md** — Tool routing: pydantic_model vs pydantic_tool vs parse_output

### Standards

- **@project_docs/active/standards/coding/PYDANTIC_PATTERNS.md** — Pydantic best practices
- **@project_docs/active/standards/testing/philosophy.md** — No-mocks testing
- **@project_docs/active/standards/git/workflow.md** — Git safety protocol

## Quick Reference

### Essential Imports

```python
from haive.core.engine.aug_llm import AugLLMConfig
from haive.core.schema.prebuilt.llm_state import LLMState
from haive.agents.simple.agent import SimpleAgent
from haive.agents.react.agent import ReactAgent
from haive.agents.multi.agent import MultiAgent
from haive.agents.memory import create_memory_agent
from haive.agents.utils.trace import run_traced
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage, AIMessage
```

### Agent Patterns (the 4 you need)

```python
# 1. SimpleAgent — conversation, no tools
agent = SimpleAgent(name="writer", engine=AugLLMConfig(
    temperature=0.8, system_message="You are a writer."
))

# 2. ReactAgent — tools + reasoning loop
@tool
def search(query: str) -> str:
    '''Search.'''
    return f"Results for {query}"

agent = ReactAgent(name="researcher", engine=AugLLMConfig(
    tools=[search], system_message="Use search tool."
), max_iterations=3)

# 3. MultiAgent — compose agents
pipeline = MultiAgent(name="pipeline",
    agents=[researcher, writer], execution_mode="sequential")

# 4. MemoryAgent — persistent memory + KG
agent = create_memory_agent(name="assistant", user_id="user123",
    connection_string="postgresql://haive:haive@localhost/haive")
```

### Debug & Trace

```python
from haive.agents.utils.trace import run_traced
result = run_traced(agent, "Hello", save_to="traces/")
```

## Critical Development Rules

1. **NO MOCKS EVER**: Test with real LLMs, real tools, real components
2. **Poetry Run Everything**: `poetry run python`, `poetry run pytest` — never run Python directly
3. **Explicit Imports**: `from haive.core.engine import X` not `from engine import X`
4. **Pydantic**: Never override `__init__`, use `model_post_init()` and Field()
5. **Tools in AugLLMConfig**: Pass tools via `AugLLMConfig(tools=[...])`, not `self.tools.append()`
6. **State Schema**: Use `LLMState` when agent has tools (includes engines dict for tool_node)
7. **System Messages**: Go in `AugLLMConfig(system_message=...)`, not ChatPromptTemplate
8. **Agent Composition**: Use MultiAgent, not complex inheritance
9. **Git Safety**: Always diff before commit, commit submodules first, never force push
10. **Async Postgres preferred**: Use PostgresStoreWrapper for production, not InMemoryStore
11. **Research First**: Check existing patterns before implementing — `grep -r "pattern" packages/`
12. **Keep It Simple**: Avoid over-engineering; one line compositions like `MultiAgent([A, B], mode="sequential")`

## Coding Standards

### Python Code Style

```python
# ✅ CORRECT — descriptive names, type hints, early returns
def process_agent_response(
    agent_response: str,
    validation_config: ValidationConfig,
) -> ProcessedResponse:
    """Process agent response with validation."""
    if not agent_response:
        raise ValidationError("Empty response")
    if not validation_config.enabled:
        return ProcessedResponse(content=agent_response, validated=False)
    validated = validate_response(agent_response, validation_config)
    return ProcessedResponse(content=validated, validated=True)

# ❌ WRONG — poor naming, no types, nested logic
def process(resp, config):
    if resp:
        if config:
            if config.enabled:
                return validate_response(resp, config)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pr1m8/haive](https://github.com/pr1m8/haive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
