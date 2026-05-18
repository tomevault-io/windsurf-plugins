---
trigger: always_on
description: > **Audience**: LLM-driven engineering agents and human developers
---

# FastMCP Development Guidelines

> **Audience**: LLM-driven engineering agents and human developers

> **Note**: `AGENTS.md` is a symlink to this file. Edit `CLAUDE.md` directly.

FastMCP is a comprehensive Python framework (Python ≥3.10) for building Model Context Protocol (MCP) servers and clients. This is the actively maintained v2.0 providing a complete toolkit for the MCP ecosystem.

## Required Development Workflow

**CRITICAL**: Always run these commands in sequence before committing.

```bash
uv sync                              # Install dependencies
uv run pytest -n auto                # Run full test suite
```

In addition, you must pass static checks. This is generally done as a pre-commit hook with `prek` but you can run it manually with:

```bash
uv run prek run --all-files          # Ruff + Prettier + ty
```

**Tests must pass and lint/typing must be clean before committing.**

## Repository Structure

| Path              | Purpose                                |
| ----------------- | -------------------------------------- |
| `fastmcp_slim/fastmcp/` | Library source code                    |
| `├─server/`       | Server implementation                  |
| `│ ├─auth/`       | Authentication providers               |
| `│ └─middleware/` | Error handling, logging, rate limiting |
| `├─client/`       | Client SDK                             |
| `│ └─auth/`       | Client authentication                  |
| `├─tools/`        | Tool definitions                       |
| `├─resources/`    | Resources and resource templates       |
| `├─prompts/`      | Prompt templates                       |
| `├─cli/`          | CLI commands                           |
| `└─utilities/`    | Shared utilities                       |
| `tests/`          | Pytest suite                           |
| `docs/`           | Mintlify docs (gofastmcp.com)          |

## Core MCP Objects

When modifying MCP functionality, changes typically need to be applied across all object types:

- **Tools** (`src/tools/`)
- **Resources** (`src/resources/`)
- **Resource Templates** (`src/resources/`)
- **Prompts** (`src/prompts/`)

**Before writing cross-component logic (dedupe, grouping, lookups, identity checks), read `FastMCPComponent` in `fastmcp_slim/fastmcp/utilities/components.py`.** The base class defines the shared surface — `name`, `version`, `tags`, `meta`, and critically the `key` property which is the canonical MCP identity (encodes type, identifier, and version). Prefer `item.key` over ad-hoc `name or uri or uri_template` fallbacks; overrides in `Resource` and `ResourceTemplate` already handle URI-based identity, and `.key` includes the version suffix so variants of the same component don't falsely collide.

## Development Rules

**Read `CONTRIBUTING.md` before opening issues or PRs.** It describes when PRs are appropriate, what we expect from enhancement proposals, and what we'll close without review.

### Git & CI

- Prek hooks are required (run automatically on commits)
- Never amend commits to fix prek failures
- Apply PR labels: bugs/breaking/enhancements/features
- Improvements = enhancements (not features) unless specified
- **NEVER** force-push on collaborative repos
- **ALWAYS** run prek before PRs
- **NEVER** create a release, comment on an issue, or open a PR unless specifically instructed to do so.
- **NEVER** merge a PR marked as do-not-merge or draft. Check title, body, AND labels for `[DNM]`, `DNM`, `DO NOT MERGE`, `DON'T MERGE`, `DONT MERGE`, `do-not-merge`, `dont-merge`, `[DRAFT]`, or `DRAFT` (case-insensitive, any variation — some authors use `[DRAFT]` in the title even when `isDraft` is false). Authors use these as hard stops — respect them even if CI is green and review looks clean. When triaging a batch of PRs, filter these out up front AND re-check each one's labels immediately before merging, since labels can change mid-session.
- **ALWAYS** read review-bot comments before approving a PR. CodeRabbit and chatgpt-codex-connector (Codex) leave substantive review comments on most PRs in this repo — these bots have read the diff and often flag real issues that aren't in the PR description. Use `gh pr view <num> --comments` and read the bot feedback as part of review. Unlike proposed solutions from issue reporters, review-bot feedback should be evaluated on its merits, not discounted.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PrefectHQ/fastmcp](https://github.com/PrefectHQ/fastmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
