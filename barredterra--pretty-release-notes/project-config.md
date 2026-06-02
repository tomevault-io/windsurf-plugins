---
trigger: always_on
description: This is a multi-mode Python tool that rewrites GitHub's auto-generated release notes into polished, human-readable sentences using an LLM. It can be used as:
---

# Project Overview: Pretty Release Notes

## Purpose

This is a multi-mode Python tool that rewrites GitHub's auto-generated release notes into polished, human-readable sentences using an LLM. It can be used as:

1. **CLI Tool** - Traditional command-line workflow
2. **Python Library** - Programmatic API for integration
3. **Web Backend** - FastAPI service for web frontends or automation

It is tuned for ERPNext and the Frappe Framework by default, but can be adapted to other repositories by providing a custom prompt template.

## Core Functionality

- Rewrites GitHub release-note PR lines into clearer, user-facing summaries
- Supports provider-qualified LLM models through `any_llm` (for example `openai:o3`, `openai:gpt-5`, `anthropic:claude-sonnet-4-5`)
- Filters non-user-facing changes by conventional commit type, label, or author
- Detects PRs that were reverted within the same release and removes both the original PR and the revert PR from output
- Reuses cached summaries for direct backports by using the original PR number as the summary key
- Falls back to commit-based generation when PR metadata is unavailable or `force_use_commits` is enabled
- Loads reviewer information and credits authors/reviewers while excluding configured bots
- Supports optional grouping by conventional commit type, with breaking changes rendered first when present
- Stores generated summaries in CSV or SQLite caches
- Supports interactive TOML-based setup and one-time migration from legacy `.env` files
- Can infer the previous tag from the GitHub compare URL in existing release notes when one is not supplied manually

## Technology Stack

- **Language**: Python 3.11+
- **Key Libraries**:
  - `typer` - CLI framework
  - `rich` - terminal output and interactive prompts
  - `requests` - GitHub REST and GraphQL communication
  - `tenacity` - retry logic for LLM calls
  - `python-dotenv` - legacy `.env` loading and migration support
  - `any-llm-sdk[all]` - provider-agnostic LLM access
- **Optional Web Stack**:
  - `fastapi` - REST API
  - `uvicorn` - ASGI server
  - `pydantic` - request/response models
- **External APIs**:
  - GitHub REST API - repositories, PRs, commits, reviews, releases
  - GitHub GraphQL API - issues closed by a PR
  - LLM provider APIs via `any_llm`

## Architecture

The project follows a lightweight **Hexagonal Architecture** / **Ports and Adapters** approach. The core release-note generation logic is UI-agnostic and reusable across CLI, library, and web modes.

### Architecture Layers

```
┌───────────────────────────────────────────────────────┐
│                   Adapters (External)                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │   CLI    │  │  Library │  │   Web    │             │
│  │ Adapter  │  │   API    │  │   API    │             │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘             │
└───────┼─────────────┼─────────────┼───────────────────┘
        │             │             │
┌───────┼─────────────┼─────────────┼───────────────────┐
│       │          Core Domain      │                   │
│  ┌────▼─────────────▼─────────────▼─────┐             │
│  │     ProgressReporter Interface       │             │
│  │   (event-based progress reporting)   │             │
│  └──────────────────────────────────────┘             │
│  ┌──────────────────────────────────────┐             │
│  │  ReleaseNotesConfig / LLMConfig      │             │
│  │  typed configuration with validation │             │
│  └──────────────────────────────────────┘             │
│  ┌──────────────────────────────────────┐             │
│  │      ReleaseNotesGenerator           │             │
│  │   core business logic, UI-free       │             │
│  └──────────────────────────────────────┘             │
└───────────────────────────────────────────────────────┘
```

## Project Structure

```text
pretty_release_notes/                 # Main package
├── __init__.py                       # Public package exports
├── __main__.py                       # `python -m pretty_release_notes`
├── main.py                           # Typer CLI entrypoint
├── setup_command.py                  # Interactive setup / migration helpers
├── api.py                            # Library client and builder
├── generator.py                      # Core release note generation workflow
├── github_client.py                  # GitHub API wrapper
├── openai_client.py                  # LLM adapter (legacy module name)
├── database.py                       # CSV / SQLite cache backends
├── ui.py                             # Rich-based CLI UI
├── prompt.txt                        # Packaged default prompt
├── py.typed                          # PEP 561 marker
├── core/
│   ├── __init__.py
│   ├── config.py                     # Typed configuration models
│   ├── config_loader.py              # TOML, dict, and .env loaders
│   ├── execution.py                  # Parallel execution strategies
│   └── interfaces.py                 # Progress interfaces and events
├── adapters/
│   ├── __init__.py
│   └── cli_progress.py               # CLI ProgressReporter adapter
├── web/
│   ├── __init__.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [barredterra/pretty_release_notes](https://github.com/barredterra/pretty_release_notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
