---
trigger: always_on
description: This file provides guidance to agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## 🔗 CRITICAL: Anchor-Based Context Management

> **⚠️ READ THIS FIRST - This is the most important pattern in this project.**

Devussy uses **stable HTML comment anchors** for efficient context management and safe file updates. **All agents MUST use anchors** when reading/writing devplan, phase, and handoff files.

### Required Anchors

| File | Anchor | Purpose |
|------|--------|---------|
| devplan.md | `<!-- PROGRESS_LOG_START -->` / `<!-- PROGRESS_LOG_END -->` | Track completed work |
| devplan.md | `<!-- NEXT_TASK_GROUP_START -->` / `<!-- NEXT_TASK_GROUP_END -->` | Current 3-5 tasks to execute |
| phase*.md | `<!-- PHASE_TASKS_START -->` / `<!-- PHASE_TASKS_END -->` | Phase-specific tasks |
| phase*.md | `<!-- PHASE_PROGRESS_START -->` / `<!-- PHASE_PROGRESS_END -->` | Outcomes and blockers |
| handoff_prompt.md | `<!-- QUICK_STATUS_START -->` / `<!-- QUICK_STATUS_END -->` | Status snapshot |
| handoff_prompt.md | `<!-- HANDOFF_NOTES_START -->` / `<!-- HANDOFF_NOTES_END -->` | Agent handoff notes |

### How to Use Anchors

**Reading (CORRECT):**
```
Read devplan.md lines between <!-- NEXT_TASK_GROUP_START --> and <!-- NEXT_TASK_GROUP_END -->
# Result: ~100 tokens loaded
```

**Reading (WRONG):**
```
Read entire devplan.md
# Result: ~3000 tokens wasted
```

**Writing:** Always use `safe_write_devplan()` from `src/file_manager.py`:
- Creates `.bak` backup before writing
- Validates anchors exist in new content
- Refuses to overwrite if anchors missing (writes to `.tmp` instead)

### Token Budget

| File | Section | ~Tokens | When to Read |
|------|---------|---------|--------------|
| handoff.md | Progress Log | ~200 | Start of session |
| devplan.md | NEXT_TASK_GROUP | ~100 | Every turn |
| devplan.md | PROGRESS_LOG | ~100 | If needed |
| phase*.md | PHASE_TASKS | ~80 | When working on phase |

**Target: Stay under 500 tokens per turn by reading ONLY anchored sections.**

### Validation

`file_manager.py:_validate_devplan_content()` enforces:
- Header: `# Development Plan` or `## 📋 Project Dashboard`
- Phase table: `### 🚀 Phase Overview` with `| Phase |`
- Anchors: `<!-- PROGRESS_LOG_START -->` and `<!-- NEXT_TASK_GROUP_START -->`

---

## Critical Commands (Non-Obvious)
- **Test single file**: `pytest tests/unit/test_cli.py::TestGenerateDesignCommand::test_generate_design_minimal_args -v`
- **Run integration tests**: `pytest tests/integration/ -v`  
- **Test with coverage**: `pytest --cov=src --cov-report=html`
- **Interactive mode**: `python -m src.cli interactive` (single-window streaming with TUI)
- **Launch interview**: `python -m src.entry` (lightweight entry point)
- **Full pipeline**: `python -m src.cli run-full-pipeline --name "project" --languages "Python" --requirements "..."`
- **Checkpoint management**: `python -m src.cli list-checkpoints`, `python -m src.cli cleanup-checkpoints --keep 5`

## Build/Lint/Test Commands
- **Format**: `black src && isort src && flake8 src`
- **Test**: `pytest -q`
- **Install development**: `pip install -e .`

## Stack & Entry Points
- **Primary CLI**: `src/cli.py` (3331 lines) using typer framework
- **Alternative entry**: `src/entry.py` (lightweight launcher)  
- **Pipeline**: Design → Basic DevPlan → Detailed DevPlan (per-phase files) → Handoff
- **Providers**: OpenAI, Requesty, Aether, AgentRouter, Generic OpenAI-compatible

## Non-Obvious Architecture Patterns
1. **Checkpoint System**: State persistence in `.devussy_state/` - pipelines can resume from failure points using `resume_from` parameter
2. **Per-Stage LLM Overrides**: `config.design_llm`, `config.devplan_llm`, `config.handoff_llm` allow different models per pipeline stage
3. **Repository Analysis**: `src/interview/repository_analyzer.py` automatically analyzes existing codebases for context-aware questioning
4. **Safe File Operations**: `src/file_manager.py:47` - `safe_write_devplan()` creates .bak files and validates content invariants before overwriting
5. **Streaming Handler**: `src/streaming.py:152` - `StreamingHandler.create_console_handler()` enables real-time token streaming with prefixes
6. **Concurrency Control**: `src/concurrency.py` - `ConcurrencyManager` limits both API calls and parallel phase generation
7. **Code Sample Extraction**: Automatically extracts architecture patterns, test examples, and representative files from analyzed repositories

## Code Style Guidelines (Project-Specific)
- **Line length**: 88 characters (black default, enforced in pyproject.toml)
- **Import sorting**: isort with black profile (line 100 in pyproject.toml)
- **Type annotations**: Use `from __future__ import annotations` for forward references
- **Async patterns**: All heavy operations use asyncio with proper event loop management
- **Error handling**: Graceful fallbacks with informative error messages, never crash on optional features
- **File encoding**: Always use `encoding='utf-8'` for all file operations to properly handle emojis and Unicode characters

## Testing Framework & Practices
- **Framework**: pytest with asyncio support (`pytest-asyncio>=0.21.0`)
- **Test markers**: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.slow` 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mojomast/devussy](https://github.com/mojomast/devussy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
