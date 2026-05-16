---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Slopometry is a Python CLI tool that tracks and analyzes Claude Code sessions by monitoring hook invocations. It collects statistics about tool usage, timing, and errors.

## Development Setup

```bash
# Install with development dependencies
uv sync --all-extras

# The CLI is available as 'slopometry' after installation

# Configure settings (optional)
# For development: copy to project directory
cp .env.solo.example .env  # For basic session tracking
# OR
cp .env.summoner.example .env  # For advanced experimentation

mkdir -p ~/.config/slopometry
cp .env.solo.example ~/.config/slopometry/.env  # For basic session tracking
# OR
cp .env.summoner.example ~/.config/slopometry/.env  # For advanced experimentation
```

## Installation as uv Tool

When installing slopometry as a global uv tool:
```bash
uv tool install .
```

After making code changes, reinstall to update the global tool:
```bash
uv tool install . --reinstall
```


## Key Architecture

### Core Components
- **CLI** (`src/slopometry/cli.py`): Hybrid CLI with flat core commands (install, uninstall, status, latest, shell-completion) and persona subcommands (solo, summoner)
- **Database** (`src/slopometry/core/database.py`): SQLite storage with platform-specific default locations
- **Hook Handler** (`src/slopometry/core/hook_handler.py`): Script invoked by Claude Code hooks to capture events
- **Models** (`src/slopometry/core/models.py`): Pydantic models for HookEvent, SessionStatistics
- **Settings** (`src/slopometry/core/settings.py`): Pydantic-settings configuration with .env support
- **LLM Wrapper** (`src/slopometry/summoner/services/llm_wrapper.py`): AI agents for analyzing git diffs and generating user stories

### How It Works
1. `slopometry install` configures Claude Code hooks in settings.json
2. Each tool invocation automatically triggers the hook handler script
3. Events are persisted to SQLite with session IDs
4. Statistics are calculated and displayed via Rich tables/trees

## Important Implementation Details

- Session IDs are provided directly by Claude Code (no generated IDs)
- Hook handler reads JSON from stdin (provided by Claude Code)
- Tool name mapping is done via `TOOL_TYPE_MAP` in hook_handler.py
- Database uses raw SQL with migration support for flexibility
- All timestamps are stored as ISO format strings

## Dataset Protocol

The dataset protocol automatically collects diff/user story pairs for AI training and analysis:

### Automatic Collection
- Every `summoner userstorify` call automatically saves to the dataset
- Includes git diff, generated user stories, model used, and prompt template
- Default rating of 3/5 for non-interactive mode

### Interactive Rating (Optional)
- Enable with `SLOPOMETRY_INTERACTIVE_RATING_ENABLED=true`
- Prompts user to rate user stories (1-5) and provide improvement guidelines
- Designed for human oversight and quality control

### Dataset Management
- `summoner user-story-stats`: View collection statistics and rating distribution
- `summoner list-user-stories`: Browse recent entries with metadata
- `summoner user-story-export`: Export user stories to Parquet format
- All data stored in SQLite with proper indexing for performance

### Export & Sharing
Export dataset locally:
```bash
# Export to Parquet
slopometry summoner user-story-export

# Export with custom path
slopometry summoner user-story-export --output my_user_stories.parquet
```

Upload to Hugging Face:
```bash
# Export and upload in one command
slopometry summoner user-story-export --upload-to-hf --hf-repo username/dataset-name

# With configured settings (SLOPOMETRY_HF_TOKEN and SLOPOMETRY_HF_DEFAULT_REPO)
slopometry summoner user-story-export --upload-to-hf
```

The dataset is automatically tagged with: `slopometry`, `userstorify`, `code-generation`, `user-stories`

### Optional Dependencies
For dataset export functionality:
```bash
# Basic export support
uv pip install "slopometry[dataset]"

# Full Hugging Face support
uv pip install "slopometry[huggingface]"
```

## Testing a Hook Handler Change

When modifying the hook handler, test it manually using the actual Claude Code hook schema:
```bash
# Test PreToolUse hook
echo '{"session_id": "test123", "transcript_path": "/tmp/transcript.jsonl", "tool_name": "Bash", "tool_input": {"command": "ls"}}' | slopometry hook-handler

# Test PostToolUse hook
echo '{"session_id": "test123", "transcript_path": "/tmp/transcript.jsonl", "tool_name": "Bash", "tool_input": {"command": "ls"}, "tool_response": {"success": true}}' | slopometry hook-handler
```

## Adding New Tool Types

1. Add to `ToolType` enum in models.py
2. Update `TOOL_TYPE_MAP` in hook_handler.py
3. No database migration needed (sqlite-utils handles schema)

## Experiment Tracking

The experiment tracking feature includes:

### CLI Commands
- `summoner run-experiments`: Run parallel experiments across commit history
- `summoner analyze-commits`: Analyze complexity evolution between commits  
- `summoner userstorify`: Generate user stories from commit diffs using AI
- `summoner list-features`: Detect feature boundaries from merge commits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TensorTemplar/slopometry](https://github.com/TensorTemplar/slopometry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
