---
trigger: always_on
description: AutoInterp is an automated mechanistic interpretability research framework. It takes a research question as input and produces a research report with original analyses, visualizations, and interpretation. Each step of the research process is executed by an LLM agent.
---

# CLAUDE.md

## Project Overview

AutoInterp is an automated mechanistic interpretability research framework. It takes a research question as input and produces a research report with original analyses, visualizations, and interpretation. Each step of the research process is executed by an LLM agent.

## Quick Start

```bash
pip install -r requirements.txt
python main.py          # interactive provider selection, then full pipeline
python main.py run      # same as above
python main.py literature-search  # run literature search only (no full pipeline)

# Headless / SLURM batch runs (no interactive prompts)
python main.py run --provider anthropic --model claude-sonnet-4-6 --topic "superposition in LLMs"
python main.py run --provider anthropic --model claude-sonnet-4-6 --topic ""  # auto-generate topic

# Prompt testing (replay individual stages against completed runs)
python test_prompt.py viz --project <completed_run> --dry-run   # preview prompt
python test_prompt.py viz --project <completed_run>             # run stage
```

Environment variables: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `OPENROUTER_API_KEY`, `HF_TOKEN` (set whichever provider you use).

## Pipeline Flow (`main.py` → `streamlined_pipeline()`)

1. **Literature Search** (optional) — sample papers from citation graph, download articles, generate research questions
2. **Question Generation** — LLM writes candidate research questions (skipped if literature search produced questions)
3. **Question Prioritization** — selects best question, extracts TITLE, renames project dir
4. **Iterative Analysis** — plans, codes, executes, debugs, evaluates autonomously (multiple iterations until confident)
5. **Visualization** — reads all analyses and generates publication-quality figures
6. **Report Generation** — writes an academic-style report
7. **AutoCritique** (optional) — automated peer review with verdict (Accept / Revise and Resubmit / Reject)
8. **Revision** (conditional) — if "Revise and Resubmit", addresses each recommendation
9. **Report Revision** (conditional) — produces revised report incorporating all changes
10. **Repo Assembly** — assembles finalized files into a clean `repo/` directory with README
11. **Notebook Generation** — creates a self-contained Jupyter notebook from the repo
12. **Publish** (manual) — upload a completed project to Future Science via `python main.py publish`

## Agent Architecture

Most pipeline stages run as CLI agent subprocesses (`claude` for Anthropic, `codex` for OpenAI) via `run_agent_with_polling()` in `src/core/agent_subprocess.py`. Each agent has:
- A `use_agent` config toggle (default `true`)
- An `agent_timeout` config value (counts only agent thinking time, not child process execution)
- A prompt template in `prompts/agent_*.yaml`
- Agent logic in the corresponding `src/*/agent_*.py` module
- **Fallback rules**: agents generally fall back to legacy LLM API calls if the agent is disabled, the provider isn't anthropic/openai, the CLI binary isn't found, or the agent fails. AutoCritique, repo, and notebook agents have no legacy fallback and simply skip.

The subprocess runner (`run_agent_with_polling()`) provides filesystem-based progress polling, heartbeat messages, and milestone detection.

## Key File Locations

| File | Purpose |
|------|---------|
| `main.py` | Main orchestrator, CLI entry point, options menu, manual config |
| `config.yaml` | All configuration (providers, agents, execution, literature search, UI) — committed |
| `config.local.yaml` | Personal overrides (API keys, author info, etc.) — gitignored, auto-merged at startup |
| `test_prompt.py` | Prompt testing harness — replay individual agent stages against completed runs |
| `src/core/llm_interface.py` | LLM API abstraction (Anthropic, OpenAI, OpenRouter) |
| `src/core/agent_subprocess.py` | Shared Popen + filesystem-polling runner for CLI agent subprocesses |
| `src/core/pipeline_ui.py` | Step tracking, LLM interaction recording, HTML dashboard |
| `src/core/utils.py` | `PathResolver` singleton, `load_config()` (merges local override), utilities |
| `src/core/interactive.py` | Interactive mode: feedback loops, revision calls |
| `src/publish/future_science.py` | Publish a completed project to Future Science (`python main.py publish`) |
| `citation_graph/literature_search/` | Literature search: sampling, download, agent questions |
| `src/questions/` | Question generation + prioritization (agent and legacy) |
| `src/analysis/` | Analysis pipeline (agent and legacy) |
| `src/visualization/` | Visualization pipeline (agent and legacy) |
| `src/reporting/` | Report generation + report revision |
| `src/autocritique/` | AutoCritique + per-recommendation revision |
| `src/repo/` | Repo assembly agent |
| `src/notebook/` | Notebook generation agent |
| `prompts/*.yaml` | Agent-specific prompt templates |
| `.last_llm.json` | Persisted provider/model selection from last run |
| `.user_options.json` | Persisted user option overrides (gitignored) |
| `.user_manual_models.json` | Persisted per-agent model overrides (gitignored) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akozlo/AutoInterp](https://github.com/akozlo/AutoInterp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
