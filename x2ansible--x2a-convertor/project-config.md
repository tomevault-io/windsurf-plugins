---
trigger: always_on
description: This file provides guidance to any software agent (Claude Code, Codex) working with code in this repository.
---

# Agents.md

This file provides guidance to any software agent (Claude Code, Codex) working with code in this repository.

## Project Overview

X2A Convertor is an AI-powered infrastructure migration tool that converts Chef, Puppet, PowerShell, and Salt configurations to Ansible. It uses LangGraph with LLM-powered agents for analysis and migration.

The system has two major pipelines:

- **Input pipeline** (`src/inputs/`): Technology-specific analyzers that parse source infrastructure code (Chef, Puppet, PowerShell, Salt) and produce a structured migration plan
- **Export pipeline** (`src/exporters/`): Technology-agnostic agents that consume migration plans and produce Ansible roles

## Development Commands

```bash
# Install dependencies
uv sync

# Format + lint
uv run ruff format
uv run ruff check . --fix

# Type checking
uv run pyrefly check

# Run tests (excludes evals)
uv run pytest -m "not eval"

# Run evals only
uv run pytest -m "eval"

# Full CI check
make ci-check

# Run migration workflow
make name=hello_world run-init      # Step 1: init
make name=hello_world run-analyze   # Step 2: analyze
make name=hello_world run-migrate   # Step 3: migrate
```

## Architecture

### Agent Hierarchy

All agents inherit from `BaseAgent[S: BaseState]` (`src/base_agent.py`), which provides:

- Automatic telemetry via `__call__` -> `execute()`
- Three invocation modes: `invoke_react()`, `invoke_structured()`, `invoke_llm()`
- Declarative tool configuration via `BASE_TOOLS` class variable
- State-derived tools via `extra_tools_from_state()` hook
- Middleware stack (rules injection, goal validation, conversation compaction, debug dump)

Two intermediate base classes specialize `BaseAgent`:

| Base Class | File | Purpose |
|---|---|---|
| `InputAgent[S]` | `src/inputs/input_agent.py` | Sets `RULES_FILE = INPUT_AGENTS_FILE` for all analysis agents |
| `ExportAgent[S]` | `src/exporters/export_agent.py` | Sets `RULES_FILE = EXPORT_AGENTS_FILE` for all export agents |

### Input Pipeline (Analysis)

Each technology has a `*Subagent` orchestrator that wires analysis services into a LangGraph `StateGraph`. Services are `InputAgent` subclasses that call `invoke_structured()` with a Pydantic schema to extract structured data from source files.

Pattern (using Puppet as example):

```
PuppetSubagent (orchestrator, not a BaseAgent)
  -> ManifestAnalysisService(InputAgent)    # invoke_structured(ManifestExecutionAnalysis)
  -> HieraDataAnalysisService(InputAgent)   # invoke_structured(HieraDataAnalysis)
  -> TemplateAnalysisService(InputAgent)    # invoke_structured(PuppetTemplateAnalysis)
  -> ReportWriterAgent(InputAgent)          # invoke_react() with tools
  -> AnalysisValidationAgent(InputAgent)    # invoke_react() with tools
  -> CleanupAgent(InputAgent)              # invoke_react()
```

### Export Pipeline (Migration)

`ToAnsibleSubagent` orchestrates the export workflow:

```
ToAnsibleSubagent (orchestrator)
  -> AAPDiscoveryAgent     # Discover AAP Private Hub collections
  -> CredentialAgent       # Extract credentials for AAP
  -> PlanningAgent         # Build migration checklist (invoke_react)
  -> WriteAgent            # Write all Ansible files (invoke_react, internal retry loop)
  -> MoleculeAgent         # Generate Molecule test scaffolding
  -> ReviewAgent           # Review generated code
  -> ValidationAgent       # Lint + fix validation issues
```

### State Management

States are `@dataclass` classes inheriting from `BaseState` (`src/types/base_state.py`). They use an immutable update pattern:

```python
@dataclass
class BaseState(ABC):
    user_message: str
    path: str
    telemetry: Telemetry | None = field(default=None, kw_only=True)
    failed: bool = field(default=False, kw_only=True)
    failure_reason: str = field(default="", kw_only=True)

    def update(self, **kwargs) -> "BaseState":
        return replace(self, **kwargs)

    def mark_failed(self, reason: str) -> "BaseState":
        return self.update(failed=True, failure_reason=reason)
```

Key concrete states:
- `InitState` (`src/init/init_state.py`) -- init workflow
- `ExportState` (`src/exporters/state.py`) -- export workflow
- `FileAnalysisState` (`src/types/file_analysis_state.py`) -- per-file analysis
- `PuppetState`, `ChefState`, etc. -- technology-specific analysis

### Workflow Pattern (LangGraph)

All multi-agent workflows follow the same pattern:

```python
class SomeSubagent:
    def __init__(self):
        self.agent_a = AgentA(model=self.model)
        self.agent_b = AgentB(model=self.model)
        self._workflow = self._create_workflow()

    def _create_workflow(self):
        workflow = StateGraph(SomeState)
        workflow.add_node("step_a", self.agent_a)
        workflow.add_node("step_b", self.agent_b)
        workflow.add_edge(START, "step_a")
        workflow.add_conditional_edges("step_a", self._check_failure_after_agent)
        workflow.add_edge("step_b", END)
        return workflow.compile()

    def _check_failure_after_agent(self, state) -> Literal["step_b", "finalize"]:
        if state.failed:
            return "finalize"
        return "step_b"
```

Agents are callable (`__call__` on `BaseAgent`) and used directly as graph nodes.

## Prompt System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [x2ansible/x2a-convertor](https://github.com/x2ansible/x2a-convertor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
