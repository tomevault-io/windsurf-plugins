---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Playwright Failure Analyzer is a GitHub Action that automatically analyzes Playwright test failures and creates comprehensive, well-formatted GitHub issues with optional AI-powered insights. It's built as a composite action using Python 3.11.

## Core Architecture

### Two-Phase Execution Model

The action runs in two distinct phases orchestrated by `action.yml`:

1. **Parse Phase** (`src/parse_report.py`):
   - Reads Playwright JSON report from configurable path (default: `playwright-report/results.json`)
   - Extracts failure information into structured `TestFailure` dataclasses
   - Limits failures to `max-failures` parameter (default: 3)
   - Writes intermediate JSON summary to `/tmp/failure_summary.json`
   - Sets GitHub Actions output: `failures-count`

2. **Issue Creation Phase** (`src/create_issue.py`):
   - Reads failure summary from temporary file
   - Optionally runs AI analysis via LiteLLM (supports OpenAI, Anthropic, OpenRouter, DeepSeek)
   - Formats comprehensive GitHub issue with markdown
   - Creates/updates GitHub issue via REST API
   - Sets outputs: `issue-number`, `issue-url`

### Key Components

- **Error Handling** (`src/error_handling.py`): Centralized error management with severity levels, error codes, and validation decorators. All errors use `ActionError` with suggestions for resolution.

- **AI Analysis** (`src/ai_analysis.py`): Optional LiteLLM integration that analyzes failure patterns, provides root cause analysis, and suggests fixes. Only runs if API keys are configured.

- **Utilities** (`src/utils.py`): Shared helpers for ANSI stripping, GitHub context parsing, duration formatting, and path manipulation.

## Development Commands

### Running Tests

```bash
# Run all tests with the test runner
python tests/run_tests.py

# Run specific test file
python -m unittest tests/test_parse_report.py

# Run with verbose output
python -m unittest discover -v tests/
```

### Code Quality

```bash
# Format code (Black)
black --line-length=100 src/ tests/

# Sort imports (isort)
isort --profile black --line-length 100 src/ tests/

# Lint code (Flake8)
flake8 src/ --max-line-length=100 --ignore=E501,W503

# Type checking (mypy)
mypy src/ --ignore-missing-imports --no-strict-optional

# Security scanning (Bandit)
bandit -r src/ -f screen
```

### Pre-commit Hooks

The repository uses extensive pre-commit hooks for security and quality:

```bash
# Install hooks
pre-commit install

# Run all hooks manually
pre-commit run --all-files

# Run specific hook
pre-commit run detect-secrets --all-files
```

**Security-first approach**: Secret scanning (detect-secrets, gitleaks) and security linting (Bandit) run before code quality checks.

## Testing Locally

### Simulating the Action Locally

```bash
# 1. Create a test Playwright report
cd /path/to/test-project
npx playwright test --reporter=json > test-results.json

# 2. Run parse phase
python src/parse_report.py \
  --report-path test-results.json \
  --max-failures 5 \
  --output-file /tmp/failure_summary.json

# 3. Set environment variables for GitHub context
export GITHUB_REPOSITORY="owner/repo"
export GITHUB_TOKEN="your-token"
export GITHUB_SHA="abc123"
export GITHUB_RUN_ID="123456"
export GITHUB_WORKFLOW="Test"
export GITHUB_ACTOR="username"
export GITHUB_SERVER_URL="https://github.com"

# 4. Run issue creation phase (dry-run without actual issue creation)
python src/create_issue.py \
  --summary-file /tmp/failure_summary.json \
  --issue-title "Test Failures" \
  --issue-labels "bug,test" \
  --deduplicate false \
  --ai-analysis false
```

### Testing AI Analysis

```bash
# Set AI provider credentials
export OPENROUTER_API_KEY="sk-or-v1-..."
export AI_MODEL="openrouter/deepseek/deepseek-chat"

# Run with AI enabled
python src/create_issue.py \
  --summary-file /tmp/failure_summary.json \
  --ai-analysis true \
  ...
```

## Important Patterns & Conventions

### Error Handling

Always use the error handling system for consistent user feedback:

```python
from error_handling import ActionError, ErrorCodes, ErrorSeverity

raise ActionError(
    code=ErrorCodes.FILE_NOT_FOUND,
    message=f"Report file not found: {path}",
    severity=ErrorSeverity.HIGH,
    suggestions=[
        "Ensure Playwright tests have run",
        "Check the report path configuration"
    ]
)
```

### Dataclass Usage

The codebase uses dataclasses extensively for structured data:

```python
@dataclass
class TestFailure:
    test_name: str
    file_path: str
    line_number: Optional[int]
    error_message: str
    stack_trace: str
    duration: float
    retry_count: int
```

### GitHub Actions Output

Set outputs using the new GitHub Actions format (not deprecated `::set-output::`):

```python
from utils import set_github_output

set_github_output("issue-number", "42")
```

### AI Analysis Integration

AI analysis is always optional and gracefully degrades if not available:

```python
try:
    from ai_analysis import analyze_failures_with_ai
    AI_ANALYSIS_AVAILABLE = True
except ImportError:
    AI_ANALYSIS_AVAILABLE = False
```

## File Structure

```
src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [decision-crafters/playwright-failure-analyzer](https://github.com/decision-crafters/playwright-failure-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
