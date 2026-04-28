---
trigger: always_on
description: Instructions for AI coding assistants and agentic frameworks (Hermes, Codex,
---

# Compounding Engineering -- Development Guide

Instructions for AI coding assistants and agentic frameworks (Hermes, Codex,
Claude Code, Copilot) working on the dspy-compounding-engineering codebase.

This is a **DSPy-native CLI tool** that implements the Compounding Engineering
philosophy: every unit of work makes subsequent work easier. It provides
multi-agent code review, planning, triage, and automated work execution --
all backed by a self-improving knowledge base.

---

## Project Structure

```
dspy-compounding-engineering/
├── pyproject.toml                 # Dependencies, ruff, pytest, build config
├── AGENTS.md                      # This file -- mandatory for all AI agents
├── cli.py                         # Typer CLI entry point
├── config.py                      # AppConfig + ServiceRegistry + DSPy configuration
├── docker-compose.yml             # Qdrant vector DB
│
├── agents/                        # DSPy agent definitions
│   ├── knowledge_gardener.py
│   ├── schema/
│   │   ├── base.py                # Base Pydantic models
│   │   ├── research.py            # Research input/output schemas
│   │   ├── review.py              # Review finding schemas
│   │   └── workflow.py            # Workflow/todo schemas
│   ├── review/                    # Multi-agent review specialists
│   │   ├── security_sentinel.py
│   │   ├── performance_oracle.py
│   │   ├── architecture_strategist.py
│   │   ├── data_integrity_guardian.py
│   │   ├── code_simplicity_reviewer.py
│   │   ├── agent_native_reviewer.py
│   │   ├── kieran_python_reviewer.py
│   │   ├── pattern_recognition_specialist.py
│   │   └── ...
│   ├── research/                  # Research agents
│   │   ├── repo_research_analyst.py
│   │   ├── best_practices_researcher.py
│   │   ├── framework_docs_researcher.py
│   │   └── git_history_analyzer.py
│   └── workflow/                  # Workflow agents
│       ├── plan_generator.py
│       ├── task_executor.py
│       ├── task_validator.py
│       ├── triage_agent.py
│       ├── command_generator.py
│       ├── feedback_codifier.py
│       └── ...
│
├── workflows/                     # Orchestration layer
│   ├── review.py                  # Multi-agent review pipeline
│   ├── triage.py                  # Interactive finding triage
│   ├── work.py                    # Unified work execution (ReAct)
│   ├── plan.py                    # Feature planning workflow
│   ├── codify.py                  # Knowledge codification
│   ├── sync.py                    # GitHub issue sync
│   └── generate_agent.py          # Meta agent generation
│
├── utils/                         # Infrastructure utilities
│   ├── io/
│   │   ├── files.py               # File I/O with safety guards
│   │   ├── logger.py              # Loguru logging setup
│   │   ├── safe.py                # Safe command execution
│   │   ├── status.py              # System status reporting
│   │   └── __init__.py            # get_system_status(), validate_agent_filters()
│   ├── knowledge/
│   │   ├── core.py                # KnowledgeBase class
│   │   ├── indexer.py             # Codebase indexing (Qdrant + keyword)
│   │   ├── embeddings.py          # Embedding provider abstraction
│   │   ├── gardener.py            # Knowledge gardening service
│   │   ├── extractor.py           # Learning extraction
│   │   ├── compression.py         # AI.md compression
│   │   └── docs.py                # Documentation fetching
│   ├── git/
│   │   └── service.py             # Git operations + worktree management
│   ├── github/
│   │   └── service.py             # GitHub API integration (PyGithub)
│   ├── mcp/
│   │   └── client.py              # MCP client
│   └── context/
│       ├── project.py             # Project context gathering
│       └── scorer.py              # Context relevance scoring
│
├── mcp_servers/                   # MCP server implementations
│   ├── compounding_server.py      # Main FastMCP server
│   ├── file_server.py
│   ├── git_server.py
│   └── search_server.py
│
├── tests/                         # Pytest suite
├── todos/                         # Pending review findings (markdown)
├── plans/                         # Feature implementation plans
├── docs/                          # MkDocs documentation
└── .knowledge/                    # Knowledge base (auto-populated)
```

---

## CORE RULES (VIOLATIONS ARE REJECTED)

### 1. ALWAYS use f-strings
Never use `.format()` or `%s` style string formatting.

```python
f"Reviewing {count} files"           # GOOD
"Reviewing {} files".format(count)   # BAD
```

### 2. ALWAYS use Pydantic BaseModel in DSPy Signatures
Every `dspy.InputField()` and `dspy.OutputField()` carrying structured data
MUST use a typed Pydantic `BaseModel`. Never use raw types (`str`, `float`,
`list[dict]`) in signatures.

```python
class ReviewFinding(BaseModel):
    file: str
    line: int
    severity: str
    description: str
    fix_strategy: str

class ReviewSignature(dspy.Signature):
    code_context: str = dspy.InputField()
    findings: list[ReviewFinding] = dspy.OutputField()
```

### 3. NEVER use `from __future__ import annotations`
Breaks DSPy type introspection and Pydantic forward reference resolution.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Strategic-Automation/dspy-compounding-engineering](https://github.com/Strategic-Automation/dspy-compounding-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
