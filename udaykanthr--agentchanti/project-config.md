---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AgentChanti is a multi-agent AI coding CLI tool (`agentchanti` command) and Python library (`agentchanti` package). It takes a plain English task description and autonomously plans, codes, reviews, and tests the solution using a pipeline of specialized LLM-powered agents. Supports local LLMs (Ollama, LM Studio) and cloud providers (OpenAI, Gemini, Anthropic).

## Common Commands

```bash
# Install in editable mode
pip install -e .

# Run tests
python -m pytest tests/ -v

# Run a single test
python -m pytest tests/test_flow.py -v

# Run the CLI
agentchanti "your task" --provider ollama --model deepseek-coder-v2:16b

# Run via library API
python -c "from agentchanti import run_task; run_task(task='...', auto=True)"
```

## Architecture

### Agent Pipeline

The system runs a sequential pipeline: **Planner -> Coder -> Reviewer -> Tester**. Each agent (`agentchanti/agents/`) extends `Agent` base class and calls `self.llm_client.generate_response(prompt)`. The pipeline is orchestrated in two places:
- **CLI path**: `orchestrator/cli.py:main()` — parses args, builds agents, runs the pipeline
- **Library path**: `api.py:run_task()` — programmatic entry point returning `TaskResult`

Both paths share the same execution engine in `orchestrator/pipeline.py`.

### Step Execution Flow

1. **PlannerAgent** generates numbered steps from the task description
2. `pipeline.py:build_step_waves()` groups steps into dependency waves for parallel execution
3. Each step is classified by `classification.py:_classify_step()` via LLM into: **CMD**, **CODE**, **TEST**, or **IGNORE**
4. Step handlers in `orchestrator/step_handlers.py` execute each type:
   - **CMD**: Runs shell commands via `Executor.run_command()`
   - **CODE**: Coder generates code -> Reviewer checks -> retry loop (up to 3x) -> diagnosis on failure
   - **TEST**: TesterAgent generates tests -> runs them -> Coder fixes failures
5. `orchestrator/diagnosis.py` handles failure analysis and auto-fix, driven by `_run_diagnosis_loop` (see **Diagnosis Loop** below for what it keeps and what it ships)

### Language Detection (agentchanti/language.py)

Auto-detects project language by scanning file extensions (`detect_language()`) or parsing task keywords (`detect_language_from_task()`). Maps languages to test frameworks via `TEST_FRAMEWORKS` dict. **Known issue**: defaults to Python/pytest when language is `None`, which causes incorrect test generation for non-Python projects (e.g., TypeScript projects get Python tests). The TesterAgent at lines 10-12 and 41-44 hard-defaults to Python when `language` is None.

### LLM Client Layer (agentchanti/llm/)

`LLMClient` base class with provider implementations: `OllamaClient`, `LMStudioClient`, `OpenAIClient`, `GeminiClient`, `AnthropicClient`. All expose `generate_response(prompt) -> str` with retry and streaming support.

Chat-native entry point: `chat(messages, tools=None) -> ChatResponse` (types in `llm/chat_types.py`: `Message`, `ToolDef`, `ToolCall`, `ChatResponse`). Ollama (`/api/chat`), OpenAI (`/chat/completions`) and Anthropic (Messages API) implement it natively with structured tool calling (`NATIVE_CHAT = True`); other providers fall back to flattening the conversation into a text prompt via `flatten_messages()`. Models that reject tools at runtime raise `ToolsNotSupportedError` and are downgraded to the text path for the session. Check availability with `client.supports_tools()`.

### Agent Tools (agent_tools.py)

`AgentTools` is the agent-computer interface for tool-calling loops: six `ToolDef`s (`list_files`, `read_file`, `write_file`, `edit_file`, `run_command`, `search_code`) scoped to a project root, backed by `Executor`, the KB `Searcher`, and `FileMemory`. `execute(ToolCall) -> str` never raises (errors return as strings for the model); `execute_all()` wraps results as `role="tool"` messages. `edit_file` is exact-match single-occurrence replace with `ast.parse` validation for Python; paths escaping the project root are rejected.

### Agent Loop (orchestrator/agent_loop.py)

Default execution path for CODE/TEST steps when the provider supports native tool calling (`agent_loop: true` by default; set `false` to use the classic generate→review→retry pipeline, which also remains the automatic fallback for providers without tool support). `run_agent_loop()` runs the step as a bounded tool-calling conversation (`agent_loop_max_turns`, default 8): stable byte-identical system prompt (KV-cache friendly), model edits/runs via `AgentTools`, observes real output, self-corrects. Exit rules: the final turn withholds tools to force a text summary; a `verify_cmd` (from `verify_cmd_for_language()`: pytest for Python, `npm test` when package.json defines it, `go test ./...`) must pass before the model's "done" claim is accepted, and passes on exhaustion still count as success. Returns the same `(success, error_info)` contract as the classic handlers; gate is at the top of `_handle_code_step`/`_handle_test_step` via `agent_loop_enabled()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [udaykanthr/agentchanti](https://github.com/udaykanthr/agentchanti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
