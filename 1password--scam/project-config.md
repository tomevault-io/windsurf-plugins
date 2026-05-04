---
trigger: always_on
description: SCAM (Security Comprehension Awareness Measure) is a CLI benchmark that evaluates the safety of AI agents with tool access. It runs multi-turn conversations where an AI agent uses tools (inbox, browser, forms, credential vault) and must proactively protect the user from threats without being told to look for them.
---

# AGENTS.md — AI Coding Guidelines for SCAM

## Project Overview

SCAM (Security Comprehension Awareness Measure) is a CLI benchmark that evaluates the safety of AI agents with tool access. It runs multi-turn conversations where an AI agent uses tools (inbox, browser, forms, credential vault) and must proactively protect the user from threats without being told to look for them.

## Architecture

```
CLI (typer) → Agentic Runner → Model.chat() (tool-calling loop) → ToolRouter (simulated env)
                ↓                                                        ↓
         Checkpoint evaluator                                  Environment (emails, URLs, vault)
                ↓
         Reporting (rich + markdown)
                ↓
         Export (HTML, video, terminal replay)

```

### Module Responsibilities

| Module | Purpose |
|--------|---------|
| `scam/cli.py` | CLI commands: `run`, `evaluate`, `replay`, `export`, `compare`, `report`, `scenarios` |
| `scam/models/base.py` | `BaseModel` (abstract: `chat()` required), `ChatResponse`, `ToolCall` |
| `scam/models/__init__.py` | `create_model()` factory function, re-exports |
| `scam/models/anthropic.py` | Anthropic Claude adapter (Messages API + tool calling) |
| `scam/models/openai.py` | OpenAI adapter (Chat Completions API + tool calling) |
| `scam/models/gemini.py` | Google Gemini adapter (google-genai SDK + function calling) |
| `scam/models/discovery.py` | Dynamic model listing from provider APIs, interactive picker |
| `scam/agentic/scenario.py` | YAML parser, `AgenticScenario` dataclass, `STANDARD_TOOLS` |
| `scam/agentic/environment.py` | `ToolRouter` — simulates inbox, URLs, forms, vault; tracks dangerous calls |
| `scam/agentic/evaluator.py` | Checkpoint-based scoring for agentic scenarios (regex + optional LLM judge) |
| `scam/agentic/judge.py` | LLM-as-judge evaluator — semantic fallback when regex pattern matching misses |
| `scam/agentic/runner.py` | Multi-turn conversation orchestrator for agentic evaluation |
| `scam/agentic/reporting.py` | Terminal reports, comparison, markdown reports (agentic) |
| `scam/agentic/aggregate.py` | Multi-run statistical aggregation (mean, std, CI, stability) |
| `scam/agentic/replay.py` | Interactive terminal replay viewer with typing effects and scorecard |
| `scam/agentic/export_html.py` | Self-contained HTML export with animated replay, per-scenario and combined index pages |
| `scam/agentic/export_video.py` | MP4 video export using Pillow for frame rendering and FFmpeg for encoding. Includes title cards, animated message bubbles with markdown rendering, tool call visualization, and scorecard overlays |
| `scam/utils/config.py` | Paths, model pricing, API keys, `skill_hash()`, `agentic_scenario_hash()`, `estimate_agentic_cost()`, `calculate_cost()` |

## Directory Layout

```
scenarios/                      # Agentic scenario YAML files (main contribution surface)
├── inbox_phishing.yaml
├── social_engineering.yaml
├── credential_exposure.yaml
├── credential_autofill.yaml
├── ecommerce_scams.yaml
├── data_leakage.yaml
├── confused_deputy.yaml
├── multi_stage.yaml
├── prompt_injection.yaml
└── _template.yaml              # Starter template for contributors

skills/                         # System prompt skill files
├── baseline.md                 # Minimal prompt (control condition)
├── security_expert.md          # Symlink to security-awareness/SKILL.md
└── security-awareness/
    └── SKILL.md                # Security awareness skill (Agent Skills format)

results/
├── official/                   # Tracked official evaluation results
└── agentic/                    # Scratch evaluations (gitignored)

exports/                        # HTML and video exports (gitignored)
```

### Key path constants (in `scam/utils/config.py`)

| Constant | Path |
|----------|------|
| `SCENARIOS_DIR` | `scenarios/` |
| `AGENTIC_SCENARIOS_DIR` | `scenarios/` |
| `RESULTS_DIR` | `results/` |
| `AGENTIC_RESULTS_DIR` | `results/agentic/` |

## Conventions

### Python Style

- Python 3.11+ with `from __future__ import annotations`
- Type hints on all function signatures
- Dataclasses for structured data, Pydantic for validated config
- `async/await` for all model API calls
- Docstrings on all public functions and classes

### Model Interface

The model interface is `BaseModel.chat()` (abstract, required). All evaluation flows through this method.

When adding a new model adapter:
1. Implement `chat()` (required)
2. Add retry logic with `tenacity` for transient errors
3. Update `resolve_model_provider()` in `config.py` with prefix detection
4. Add pricing to `MODEL_PRICING` in `config.py` (if applicable)
5. Add a listing function in `scam/models/discovery.py`
6. Add to `create_model()` in `scam/models/__init__.py`

### Agentic Tools

The standard tool set (defined in `scam/agentic/scenario.py`) gives the agent full authority:

| Tool | Description | Danger detection |
|------|-------------|-----------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1Password/SCAM](https://github.com/1Password/SCAM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
