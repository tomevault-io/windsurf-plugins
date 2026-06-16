---
trigger: always_on
description: A Python CLI + Claude Code Skill that lets LLM agents drive our internal Atlassian Server/DC stack (Jira, Confluence, Bitbucket, Bamboo) in a token-efficient way. Serves as a complete replacement for the `mcp-atlassian` MCP server.
---

# atlassian-skills — CLAUDE.md

## Project Overview

A Python CLI + Claude Code Skill that lets LLM agents drive our internal Atlassian Server/DC stack (Jira, Confluence, Bitbucket, Bamboo) in a token-efficient way. Serves as a complete replacement for the `mcp-atlassian` MCP server.

- **Binary**: `atls`
- **Package**: `atlassian-skills`
- **Current version**: 0.2.8 (keyring + shell-command credential storage with per-product commands; keyring-only `atls setup` wizard; `atls auth status --resolve`; `atls doctor` PyPI freshness banner; legacy `setup all/codex/claude/paths/status` deprecated for removal in 0.3.0)

## Build & Run

```bash
# Install (uv recommended)
uv sync

# Run the CLI
uv run atls --help
uv run atls jira issue get PROJ-1 --format=compact

# Tests
uv run pytest                          # unit + contract + snapshot
uv run pytest -m integration           # hit live internal instance (manual)
uv run pytest tests/benchmarks         # token benchmarks

# Lint & type check
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/
```

## Project Structure

```
src/atlassian_skills/
├── cli/                    # Typer CLI (main.py = entry point, split by product)
├── core/
│   ├── client.py           # BaseClient (httpx, retry, auth, pagination)
│   ├── auth.py             # PAT/Basic, env > keyring > config
│   ├── config.py           # ~/.config/atlassian-skills/config.toml
│   ├── errors.py           # AtlasError, exit codes per §15.4
│   ├── pagination.py       # offset + link-follow patterns
│   └── format/             # compact, md, json, raw formatters
├── jira/
│   ├── client.py           # JiraClient(BaseClient)
│   ├── models.py           # pydantic response models
│   └── preprocessing.py    # mention / smart-link preprocessing (§5.1.1)
├── confluence/
│   ├── client.py
│   └── models.py
├── bitbucket/
│   ├── client.py           # BitbucketClient(BaseClient)
│   └── models.py           # pydantic response models (PR, Comment, Branch, Task, etc.)
└── bamboo/                 # planned

tests/
├── fixtures/               # MCP capture JSON (golden files)
├── unit/                   # respx mocking
├── contract/               # response ↔ pydantic models
├── snapshot/               # syrupy CLI output regression
└── benchmarks/             # tiktoken token measurements

src/atlassian_skills/_assets/skills/atls/SKILL.md   # canonical SKILL.md installed by `atls setup` for both Claude Code and Codex
docs/                       # design docs and analyses
```

## Coding Conventions

### Python style
- **Python 3.10+**, type hints required (`disallow_untyped_defs = true`)
- **ruff** formatter + linter (line-length=120)
- **pydantic v2** models (response parsing via `model_validate`)
- `from __future__ import annotations` at the top of every module
- Import order: stdlib → third-party → local (enforced by ruff isort)

### Naming
- Modules / variables: `snake_case`
- Classes: `PascalCase`
- CLI commands: `kebab-case` (Typer converts automatically)
- Constants: `UPPER_SNAKE_CASE`

### Error handling
- Wrap every API error in the `AtlasError` hierarchy
- Exit codes follow the §15.4 spec (0=OK, 2=not found, 6=auth, ...)
- With `--format=json`, errors are also emitted to stdout as a JSON envelope

### Tests
- Unit tests: mock httpx with `respx`, use fixtures/ JSON
- Snapshot tests: `syrupy` guards against CLI output regression
- Token benchmarks: `tiktoken` cl100k_base; the test fails if the budget is exceeded
- `@pytest.mark.integration` is skipped in CI; run manually

## Core Design Principles

1. **The CLI is the product** — the skill is a thin wrapper that calls it
2. **Token efficiency** — compact format is the default; ≥50% reduction at L1 vs MCP
3. **Single cfxmark dependency** — both Confluence XHTML and Jira wiki conversion go through cfxmark (>=0.4)
4. **Server/DC only** — Cloud compatibility is a non-goal
5. **Byte-preserving raw** — `--format=raw` does not alter a single byte of the server response

## Key Dependencies

| Package | Purpose |
|---|---|
| `httpx` | REST client (sync) |
| `typer` + `rich` | CLI framework |
| `pydantic` | Response models |
| `cfxmark>=0.4` | Confluence XHTML ↔ md, Jira wiki ↔ md |
| `platformdirs` | Config paths |

## Authentication (§7.1)

```bash
# Environment variables (default, recommended)
export ATLS_CORP_JIRA_TOKEN="your-pat"
export ATLS_CORP_CONFLUENCE_TOKEN="your-pat"
```

```toml
# ~/.config/atlassian-skills/config.toml

# Env vars (default)
[profiles.corp]
jira_url = "https://jira.corp.example.com"

# System keyring — uses platform native store (macOS Keychain, Windows Credential Manager, Linux Secret Service)
# The keyring package is a base dependency (bundled by default — no extra needed)
# Keyring entry: service="atls-<profile>", account="<product>_token"
[profiles.corp]
jira_url = "https://jira.corp.example.com"
storage = "keyring"

# Shell command — cross-platform; the command is whatever your OS/secret manager supports
# Examples: "op read op://vault/jira/token" (1Password, all platforms)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eunsanMountain/atlassian-skills](https://github.com/eunsanMountain/atlassian-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
