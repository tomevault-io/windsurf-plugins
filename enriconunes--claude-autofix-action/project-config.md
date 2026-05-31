---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository implements **AI-powered automated test fixing** using Claude AI (Anthropic API) integrated into GitHub Actions workflows. The system automatically analyzes failing tests in Pull Requests, generates fixes, and creates new PRs with the corrections.

**Key Concept**: The Python files in the root directory (`dividir.py`, `media.py`, etc.) are **test fixtures** used to demonstrate and validate the workflows. They are intentionally simple and contain bugs to trigger the auto-fix workflows. The real value of this repository is the **reusable GitHub Actions workflows** that can be integrated into any Python project.

## Architecture

### Workflow Execution Flow

```
1. Developer creates PR → main
2. claude-ci.yml triggers:
   - Runs pytest with JSON reporting
   - Sends failures to Claude AI for analysis
   - Posts analysis comment on PR

3. If tests fail → claude-auto-fix.yml triggers:
   - Detects the failing branch
   - Runs tests again to generate failure report
   - Sends failures to Claude AI for code fixes
   - Applies fixes to actual source files
   - Creates new fix branch (claude-auto-fix-TIMESTAMP)
   - Creates PR with fixes → original failing branch
   - Comments on original PR with link to fix PR
```

### Execution Flow Diagrams

#### GitHub Action: `claude-ci.yml` → Test Analysis

```
.github/workflows/claude-ci.yml
    │
    ├─ Runs: pytest --json-report
    │         └─ Generates: .report.json
    │
    └─ Executes: ci/claude_report.py
                    │
                    ├─ Imports: api/
                    │           ├─ client.py
                    │           │   └─ Calls: Claude API (Anthropic)
                    │           └─ models.py
                    │               └─ Resolves: Claude model name
                    │
                    ├─ Imports: pytest/
                    │           ├─ parser.py
                    │           │   └─ Loads: .report.json
                    │           │   └─ Extracts: test failures
                    │           └─ formatter.py
                    │               └─ Formats: tracebacks, responses
                    │
                    ├─ Imports: file_utils.py
                    │           └─ Reads: source files
                    │
                    ├─ Imports: config.py
                    │           └─ Provides: BASE_ANALYSIS_PROMPT
                    │
                    └─ Generates: claude_comment.md
                                  └─ Posted to PR as comment
```

#### GitHub Action: `claude-auto-fix.yml` → Automated Fix Generation

```
.github/workflows/claude-auto-fix.yml
    │
    ├─ Runs: pytest --json-report
    │         └─ Generates: .report.json
    │
    └─ Executes: ci/claude_fix.py
                    │
                    ├─ Imports: api/
                    │           ├─ client.py
                    │           │   └─ Calls: Claude API (Anthropic)
                    │           └─ models.py
                    │               └─ Resolves: Claude model name
                    │
                    ├─ Imports: pytest/
                    │           ├─ parser.py
                    │           │   └─ Loads: .report.json
                    │           │   └─ Extracts: test failures
                    │           └─ formatter.py
                    │               └─ Formats: tracebacks
                    │
                    ├─ Imports: fix/
                    │           ├─ inference.py
                    │           │   └─ Infers: test_X.py → X.py
                    │           ├─ extractor.py
                    │           │   └─ Extracts: Python code from response
                    │           └─ patcher.py
                    │               └─ Validates/applies: patches
                    │
                    ├─ Imports: file_utils.py
                    │           └─ Reads: source files
                    │
                    ├─ Imports: config.py
                    │           └─ Provides: FIX_PROMPT
                    │
                    ├─ Generates: claude-patches/
                    │             ├─ 01_response.txt (debug)
                    │             ├─ 01_<filename>.py (fixed code)
                    │             └─ summary.json
                    │
                    └─ Modifies: source files (with --apply flag)
                                 └─ Creates: fix branch + PR
```

### Python Module Structure

```
ci/
├─ claude_report.py (Entry point for analysis)
│  └─ Dependencies:
│     ├─ api.client → send_to_claude()
│     ├─ api.models → resolve_model_name()
│     ├─ pytest.parser → load_report(), extract_failures()
│     ├─ pytest.formatter → format_longrepr(), extract_response_text()
│     ├─ file_utils → read_source()
│     └─ config → BASE_ANALYSIS_PROMPT
│
├─ claude_fix.py (Entry point for auto-fix)
│  └─ Dependencies:
│     ├─ api.client → send_to_claude()
│     ├─ api.models → resolve_model_name()
│     ├─ pytest.parser → load_report(), extract_failures()
│     ├─ pytest.formatter → format_longrepr()
│     ├─ fix.inference → infer_source_file()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enriconunes/claude-autofix-action](https://github.com/enriconunes/claude-autofix-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
