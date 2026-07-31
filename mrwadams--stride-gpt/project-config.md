---
trigger: always_on
description: Orientation for agents (and humans) working in this codebase. Skim the whole file before starting; jump to "Progressive disclosure pattern" if you're looking to reuse that specific idea.
---

# AGENTS.md

Orientation for agents (and humans) working in this codebase. Skim the whole file before starting; jump to "Progressive disclosure pattern" if you're looking to reuse that specific idea.

## What this project is

STRIDE-GPT is an AI-powered threat modelling tool that produces STRIDE reports for software systems. It ships as **two separate products** with a shared library between them:

- **CLI** (`stride_gpt/`) — the primary product. Includes an **agentic analysis engine** that explores a real codebase via filesystem tools and produces a per-subsystem STRIDE report.
- **Streamlit web UI** (`apps/web/`) — a hosted single-shot threat modeller. Takes a text description (or a GitHub URL) and produces a one-shot report. **Does not use the agentic engine**, by design — filesystem/agentic features don't fit Streamlit Community Cloud-style deployments.

Both surfaces share `stride_gpt/core/` for prompts, schemas, and the LLM abstraction.

## Repository layout

```
stride_gpt/                 # CLI package + shared library
├── cli.py                  # Typer commands + interactive REPL
├── prompt.py               # prompt_toolkit completer (slash commands)
├── config.py               # ~/.stride-gpt/config.json + provider registry
├── models.py               # model catalogue (id, default tokens, thinking support)
├── agent/                  # agentic loop (CLI-only by design)
│   ├── loop.py             # run_analysis, per-subsystem agent loop
│   ├── planner.py          # phase 1 — classify app type, propose subsystems
│   ├── context.py          # context window management + compression
│   ├── tools.py            # filesystem tools (read_file, grep, ..., list_references, load_reference)
│   ├── progress.py         # Rich-based progress callbacks
│   └── report.py           # markdown / JSON / SARIF rendering, save/load
└── core/                   # shared between CLI and web
    ├── llm.py              # unified call_llm / call_llm_with_tools via litellm
    ├── schemas.py          # LLMConfig, AnalysisPlan, AnalysisReport, etc.
    ├── prompts/
    │   ├── builder.py      # legacy single-shot prompt builder (web UI uses this)
    │   ├── variants.py     # base_system_prompt(), list_references(), load_reference(), coerce_app_type()
    │   └── threat_model/   # packaged reference cards (see "Progressive disclosure")
    │       ├── base.md
    │       ├── quick_base.md
    │       ├── genai.md
    │       ├── agentic.md
    │       ├── insider_threat.md
    │       ├── mitre_enterprise.md
    │       └── mitre_atlas.md
    └── ...                 # attack_tree, dread, mitigations, test_cases, threat_model

apps/web/                   # Streamlit UI (separate product)
└── ...                     # imports from stride_gpt.core, NOT stride_gpt.agent

tests/                      # pytest, fixtures in conftest.py
```

## How the agentic analysis works

Three phases, all driven from `cli.py:analyze` (the subcommand) or `_handle_analyze` (the interactive `/analyze`):

1. **Planning** (`agent/planner.py:create_plan`) — single LLM call. Scans the codebase, classifies the application type (`web` / `genai` / `agentic`), proposes 3–7 subsystems. User approves the plan interactively (or `--yes`).
2. **Per-subsystem analysis** (`agent/loop.py:_analyze_subsystem`) — for each subsystem, a tool-using agent loop. The model reads files, greps, lists directories, and loads OWASP reference cards on demand, then emits a JSON finding. Token budget is shared across subsystems (remaining-budget arithmetic in `run_analysis`).
3. **Synthesis** (`agent/loop.py:_synthesize`) — one LLM call. Reviews all per-subsystem findings and surfaces cross-cutting threats.

Reports auto-save under `~/.stride-gpt/reports/`, split by kind: `/analyze` runs land in `reports/analyze/<target>_<timestamp>.json` and `/quick` runs in `reports/quick/<name>_<timestamp>.json` (see `config.py:analyze_reports_dir` / `quick_reports_dir`). The `/reports` slash command lists and re-renders saved reports (`--quick` / `--all` switch which kind it shows).

## Progressive disclosure pattern

The agent doesn't carry every threat framework in its system prompt — that would balloon every per-subsystem call. Instead it follows the same pattern Claude Code skills use: **a small always-loaded base prompt advertises optional reference cards; the model loads them on demand via a tool.**

Current card catalogue (in `core/prompts/threat_model/`):
- `genai.md` — OWASP Top 10 for LLM Applications (LLM01–LLM10). Asset-under-attack lens for LLM-using subsystems.
- `agentic.md` — OWASP Top 10 for Agentic Applications (ASI01–ASI10). Asset-under-attack lens for agentic subsystems; loaded in addition to the genai card.
- `insider_threat.md` — AI Insider Threat framework (distilled from <https://ai-insider-threat.matt-adams.co.uk>). Agent-as-insider lens; complementary to the OWASP cards rather than alternative. Loaded for high-autonomy agentic subsystems.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrwadams/stride-gpt](https://github.com/mrwadams/stride-gpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
