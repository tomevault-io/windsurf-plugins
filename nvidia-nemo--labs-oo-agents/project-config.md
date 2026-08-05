---
trigger: always_on
description: Python framework for building AI agents. An agent is a Python object — its methods are its capabilities.
---

# NVIDIA OO Agents Framework

Python framework for building AI agents. An agent is a Python object — its methods are its capabilities.

## Core Concept

```python
class MyAgent(Agent, llm=llm):
    # Deterministic method — a tool the LLM can call
    def get_stock(self, item: str) -> int:
        return INVENTORY.get(item, 0)

    # Generation method — LLM executes this; docstring = prompt
    async def analyze(self, data: str) -> Result:
        """Analyze the data. Use self.get_stock() for inventory checks."""
        ...  # ← ellipsis triggers LLM generation
```

**Ellipsis (`...`) = LLM generation.** No ellipsis = regular Python.
**Docstring = prompt.** Instructions only — arguments are rendered to the LLM by default.
**Return type = contract.** Pydantic models force structured LLM output.

### Arguments Are Rendered By Default — Never `{param}` in Docstrings

The framework already shows the LLM every argument: the method signature accompanies the task, the default CodeAct prefill pprint()s each parameter value under the truncation config (and the values are live REPL variables), and Predict serializes parameters with size caps. Writing `{data}` in a docstring re-injects the raw value into the instruction text — redundant, **untruncated** (huge arguments blow up the context), and it moves untrusted data into the instruction channel. Reserve `{...}` templating for what the signature cannot show: `{self.attr}` instance state and computed expressions like `{len(items)}`.

## Quick Rules

### Method Design

- **One method = one LLM task.** Don't make a method do classification AND implementation. If your method identifies terms, greps, and summarizes — split it into three.
- **Orchestrators are pure Python.** Workflow sequence methods have real bodies (no `...`), calling generation methods for each step. If a class has no `...` methods it doesn't need to subclass `Agent` at all.
- **Helpers beat prompts.** Deterministic logic as regular methods > telling the LLM to figure it out. Methods are visible to the LLM via `doc(self)` (auto-generated API documentation). Define helpers as class methods, not as lambda/function references assigned to `self`.
- **Evidence before assertions.** Run tests/verification before claiming work is done. Enforce in the orchestrator.
- **Everything visible by default.** Module-level and agent-level names are visible to the LLM (in `doc(self)` and exec_globals). Hide explicitly with `@hidden`, `Annotated[T, hidden]`, or `with hidden:`.

### Visibility

Single rule (Python-style): **visible by default, hide explicitly.**

- **Module level:** Imports, constants, functions, and classes are visible to agent-generated code unless you hide them:
  - `@hidden` on module-level functions
  - `Annotated[T, hidden]` on module-level variables (e.g. `API_KEY: Annotated[str, hidden] = "..."`)
  - `with hidden:` for imports or unannotated names you want to keep out of exec_globals
- **Agent** (methods, fields): Public names visible by default. `_private` methods/fields hidden by default. Opt out with `@hidden` or `Annotated[T, hidden]`; opt a `_private` method back in with `@spec(hidden=False)`.
- **Types:** Types used in the agent's public API (return types, parameter types, fields) must be **defined or imported at module level** so they appear in exec_globals. No automatic injection.

| Scope | Default | Opt-out / Opt-in |
|-------|---------|-----------------|
| Module level | VISIBLE | `@hidden`, `Annotated[T, hidden]`, `with hidden:` |
| Class methods (public) | VISIBLE | `@hidden` |
| Class methods (`_private`) | HIDDEN | `@spec(hidden=False)` |
| Class fields (public) | VISIBLE | `Annotated[T, hidden]` |
| Class fields (`_private`) | HIDDEN | — |
| Types | Must be at module level (import or define) to be in exec_globals | — |
| `context` / `events` | **Hidden by default** | Opt in: `spec(self, "context", hidden=False)` in subclass `__init__` |

```python
from __future__ import annotations

import json
import re
from pathlib import Path
from typing import Annotated

from nooa import Agent, hidden, spec

CATEGORIES = ["billing", "technical", "general"]   # visible to LLM by default
MAX_RESULTS = 10

with hidden:
    import secrets  # LLM cannot see this

class SearchAgent(Agent, llm=llm):
    index_path: Path = Path("data/index.json")
    api_key: Annotated[str, hidden] = ""       # hidden from LLM

    def search(self, query: str) -> list[str]:
        """Search the index for the query."""
        ...

    @hidden
    def rebuild_index(self) -> None:
        raw = Path("data/raw.json").read_text()
        self._entries = json.loads(raw)

    @spec(hidden=False)
    def _shown_helper(self) -> str:
        """This private method is explicitly shown in doc() output."""
        return self._compute()
```

To unhide a parent's hidden field, re-declare in the subclass without `hidden`:

```python
class MyAgent(Agent, llm=llm):
    my_tool: MyTool  # unhides Parent's Annotated[MyTool, hidden]
```

> **Note:** `context` and `events` are exceptions — do NOT re-declare them. Use `spec()` instead (see below).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-NeMo/labs-OO-Agents](https://github.com/NVIDIA-NeMo/labs-OO-Agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
