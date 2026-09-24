---
trigger: always_on
description: SignalForce is an open-source collection of Codex agent skills and n8n workflows for signal-based outbound sales. Configure it for any ICP by editing `config/config.yaml` and `config/gtm-context.md`.
---

# SignalForce

## Project Overview

SignalForce is an open-source collection of Codex agent skills and n8n workflows for signal-based outbound sales. Configure it for any ICP by editing `config/config.yaml` and `config/gtm-context.md`.

**Three-layer architecture:**
- **Skills** (`skills/*/SKILL.md`) — Codex instruction files. The primary user interface. Each skill tells Codex how to perform a GTM task.
- **Scripts** (`scripts/*.py`) — Python modules that hit external APIs (GitHub, Semantic Scholar, HF Hub, etc.). Skills invoke these as tools.
- **n8n Workflows** (`n8n-workflows/*.json`) — Scheduled automation that runs autonomously (daily signal scans, enrichment pipelines, CRM sync).

## Directory Structure

```
config/           — Your active ICP config (gitignored; copy from examples/ or run /setup)
config.example/   — Annotated reference config (committed; documents all config options)
examples/         — Pre-built ICP configs for four verticals (rl-infrastructure, cybersecurity, data-infra, devtools)
skills/           — Codex SKILL.md files (13 skills)
scripts/          — Python modules for API interactions and data processing
scripts/scanners/ — Per-source scanner modules (github_scanner, arxiv_scanner, etc.)
n8n-workflows/    — JSON workflow definitions for n8n automation
templates/        — Reusable email sequences and scoring rubrics
docs/             — Architecture, setup guide, cost analysis, results framework
tests/            — pytest test suite (unit + integration)
tasks/            — Current work tracking (todo.md, lessons.md)
```

## Conventions

### Python
- **Pydantic models** for all data structures (`scripts/models.py`). Never use raw dicts for structured data.
- **Type hints** required on all function signatures.
- **Ruff** for formatting and linting (line-length 100, target Python 3.11).
- **Immutability**: All Pydantic models use `frozen=True`. Never mutate data — return new objects.
- Every script must be importable as a module AND runnable as CLI (`if __name__ == "__main__"`).

### Error Handling
- All API calls must handle rate limits (429/403), timeouts, and auth failures.
- Log errors with full context (URL, status code, response body).
- Never silently swallow errors.
- Fail fast: validate API keys at scan start, not mid-scan.

### Environment Variables
- All API keys loaded via `python-dotenv` from `.env`. Never hardcode secrets.
- `.env.example` documents all variables (no real values).
- `scripts/config.py` centralizes API key loading and validation (env vars only).
- `scripts/config_loader.py` loads the ICP config from `config/config.yaml` (separate from env vars).

### Skills
- Follow `superpowers:writing-skills` format. YAML frontmatter with `name` and `description` only.
- Description starts with "Use when..." and never summarizes the workflow.
- Under 500 words. Token-efficient.

### Testing
- **TDD required**: Write tests first (RED), implement (GREEN), refactor (IMPROVE).
- **pytest** with 80% minimum coverage.
- Mock all HTTP calls in tests — never hit real APIs.
- Test fixtures live in `tests/fixtures/`.

## Commands

```bash
# Run tests
pytest --cov=scripts --cov-report=term-missing -v

# Format
ruff format .

# Lint
ruff check . --fix

# Run all configured scanners via runner
python -m scripts.scanner_runner --lookback-days 7 --output /tmp/signals/

# Run a specific scanner directly
python -m scripts.scanners.github_scanner --lookback-days 7 --output /tmp/github.json

# Run signal stacker
python -m scripts.signal_stacker --inputs scan1.json scan2.json --output stacked.json
```

## Key Files

- `config/gtm-context.md` — Your active ICP definitions, product positioning, voice/tone rules. Loaded by all skills. (gitignored; copy from `config.example/` or `examples/`)
- `config/config.yaml` — Scanner keywords, ICP tier definitions, scoring thresholds. (gitignored)
- `config.example/` — Annotated reference showing all available config options.
- `scripts/models.py` — Pydantic data models (Signal, CompanyProfile, Contact, Deal, etc.)
- `scripts/config.py` — API key env var loading and validation
- `scripts/config_loader.py` — ICP config loading from `config/config.yaml`
- `scripts/scanner_runner.py` — Discovers and runs all configured scanners
- `scripts/api_client.py` — Base API client with retry/rate-limit handling
- `templates/scoring-rubrics/icp-scoring-model.md` — Weighted ICP scoring criteria

## External APIs

| API | Purpose | Key Required |
|-----|---------|-------------|
| GitHub API | Repo detection (keywords from config.yaml) | Yes (GITHUB_TOKEN) |
| Semantic Scholar | ArXiv paper author affiliation mapping | Optional (rate limited without) |
| Hugging Face Hub | Model upload detection | No (public API) |
| Apollo.io | Contact enrichment | Yes |
| Hunter.io | Email pattern lookup | Yes |
| Prospeo | LinkedIn email enrichment | Yes |
| Instantly.ai | Cold email sequencing | Yes |
| HubSpot | CRM pipeline | Yes |
| ZeroBounce | Email validation | Yes |

---

# Bookkeeping

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sami2919/SignalForce](https://github.com/sami2919/SignalForce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
