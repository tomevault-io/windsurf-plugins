---
trigger: always_on
description: This file tells AI agents (Claude Code, Codex, Cursor, Hermes, etc.) how to work with this project.
---

# AGENTS.md — Swarm v2

This file tells AI agents (Claude Code, Codex, Cursor, Hermes, etc.) how to work with this project.

## Project Overview

Multi-agent research orchestration using Ollama cloud models. Spawns parallel workers with focused research angles, each with tool access, and collects their outputs via a shared write-only scratchpad.

**Library core is still pure stdlib**, but the optional persistent TUI uses `textual` as its one external dependency.

## Architecture

```
swarm/
├── __init__.py       # Public API: from swarm import run_swarm
├── __main__.py       # CLI entry point (thin wrapper, ~60 lines)
├── runner.py         # Library entry point: run_swarm()
├── orchestrator.py   # Spawns workers, manages scratchpad, pipeline mode
├── preflight.py      # LLM-based question analysis + skill assignment
├── worker.py         # Worker agent loop (Ollama chat + tool calls)
├── scratchpad.py     # Write-only RAM SQLite for raw findings
├── search.py         # Search backends: SearXNG, DuckDuckGo, Google
├── synthesis.py      # Orchestrator synthesis (boss reads the room)
├── llm.py            # Shared LLM helper: OpenAI-compat + optional LiteLLM, retry/backoff, streaming, cost
├── providers.py      # Provider resolution: model tags → endpoint, API key, headers
├── credibility.py    # AI-based probabilistic source credibility (Bayesian)
├── cache.py          # SQLite search/extract result cache
├── config.py         # Config loader from JSON file
├── complexity.py     # Model-based complexity estimation (1-5)
├── output.py         # Output formatting + markdown file saving
├── skills/           # Skill system (capability packs)
│   ├── __init__.py   # SkillRegistry, get_skill_registry()
│   ├── _base.py      # Skill dataclass + registry + hand-rolled YAML parser
│   ├── default/SKILL.md
│   ├── research/     # Full pack: SKILL.md + team.json
│   ├── search/SKILL.md
│   ├── vision/SKILL.md
│   ├── code/SKILL.md
│   ├── code-debug/SKILL.md
│   ├── files/SKILL.md
│   ├── fact-check/   # Full pack: SKILL.md + team.json
│   ├── multi-hop/SKILL.md
│   ├── comparison/SKILL.md
│   ├── academic/     # Full pack: SKILL.md + team.json
│   ├── legal/        # Full pack: SKILL.md + team.json
│   ├── medical/      # Full pack: SKILL.md + team.json
│   ├── finance/      # Full pack: SKILL.md + team.json
│   ├── data-analysis/# Full pack: SKILL.md + team.json (pipeline)
│   ├── summarize/    # Full pack: SKILL.md + team.json
│   ├── translate/    # Full pack: SKILL.md + team.json
│   ├── historical/   # Full pack: SKILL.md + team.json
│   ├── code-review-swarm/  # Full pack: SKILL.md + team.json
│   ├── debate/        # Full pack: SKILL.md + team.json
│   └── reverse-engineering/  # Full pack: SKILL.md + team.json
├── integrations/     # External harness adapters
│   └── mcp/          # MCP server: swarm_research tool (optional mcp extra)
├── prompts/          # External markdown prompt templates
│   ├── __init__.py   # load_prompt() and render_prompt()
│   ├── preflight.md  # Preflight JSON-generation prompt
│   ├── worker.md     # Worker system prompt template
│   ├── synthesis.md  # Synthesis prompt template
│   ├── mode_*.md     # Objective / subjective mode instructions
│   └── fallback_*.md # Fallback model prompts
├── tui/              # Optional persistent Textual TUI
│   ├── __init__.py   # Exports run_tui, Session, SessionStore
│   ├── app.py        # Main Textual app + event loop
│   ├── session.py    # In-memory session model + follow-up context
│   ├── store.py      # SQLite persistence for sessions/results
│   └── widgets.py    # ChatLog, WorkerGrid, SessionList, InputBar
└── tools/            # Modular tool registry
    ├── __init__.py   # Registry: get_registry(), reset_registry()
    ├── base.py       # BaseTool abstract class
    ├── registry.py   # ToolRegistry: discover, register, skill delegation
    ├── web_search.py # Search the web
    ├── web_extract.py# Read content from URLs
    ├── scratchpad.py # Log findings tool
    ├── vision.py     # Read images via Gemma4
    ├── python_exec.py# Execute Python code
    ├── file_reader.py# Read .txt/.csv/.json/.xlsx
    ├── wikipedia_search.py # Search Wikipedia for encyclopedic facts
    ├── arxiv_search.py # Search arXiv for academic papers
    ├── github_search.py # Search GitHub repos/issues/code
    ├── wayback_machine.py # Find archived snapshots of URLs
    ├── http_request.py # Generic REST API client
    ├── pdf_extract.py # Read PDFs (optional pdf extra)
    ├── sql_query.py # Run read-only SQL against a local DB
    ├── regex_extract.py # Extract structured data from text
    ├── text_diff.py # Unified diff between two texts
    └── date_calculator.py # Date arithmetic (days between, weekday, age)
```

## Key Design Decisions

### Library-first
The main entry point is `swarm/runner.py` → `run_swarm()`. The CLI (`__main__.py`) is a thin wrapper. Use as a library:

```python
from swarm import run_swarm
from swarm.output import save_markdown

result = run_swarm("Your question", mix=True)
save_markdown(result, result["goal"])
```

### Preflight (LLM-based question analysis)
Before spawning workers, the orchestrator calls DeepSeek V4 Flash to analyze the question:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adilfaisal01/llm-multiagent-swarm](https://github.com/adilfaisal01/llm-multiagent-swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
