---
trigger: always_on
description: generates Playwright Python test code via GPT-4.
---

# AISquare Studio AutoQA — Repository Custom Instructions

<!-- This file is automatically read by GitHub Copilot when working in this
     repository. It provides project-specific context so that Copilot and AI
     agents can work efficiently without redundant codebase exploration. -->

## Project Overview

**AISquare Studio AutoQA** is an AI-powered GitHub Action that converts natural
language test descriptions in pull request bodies into fully automated Playwright
tests. It uses CrewAI multi-agent orchestration and OpenAI GPT-4.

| Attribute        | Value                                            |
| ---------------- | ------------------------------------------------ |
| Language          | Python 3.11+                                     |
| Action type       | Composite (`action.yml`)                         |
| AI framework      | CrewAI (multi-agent)                             |
| LLM              | OpenAI GPT-4                                     |
| Browser engine    | Playwright (Chromium)                            |
| Test framework    | pytest                                           |
| License          | Apache 2.0                                       |
| Repository       | `AISquare-Studio/AISquare-Studio-QA`             |

### What it does

1. Developer writes numbered test steps in a PR description inside a fenced
   `` ```autoqa `` block with metadata (`flow_name`, `tier`, `area`).
2. The GitHub Action triggers on PR open/edit/sync.
3. AutoQA parses metadata and steps, generates Playwright Python tests via CrewAI agents.
4. Generated code is AST-validated and executed against the staging environment.
5. On pass, the test file is committed to `tests/autoqa/{tier}/{area}/test_{flow_name}.py`.
6. Results and screenshots are posted as a PR comment.

### Execution modes

| Mode             | Behavior                                                                   |
| ---------------- | -------------------------------------------------------------------------- |
| `generate`       | Parse PR, generate a new test, execute, commit on success (default)        |
| `suite`          | Run existing test suite only (regression)                                  |
| `all`            | Generate a new test **and** run the full suite                             |
| `auto-criteria`  | AI generates test criteria from PR diff for developer review               |
| `gap-driven`     | Uses memory coverage gaps to generate criteria for uncovered modules       |
| `gap-analysis`   | Scans for present/missing test workflows, persists to SQLite DB            |

---

## Repository Layout

```
AISquare-Studio-QA/
├── action.yml                     # GitHub Action composite definition (entry point for consumers)
├── qa_runner.py                   # Local test runner CLI entry point
├── requirements.txt               # Python runtime dependencies
├── pyproject.toml                 # Python project config (black, isort, pytest)
├── pytest.ini                     # Pytest config (markers, paths)
├── env.template                   # Environment variables template
├── .flake8                        # Flake8 linting config
│
├── config/
│   ├── autoqa_config.yaml         # Master AutoQA policy (tiers, quotas, execution, security)
│   └── test_data.yaml             # Test scenarios and selectors
│
├── src/
│   ├── __init__.py
│   ├── agents/
│   │   ├── planner_agent.py       # CrewAI agent: generates Playwright code from steps
│   │   ├── executor_agent.py      # CrewAI agent: validates (AST) and executes code
│   │   └── step_executor_agent.py # Active execution: one step at a time with live browser
│   ├── autoqa/
│   │   ├── action_runner.py       # Main GitHub Action orchestrator (reads env vars, drives flow)
│   │   ├── parser.py              # PR body parser: extracts autoqa block, metadata, steps
│   │   ├── action_reporter.py     # Generates PR comment with results/screenshots
│   │   ├── cross_repo_manager.py  # Commits generated test files across repos
│   │   ├── criteria_generator.py  # Proposal 16: auto-generates test criteria from PR diffs
│   │   ├── dashboard_results.py   # Produces dashboard JSON schema output
│   │   ├── gap_analysis_db.py     # SQLite-backed gap analysis persistence
│   │   ├── gap_driven_generator.py# Gap-driven test criteria generation
│   │   ├── memory_tracker.py      # Tracks test status and coverage over time
│   │   └── testid_scanner.py      # Scans for data-testid attributes
│   ├── crews/
│   │   └── qa_crew.py             # CrewAI agent orchestration / crew definition
│   ├── execution/
│   │   ├── iterative_orchestrator.py  # Step-by-step execution coordinator
│   │   ├── execution_context.py       # State tracking between execution steps
│   │   └── retry_handler.py           # Failure analysis and retry logic
│   ├── tools/
│   │   ├── playwright_executor.py     # Playwright test code execution engine
│   │   └── dom_inspector.py           # Live page selector discovery (DOMInspectorTool)
│   ├── templates/
│   │   └── test_execution_template.py # Runtime template injected into generated tests
│   └── utils/
│       ├── logger.py                  # GitHub Actions-aware structured logging
│       ├── github_comment_client.py   # GitHub API client for PR comments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AISquare-Studio/AISquare-Studio-QA](https://github.com/AISquare-Studio/AISquare-Studio-QA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
