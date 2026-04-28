---
trigger: always_on
description: > Pipeline for converting merged GitHub pull requests into Harbor evaluation tasks.
---

# AGENTS.md - SWE-gen CLI

> Pipeline for converting merged GitHub pull requests into Harbor evaluation tasks.

## Overview

SWE-gen CLI automates the creation of [Harbor](https://github.com/laude-institute/harbor) tasks from real-world bug fixes in open-source repositories. The pipeline:

1. Takes a merged GitHub PR that fixes a bug
2. Reverses the PR to recreate the buggy state
3. Uses Claude Code to detect language and complete the task skeleton
4. Validates that tests fail on the buggy baseline (NOP agent)
5. Validates that tests pass after applying the fix (Oracle agent)
6. Produces a fully containerized, reproducible evaluation task

**Supported Languages:** Any language (Python, JavaScript, TypeScript, Go, Rust, Ruby, Java, etc.)

The pipeline is **language-agnostic** - Claude Code analyzes the repository to automatically detect the language, runtime, build system, and test framework.

## Installation

```bash
uv pip install -e .
```

**Requirements:**
- Python 3.12+
- Docker
- uv
- [Claude Code CLI](https://github.com/anthropics/claude-code)
- GitHub token (for API access)
- OpenAI API key (for PR evaluation)

**Environment variables (.env):**
```bash
export GITHUB_TOKEN=<token>
export OPENAI_API_KEY=<key>
export ANTHROPIC_API_KEY=<key>  # or Claude Code OAuth
```

## CLI Commands

Entry point: `swegen` (defined in `src/swegen/cli.py`)

### `swegen create`
Generate a single Harbor task from a merged PR.

```bash
swegen create --repo <owner/repo> --pr <number>
```

Key options:
- `--cc-timeout`: Timeout for Claude Code session in seconds (default: 3200)
- `--no-validate`: Skip Harbor validation
- `--no-require-issue`: Allow PRs without linked issues
- `--no-require-minimum-difficulty`: Skip 3+ file requirement
- `--no-cache`: Disable reusing cached artifacts from previous tasks
- `--force`: Bypass local dedupe and regenerate

### `swegen farm`
Continuously process PRs from a repository's entire history.

```bash
swegen farm fastapi/fastapi
swegen farm fastapi/fastapi --resume-from 2024-01-15
```

Key options:
- `--dry-run`: Preview without generation
- `--no-require-issue`: Allow PRs without linked issues (default requires issue)
- `--reset`: Start from beginning
- `--timeout`: Timeout per PR in seconds (default: 300)
- `--cc-timeout`: Claude Code session timeout (default: 3200)
- `--task-delay`: Delay between tasks in seconds (default: 60)
- `--no-validate`: Skip Harbor validation
- `--skip-list PATH`: Path to file with task IDs to skip (one per line)

### `swegen validate`
Validate existing Harbor tasks.

```bash
swegen validate tasks/<task_id>
swegen validate tasks/  # Batch mode
```

### `swegen analyze`
Run multiple agent trials and analyze task quality.

```bash
swegen analyze tasks/<task_id> -k 3 -a claude-code
swegen analyze tasks/<task_id> -k 5 -n 3  # run 5 trials, 3 concurrent
```

Key options:
- `-k, --n-trials`: Number of trials to run (default: 3)
- `-n, --n-concurrent`: Number of concurrent trials (default: 3)
- `--analysis-model`: Model for Claude Code classification (default: claude-sonnet-4-5)
- `--skip-baseline`: Skip baseline validation (nop/oracle)
- `--skip-classify`: Skip AI-powered trial classification

**Note**: For programmatic access to classification and verdict synthesis (e.g., CI integration), use the library directly:

```python
from swegen.analyze import classify_trial, compute_task_verdict

# Classify a single trial (simplest API)
classification = classify_trial("path/to/trial", "path/to/task")
print(classification.classification)  # GOOD_SUCCESS, BAD_FAILURE, etc.

# Compute verdict from multiple classifications
verdict = compute_task_verdict([classification1, classification2, ...])
print(verdict.is_good, verdict.primary_issue)
```

---

## Architecture

```
src/swegen/
├── cli.py                  # Typer CLI entry point
├── config.py               # Configuration dataclasses
├── create/                 # Core task generation logic
│   ├── orchestrator.py     # PRToHarborPipeline - main orchestrator
│   ├── create.py           # run_reversal() - CLI command implementation
│   ├── pr_fetcher.py       # GitHub API interactions
│   ├── repo_cache.py       # Local git repo caching
│   ├── task_skeleton.py    # Language-agnostic skeleton generation
│   ├── task_instruction.py # PR evaluation and instruction generation
│   ├── claude_code_runner.py   # Claude Code integration
│   ├── claude_code_utils.py    # Claude Code utilities
│   ├── task_reference.py   # Cache successful tasks for reuse
│   ├── diff_utils.py       # Git diff utilities
│   └── utils.py            # Utility functions and test file detection
├── analyze/                # Task quality analysis
│   ├── run.py              # run_analyze() - main analysis orchestrator
│   ├── classifier.py       # AI-powered failure classification
│   ├── models.py           # Pydantic models for analysis
│   ├── classify_prompt.txt # Prompt for failure classification
│   └── verdict_prompt.txt  # Prompt for solution verdict
├── farm/                   # Continuous PR farming
│   ├── stream_farm.py      # StreamFarmer - main farming loop
│   ├── farm_hand.py        # Per-PR processing logic
│   ├── fetcher.py          # StreamingPRFetcher - GitHub PR streaming

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abundant-ai/SWE-gen](https://github.com/abundant-ai/SWE-gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
