---
trigger: always_on
description: **Note**: This is a reference export from a working Claude Code project. These agent
---

# 📚 Reference Export - Agents Configuration

**Note**: This is a reference export from a working Claude Code project. These agent
configurations and guidelines may contain project-specific references that need adaptation.

Customize the following for your project:
- Project-specific paths ($PROJECT_ROOT/ → your main source directory)
- Repository and domain references
- CI commands and test runner paths

---

# Repository Guidelines (Compact Core)

## Purpose
This is the enforceable core contract for agents working in this repo. Keep this file concise; place detailed procedures in `.claude/skills/` or `.codex/skills/` and link them here.

## Non-Negotiables
- Preserve existing `GITHUB_TOKEN=...` lines in the user's real crontab unless explicitly asked to modify them.
- Do not use sparse checkout in this repository.
- Keep `.beads/` tracked and include beads changes in PRs.
- Use `gh` for GitHub operations (PRs/issues/checks/releases).
- Never delete/remove the worktree directory you are currently operating in (or the user's stated active worktree) unless explicitly asked.
- At session end: quality checks + push branch updates; work is not complete until push succeeds.

## LLM Architecture Principles
### Core rule
- LLM decides, server executes.
- Provide full context to the LLM for decision-making.
- Execute tools/state changes server-side.
- Incorporate actual tool results in final output.

### Banned patterns
- Keyword intent bypasses that skip LLM judgment.
- Stripping tool definitions based on predicted need.
- Pre-computing results the LLM should request.
- "Optimization" that removes required context.
- Requested features hidden behind disabled-by-default flags.

### Prefer schemas
- Prefer structured JSON input/output schemas over prose templates.

## File Protocol (Required)
Before touching any file, document:
- GOAL
- MODIFICATION
- NECESSITY
- INTEGRATION PROOF

Default to integrating into existing files. New file creation is last resort.
Integration order:
1. Existing similar file
2. Existing utility
3. Existing `__init__.py`
4. Existing test file
5. Existing class method
6. Config file
7. New file only if justified

## Python Logging & Imports
- In `$PROJECT_ROOT/`, use unified logger: `from mvp_site import logging_util`.
- Keep imports module-level (no inline function imports outside tests).

## Skills Discovery
Before starting work, check relevant skills in:
- Personal: `~/.claude/skills/`
- Project: `.claude/skills/`

When conflicts exist, personal skills win.
Use matching skills automatically based on request context.

## Project Structure & Module Organization
- `$PROJECT_ROOT/`: Python 3.11 backend (Flask, MCP tooling), tests, templates, and static assets. Frontends live in `$PROJECT_ROOT/frontend_v1/` and `$PROJECT_ROOT/frontend_v2/`.
- `$PROJECT_ROOT/tests/` and `$PROJECT_ROOT/test_integration/`: Unit and integration tests. Additional top‑level tests exist in `tests/`.
- `scripts/` and top‑level `run_*.sh`: Development, CI, and utility commands.
- `.claude/` and related folders: Agent orchestration commands and docs (detailed conventions live in `docs/CLAUDE.md`).
- `docs/`, `roadmap/`, `world/`: Documentation, plans, and content assets.

## Build, Test, and Development Commands
- `./vpython $PROJECT_ROOT/main.py serve`: Run the API locally. Alt: `./run_local_server.sh`.
- `./run_tests.sh [--full|--integration|--coverage]`: Run Python tests.
- `./run_tests_with_coverage.sh`: Generate coverage; HTML output under `/tmp/$PROJECT_NAME/coverage/`.
- `./run_ui_tests.sh`: Execute UI/browser tests.
- `./run_lint.sh` or `pre-commit run -a`: Lint/format.

### testing_mcp and testing_ui execution policy
- Streaming is the primary preview/prod execution path; investigate and validate streaming first before non-streaming.
- Do not use pytest collection for script-style `testing_mcp/` suites. Run directly with `vpython`.
- Canonical: `cd testing_mcp && ../vpython test_<name>.py --server http://127.0.0.1:8001`
- Browser tests: Start server with `TESTING_AUTH_BYPASS=true`, navigate with `?test_mode=true&test_user_id=<id>`.
- For these suites, run with real services only. Mock toggles (`MCP_TEST_MODE`, `MOCK_SERVICES_MODE`, `USE_MOCK_*`) are intentionally not used here.
- `testing_mcp/` and `testing_ui/` scripts must never use mock mode paths. This is a hard policy for smoke/integration and evidence-bearing runs.

## Coding Style
- Python: 4‑space indent, 88‑char lines, double quotes. Lint/format via Ruff; imports via isort.
- JavaScript/CSS: Prettier (2‑space tabs, single quotes).
- Naming: `snake_case` for files/functions, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants.

## Testing Guidelines
- Frameworks: unittest/pytest via `run_tests.sh`. Integration tests opt‑in (`--integration`).
- Environment: Default `TEST_MODE=mock`.
- For `testing_mcp/` and `testing_ui/`, do not use mock mode. These suites should validate real MCP/UI behavior with real services.
- Conventions: Feature tests near code in `$PROJECT_ROOT/tests/`; prefer small, deterministic tests.

## Commit & Pull Request Guidelines
- Commits: Imperative mood, e.g., `Context Optimization: Reduce token usage`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jleechanorg/claude-commands](https://github.com/jleechanorg/claude-commands) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
