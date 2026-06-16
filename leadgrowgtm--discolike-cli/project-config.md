---
trigger: always_on
description: Python CLI for the DiscoLike B2B company discovery API. Wraps all endpoints with cost tracking, dual output, and agent-native design.
---

# discolike-cli

Python CLI for the DiscoLike B2B company discovery API. Wraps all endpoints with cost tracking, dual output, and agent-native design.

## Quick Start

```bash
pip install -e ".[dev]"
export DISCOLIKE_API_KEY="dk_..."
discolike --version
```

## Architecture

- **Entry point:** `src/discolike/cli.py` (Click group)
- **HTTP client:** `src/discolike/client.py` (httpx with retry/backoff)
- **Models:** `src/discolike/types.py` (Pydantic v2 for all API shapes)
- **Cost tracking:** `src/discolike/cost.py` (per-call + session totals)
- **Output:** `src/discolike/output.py` (Rich tables / JSON / CSV)
- **Cache:** `src/discolike/cache.py` (SQLite at ~/.discolike/cache.db)
- **Commands:** `src/discolike/commands/` (one file per command group)
- **Exporters:** `src/discolike/exporters/` (CSV/JSON writers)

## Key Patterns

1. **Shared filters:** `count` and `discover` share filter options via Click decorator
2. **stderr for progress, stdout for data:** never mix progress with parseable output
3. **Exit codes 0-6:** each maps to a specific failure mode (see PRD US-016)
4. **Cost on every call:** displayed in footer (table) or `_meta.cost` (JSON)
5. **Cache with TTL:** account-status (1h), extract (90d), profile/score (7d)

## Dev Commands

```bash
pytest tests/ -v --cov=discolike    # Run tests
ruff check src tests                 # Lint
mypy src                             # Type check
```

## Testing

- Mock httpx with `respx` library
- CLI tests via `click.testing.CliRunner`
- Fixtures in `tests/fixtures/` (real API response shapes)
- No live API calls in CI (gated behind DISCOLIKE_API_KEY env var)

## API Reference

- Auth header: `x-discolike-key`
- Base URL: `https://api.discolike.com/v1/`
- Full field reference: `reference/discolike-field-reference.md`
- Workflow reference: `reference/discolike-workflow.md`
- Official docs: https://api.discolike.com/v1/docs/

## PRD

Full product requirements: `PRD.md` (16 user stories, functional requirements, architecture)

<!-- GSD:project-start source:PROJECT.md -->
## Project

**DiscoLike CLI v2 — OLM Feedback Loop & AI Discovery**

Upgrade the DiscoLike CLI from a one-shot discovery tool into a full discovery-to-action pipeline. Adds the OLM feedback loop (query plan confirmation and iterative refinement), ICP validation, AI-powered enrichment via DiscoGen, and auto-segmentation. Paired with an interactive multi-dimensional discovery skill that guides users through building precise lookalike queries across product, buyer, industry, size, and geo dimensions.

**Core Value:** The feedback loop between Claude Code's client context and DiscoLike's search model — both models iterate on the query plan until convergence, then run full TAM queries with confidence.

### Constraints

- **API**: Build against documented endpoints only — no undocumented/beta endpoints
- **Async operations**: DiscoGen, Validate ICP, and Segment are all async (task_id → poll → results). Need consistent async task management pattern
- **Rate limits**: 5 req/min on discover (Starter), 2 req/min on segment. Feedback loop iterations consume quota
- **Cost awareness**: Every API call costs money. Feedback loop iterations should be explicit about cumulative cost
- **Backwards compatibility**: Existing `discover` command behavior unchanged without `--confirm` flag
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Existing Stack (Do Not Revisit)
| Technology | Version | Role |
|------------|---------|------|
| Python | 3.11+ | Runtime |
| Click | 8.1.x | CLI framework |
| httpx | 0.27+ | HTTP client (sync) |
| Rich | 13.x | Tables, progress, console output |
| Pydantic v2 | 2.x | API response models |
| PyYAML | 6.x | Config |
| SQLite | stdlib | Local cache |
## New Stack Additions
### 1. Interactive Prompts — questionary 2.1.1
- **vs. Click's built-in prompts** — Click's `click.prompt()` and `click.confirm()` are adequate for single yes/no gates but have no multi-select, no styled selection menus, and no in-place edit flow. Building an iterative query review UI on raw Click would require 200+ lines of custom prompt logic. questionary collapses that to ~20 lines.
- **vs. InquirerPy** — InquirerPy has richer styling options but its last PyPI release was 2023. Maintenance is inactive. questionary 2.1.1 shipped August 2025 and is actively maintained.
- **vs. python-inquirer** — Unix-only (blessed library dependency), experimental Windows. Disqualified — this CLI runs on Windows (confirmed from env context).
- **vs. prompt_toolkit directly** — questionary IS a thin wrapper over prompt_toolkit 3.x. Using raw prompt_toolkit for this problem would be over-engineering. questionary exposes exactly the API shape needed: `select`, `checkbox`, `text`, `confirm`.
# Query plan field-by-field review
# Phrase match multi-select (accept/drop individual phrases)
# Free-text edit for icp_text override
# Final confirm before full TAM query (rate limit + cost gate)
### 2. Async Task Polling — no new library, pure stdlib pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeadGrowGTM/discolike-cli](https://github.com/LeadGrowGTM/discolike-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
