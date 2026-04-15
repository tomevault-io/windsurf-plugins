---
trigger: always_on
description: This document provides context for GitHub Copilot and other AI assistants working on this codebase.
---

# LangGraph Multi-Agent Requirement System - Copilot Instructions

This document provides context for GitHub Copilot and other AI assistants working on this codebase.

## Project Overview

- **Name:** LangGraph Multi-Agent Requirement System
- **Type:** Python/TypeScript multi-agent pipeline
- **Purpose:** Analyze software requirements and generate Gherkin BDD features + Playwright automation
- **Key Technologies:** LangGraph, OpenAI, Groq, SQLite (MCP-based), Playwright, Pydantic

## Architecture Summary

### Pipeline Flow

```
Input (Jira/File/Text)
  ↓ [Ingestion] → fetch via MCP tools
  ↓ [Change Detection] → semantic diff via Groq
  ↓ (skip if wording-only)
  ↓ [Classification] → area via OpenAI
  ↓ [Feature Gen] → Gherkin via OpenAI
  ↓ [Coverage Eval] → assessment via Groq
  ↓ [Router] → decide specialist path
  ├─ frontend? → [Playwright Agent]
  └─ other → [Post-Eval Agent]
  ↓ [Post-Eval] → QA via Groq
  ↓
Output (Features, Coverage, Automation)
```

### MCP Tool Usage Pattern

**All IO operations must use MCP tools, not direct calls:**

```python
# ✅ CORRECT: Use MCP tool abstraction
from services.db import get_db_service
db = get_db_service()
db.create_or_update_requirement(...)

# ❌ WRONG: Direct sqlite3 call
import sqlite3
conn = sqlite3.connect("db/context.db")
```

**MCP tools required:**
- `filesystem`: File read/write
- `sqlite`: Database operations
- `http` / `jira`: Jira REST API
- `shell`: npm, playwright commands
- `pdf_extract` / `docx_extract`: Document parsing

## Key Files & Responsibilities

| File | Responsibility |
|------|-----------------|
| `main.py` | CLI entrypoint using Typer |
| `graph.py` | LangGraph StateGraph orchestration |
| `state.py` | Pydantic state models & configs |
| `services/db.py` | MCP SQLite abstraction layer |
| `services/jira_client.py` | MCP HTTP/Jira tool wrapper |
| `services/nlp_utils.py` | Text processing (normalize, diff, extract) |
| `services/logging_utils.py` | Structured logging + LangSmith |
| `agents/*.py` | Individual agent nodes |

## Code Patterns & Conventions

### 1. Agent Structure

Every agent follows this pattern:

```python
class MyAgent(LoggingMixin):
    def __init__(self):
        super().__init__("my_agent")
        self.db_service = get_db_service()
    
    def process(self, state: RunState) -> RunState:
        self.log_info("Starting", run_id=state.run_id, requirement_id=state.requirement_id)
        # 1. Read from state
        # 2. Call MCP tools for IO
        # 3. Use LLM if needed (OpenAI or Groq)
        # 4. Update state
        # 5. Persist to SQLite if needed
        self.log_info("Complete", run_id=state.run_id)
        return state

def agent_node(state: RunState) -> RunState:
    if state.status != "expected_status":
        return state
    agent = MyAgent()
    return agent.process(state)
```

### 2. Logging Pattern

Always use LoggingMixin:

```python
self.log_info(
    "Message",
    run_id=state.run_id,
    requirement_id=state.requirement_id,
    metadata={"key": "value"}
)

self.log_error(
    "Error message",
    error=exception,
    run_id=state.run_id
)
```

### 3. MCP Tool Calls

All MCP tool calls go through service abstraction:

```python
# Database
db = get_db_service()
db.create_or_update_requirement(...)
db.create_artifact(...)
db.record_change(...)

# Jira
jira = JiraClient()
issue_data = jira.get_issue_as_requirement("JIRA-123")

# Files (directly or via services)
import os
with open(path, 'w') as f:
    f.write(content)
```

### 4. LLM Calls

Pattern for OpenAI (main agents):

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=[{"role": "user", "content": prompt}],
    temperature=0.7,
    max_tokens=2000
)
result = response.choices[0].message.content.strip()
```

Pattern for Groq (evaluators):

```python
from groq import Groq
client = Groq(api_key=os.getenv("GROQ_API_KEY"))

response = client.chat.completions.create(
    model="mixtral-8x7b-32768",
    messages=[{"role": "user", "content": prompt}],
    temperature=0.3,  # Lower for consistency
    max_tokens=500
)
result = response.choices[0].message.content.strip()
```

### 5. Type Hints

All functions must have type hints:

```python
def process(self, state: RunState) -> RunState:
    pass

def fetch_requirement(self, key: str) -> Dict[str, Any]:
    pass

def analyze_coverage(
    self,
    requirements: List[str],
    features: Dict[str, str]
) -> Dict[str, Any]:
    pass
```

### 6. Error Handling

Log errors, don't silently fail:

```python
try:
    # operation
except Exception as e:
    self.log_error(
        "Operation failed",
        error=e,
        run_id=state.run_id,
        requirement_id=state.requirement_id
    )
    state.errors.append(f"Operation failed: {str(e)}")
    return state
```

## State Management

RunState is immutable for LangGraph:

```python
# ✅ CORRECT: Return new/updated state
state.status = "new_status"
state.coverage_markdown_path = path
return state

# Don't modify in-place and forget to return
```

## Database Schema (SQLite via MCP)

Tables managed via `services/db.py`:

- `requirements`: Requirement master records

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wonderfull) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
