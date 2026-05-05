---
trigger: always_on
description: AI-powered journaling system with markdown entries, semantic backlinks, and LLM-powered analysis. Integrates with Obsidian or any markdown system. Uses Azure OpenAI for LLM generation.
---

# GitHub Copilot Custom Instructions

## Project Overview

AI-powered journaling system with markdown entries, semantic backlinks, and LLM-powered analysis. Integrates with Obsidian or any markdown system. Uses Azure OpenAI for LLM generation.

**Tech Stack:**
- Python 3.13+ with full type hints
- Package manager: `uv` (ALWAYS use `uv`, never `pip`)
- Testing: pytest with 57 tests, 47% coverage
- CLI: typer (add_completion=False) + rich
- LLM: Azure OpenAI or Ollama (configurable via LLM_PROVIDER)

## Architecture

Two main modules with clear separation:

### brain_core/ - Business Logic
Core functionality with comprehensive type hints and dataclasses:

- `config.py` - Configuration management (91% coverage)
- `constants.py` - Shared constants (100% coverage)
- `entry_manager.py` - Entry I/O and parsing (93% coverage)
- `llm_client.py` - Abstract LLM interface (73% coverage)
- `openai_client.py` - Unified OpenAI client for both Azure and Ollama (0% coverage)
- `llm_analysis.py` - Atomic LLM operations: entities, backlinks, tags (17% coverage)
- `report_generator.py` - Report orchestration (18% coverage)
- `template_generator.py` - AI prompt generation (42% coverage)

### brain_cli/ - CLI Interface
User-facing commands with typer:

- `main.py` - Root CLI entry point (92% coverage)
- `diary_commands.py` - Diary management (23% coverage)
- `plan_commands.py` - Daily planning with LLM task extraction (49% coverage)

## Common Commands

```bash
# Planning commands
uv run brain plan create          # Create daily plan with LLM task extraction
uv run brain plan create tomorrow # Plan for tomorrow

# Diary commands
uv run brain diary create         # Create evening reflection entry
uv run brain diary link           # Generate semantic backlinks + tags
uv run brain diary report 7       # Memory trace analysis for 7 days
uv run brain diary patterns 7     # Statistical patterns
uv run brain diary list           # List all entries
uv run brain diary refresh 30     # Regenerate links for last 30 days

# Development
uv sync                           # Install dependencies
uv add <package>                  # Add dependency
uv add --dev <package>            # Add dev dependency
uv run pytest tests/ -v           # Run tests
uv run pytest tests/ --cov        # With coverage
```

## Configuration (.env)

**Required Paths:**
- `DIARY_PATH` - Path to Obsidian vault or markdown directory (for reflection entries)
- `PLANNER_PATH` - Path to directory for daily plan files (separate from diary)

**LLM Provider (choose one):**
- `LLM_PROVIDER` - Set to "azure" or "ollama" (default: azure)

**Azure OpenAI (cloud-based):**
- `AZURE_OPENAI_API_KEY` - API key
- `AZURE_OPENAI_ENDPOINT` - Service endpoint
- `AZURE_OPENAI_DEPLOYMENT` - Model deployment name (e.g., gpt-4o)
- `AZURE_OPENAI_API_VERSION` - Default: 2024-02-15-preview
- ✅ **Full functionality:** All features, cost tracking enabled

**Ollama (local, free):**
- `OLLAMA_BASE_URL` - API URL (default: http://localhost:11434)
- `OLLAMA_MODEL` - Model name (default: llama3.1)
- ✅ **Full functionality:** All features, no cost tracking (local is free!)

## Entry Structure

Two entry types with specific formats:

**Morning Plan** (YYYY-MM-DD-plan.md):
- Saved to `PLANNER_PATH` (separate from diary)
- Action Items section ONLY
- LLM intelligently extracts actionable tasks from yesterday's diary entry:
  - Identifies incomplete/pending tasks
  - Extracts follow-ups from meetings
  - Filters out completed activities and vague intentions
- Auto-carries forward unchecked todos from yesterday's plan
- All tasks include backlinks to source entries (e.g., "from [[2025-10-14]]")
- Combines both sources (diary + plan) with deduplication
- Simple, distraction-free format for task management

**Evening Reflection** (YYYY-MM-DD.md):
- Reflection Prompts (AI-generated from past 3 calendar days)
- Brain Dump section (main content)
- Memory Links section (automatic [[backlinks]] and #tags)

**Sunday Special**: 5 weekly prompts instead of 3 daily prompts

## Code Patterns & Best Practices

### Type Safety
- Full type hints on all functions (Python 3.13+)
- Use `Literal` for string enums (e.g., `ConfidenceLevel = Literal["high", "medium", "low"]`)
- Dataclasses for structured data (e.g., `SemanticLink`, `DiaryEntry`)
- Type aliases for clarity

### Error Handling
- Specific exceptions (RuntimeError for LLM failures, ValueError for validation)
- Comprehensive logging with context
- Graceful degradation (return empty results, not crashes)

### LLM Calls
- Always use named parameters: `prompt=`, `system=`, `temperature=`, `max_tokens=`
- Add timing metrics: `start_time = time.time()` → log `elapsed`
- Clean JSON responses with `_clean_json_response()`
- Truncate text to manage token costs

### Testing
- Use pytest with fixtures in `tests/conftest.py`
- Mock environment variables with `monkeypatch`
- Integration tests via CLI commands preferred
- Run with: `uv run pytest tests/ -v`

### Constants
- All magic numbers in `brain_core/constants.py`
- Import specific constants needed
- Never hardcode values like lengths, thresholds, temperatures

### Logging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebygps/second-brain](https://github.com/madebygps/second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
