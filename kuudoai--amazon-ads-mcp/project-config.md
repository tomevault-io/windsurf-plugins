---
trigger: always_on
description: > **Audience**: LLM-driven engineering agents and human developers
---

# Amazon Ads MCP Development Guidelines

> **Audience**: LLM-driven engineering agents and human developers

Amazon Ads MCP is a Python framework (Python ≥3.10) for integrating Amazon Advertising API with Model Context Protocol (MCP) servers. This project provides a complete toolkit for building AI-powered advertising applications with comprehensive campaign management, reporting, and optimization capabilities.

## Do This First (for Agents)

- Ensure Python ≥3.10 and uv are installed
- `uv sync` to install dependencies
- Start the server: `docker compose up -d`
- Connect Claude to the MCP server (HTTP):
  - `claude mcp add amazon-ads-mcp -- python -m amazon_ads_mcp.server --transport http --port 9080`
- Verify: `claude mcp list` and use `/mcp` inside Claude

## Required Development Workflow

**CRITICAL**: Always run these commands in sequence before committing:

```bash
# Install dependencies
uv sync                              # Install dependencies

# Validate code
uv run ruff check --fix             # Lint and auto-fix
uv run pytest                        # Run full test suite
```

**All must pass** - tests/linting must be clean before committing.

## Agent Ops (LLM Guidance)

- Preambles: Send a brief 1–2 sentence note before running tool commands.
- Plans: Use `TodoWrite` for multi-step work; keep exactly one `in_progress` step.
- Edits: Use `Edit` or `MultiEdit` to modify files; keep changes focused and avoid unrelated edits.
- Testing: Run the smallest relevant tests first; do not fix unrelated failures.
- Sandboxing: Assume workspace-write FS and restricted network; prefer local resources over external APIs unless keys are present.

## Agent Success Playbook

Follow these steps for reliable outcomes in Claude contexts:

1) Understand & Plan
- Clarify task type: API integration, MCP connectivity, Docker, tests, or GitHub workflow.
- Post a short preamble and, for multi-step work, create a minimal `TodoWrite` with exactly one `in_progress` step.

2) Connect & Verify (MCP + Server)
- Start server: `docker compose up -d` (Amazon Ads MCP at `http://localhost:9080`).
- Add MCP to Claude (HTTP):
  - `claude mcp add amazon-ads-mcp -- python -m amazon_ads_mcp.server --transport http --port 9080`
- Verify in Claude: `claude mcp list` then `/mcp` → run a tool (e.g., list profiles).

3) Implement Safely
- Prefer the smallest viable change; touch only relevant files.
- Use `Edit` or `MultiEdit` for edits. Only run `git` if explicitly requested.
- Keep environment secrets out of logs; mask sensitive data.

4) Test Incrementally
- Lint: `uv run ruff check --fix`.
- Targeted tests first: `uv run pytest tests/unit/test_specific.py`.
- Full suite if needed: `uv run pytest`.

5) Validate Behavior
- For MCP changes: exercise tools via `/mcp` in Claude.
- For Docker changes: `docker build -t amazon-ads-mcp .` and check logs.

6) Prepare Handoff
- If commits are requested: propose a branch name and show the exact commands; otherwise provide patch summary and affected files.
- Summarize changes, risks, and next steps in your final message.

Guardrails (Never Do)
- Do not push to `main` or force-push; avoid branch deletions.
- Do not log secrets (API keys, tokens) or large payloads.
- Do not widen scope or refactor unrelated code.

Escalation Prompts (When Blocked)
- "I need permission to run networked commands/install packages; approve or provide offline alternative?"
- "Tests rely on external API keys; provide keys or allow me to skip/mark accordingly?"
- "MCP output exceeds token limits; can I raise `MAX_MCP_OUTPUT_TOKENS`?"

Claude Command Cheatsheet
- List servers: `claude mcp list`
- Add server (HTTP): `claude mcp add amazon-ads-mcp -- python -m amazon_ads_mcp.server --transport http --port 9080`
- In-session MCP menu: `/mcp`
- Read files: `/read <path>`; Edit files: `/edit <path>`

## Repository Structure

| Path             | Purpose                                                |
| ---------------- | ------------------------------------------------------ |
| `src/amazon_ads_mcp/`| Library source code (Python ≥ 3.10)                |
| `├─server/`      | MCP server implementation and FastMCP integration     |
| `├─auth/`        | Authentication providers (Direct, Openbridge)         |
| `├─tools/`       | Amazon Ads tool implementations                       |
| `├─models/`      | Pydantic models for API responses                     |
| `├─middleware/`  | Authentication, OAuth, and sampling middleware        |
| `└─utils/`       | HTTP client, OpenAPI handling, security utilities     |
| `openapi/`       | OpenAPI specifications and transformations            |
| `├─resources/`   | Individual API resource definitions                   |
| `tests/`         | Pytest test suite                                     |
| `examples/`      | Example usage and demo scripts                        |
| `docker-compose.yaml` | Docker service configuration                      |

## Claude / MCP Connectivity

- Amazon Ads MCP Server:
  - The HTTP listen port follows `PORT` in `.env` (`.env.example` uses **9080**); examples below use `http://localhost:9080`.
  - It connects to Amazon Ads API with proper authentication.
- Quick connect from host:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KuudoAI/amazon_ads_mcp](https://github.com/KuudoAI/amazon_ads_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
