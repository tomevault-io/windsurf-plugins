---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an agentic system that takes articles from websites and generates short video scripts through a multi-agent pipeline. The system uses LangChain/LangGraph for orchestration with a terminal input/output interface.

**Key agents:**
- **Writer**: Generates video scripts from article content (700-1000 chars), has scraper tool for fetching URLs
- **Editor**: Reviews scripts for tone match, hook quality, clarity, length (can reject)
- **FactChecker**: Verifies factual accuracy against source article (can reject)
- **User**: Human-in-the-loop approval/feedback after each iteration

Agents can reject and return work to the writer with feedback, creating an iterative improvement loop.

## Development Setup

**Install dependencies:**
```bash
uv sync
```

**Run linter:**
```bash
uv run ruff check .
```

**Auto-fix linting issues:**
```bash
uv run ruff check --fix .
```

**Format code:**
```bash
uv run ruff format .
```

## Architecture

### Core Design Pattern: LangGraph State Machine

The system uses **LangGraph** (not plain LCEL) because it provides:
- Built-in cycle support for rejection loops
- Conditional edge routing (editor → writer if rejected, → factchecker if approved)
- Automatic state management
- Native async/await support

**Pipeline flow:**
```
Article → Writer → Editor → FactChecker → User Approval
             ↑________|_________|
            (rejection with feedback)
```

### State Management

Central state flows through all agents as `PipelineState` (TypedDict):

```python
class PipelineState(TypedDict):
    messages: Annotated[list[BaseMessage], add]  # Unified conversation history
    drafts: Annotated[list[str], add]            # All script versions
    article_content: Annotated[list[str], add]   # Source articles (can have multiple)
    iteration: int                                # Rejection loop counter
    editor_approved: bool                         # Editor approval flag
    factchecker_approved: bool                    # FactChecker approval flag
    user_approved: bool                           # User exit signal
```

**Key patterns:**
- **Unified messages**: All conversation (user, writer, editor, factchecker) in one list
- **Message identification**: Each agent uses `AIMessage(content=..., name="writer"|"editor"|"factchecker")`
- **State accumulation**: `Annotated[list, add]` automatically appends to lists
- **Immutability**: Agents return partial state updates, LangGraph merges them

State is immutable between nodes; agents return dict updates that get merged into state.

### Agent Interface

**Current implementation**: Agents are simple functions that take `state` and return partial state updates.

```python
def writer_agent(state: PipelineState) -> dict:
    """Generate script from article."""
    # Build message history with system prompt
    messages = [SystemMessage(content=WRITER_PROMPT)] + state["messages"]

    # Add article context
    if state["article_content"]:
        messages.append(HumanMessage(content=f"Article:\n{article_text}"))

    # Tool calling loop for scraper
    response = writer_llm.invoke(messages)

    # Return updates
    return {
        "messages": [AIMessage(content=response.content, name="writer")],
        "drafts": [script],
        "iteration": state["iteration"] + 1,
    }
```

**Agent patterns:**
- **user_input_node**: Gets terminal input, returns `HumanMessage`
- **writer_agent**: Invokes LLM with tool calling loop, returns `AIMessage` with script
- **editor_agent**: Uses LCEL chain + structured output (Pydantic), returns approval + feedback
- **factchecker_agent**: Same pattern as editor

**Routing functions** (in `routes.py`) determine next node based on approval flags and iteration count.

### Future: Plugin Architecture (Planned)

The system is designed to eventually support:
- Abstract `BaseAgent` class with `@register_agent` decorator
- Config-driven agent composition (`config.yaml`)
- Zero-code agent addition

Currently: Functional implementation for simplicity and rapid iteration.

## Project Structure

```
src/langchain_examples/
├── agents/
│   ├── agents.py     # All agent functions (writer, editor, factchecker, user_input)
│   ├── prompts.py    # System prompts (WRITER_PROMPT, EDITOR_PROMPT, FACTCHECKER_PROMPT)
│   ├── models.py     # Pydantic models (EditorOutput, FactCheckerOutput)
│   ├── routes.py     # Routing functions (route_after_writer, route_after_editor, etc.)
│   └── state.py      # PipelineState TypedDict
├── tools/
│   └── scrapers.py   # Article scraper tool
└── main.py           # Pipeline builder and executor

examples/             # Example scripts
├── example1.py
└── example2.py

docs/                 # Architecture documentation
├── architecture.md
├── design-decisions.md
└── extensibility-guide.md

data/                 # Runtime data
└── runs/
    └── YYYYMMDD_HHMMSS/
        ├── pipeline_result.json
        ├── pipeline_graph.png
        └── final_script.txt

TODO.md               # Feature roadmap
CLAUDE.md             # This file
```

## Key Design Decisions

### Functional Approach (Current)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kkruglik/langchain_examples](https://github.com/kkruglik/langchain_examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
