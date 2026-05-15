---
trigger: always_on
description: Skill adapter enabling AI agents (Claude Code, Codex) to work with the [EODHD financial data API](https://eodhd.com/). Distributed as a Claude Code plugin (`EodHistoricalData/eodhd-claude-skills`). Version: **0.3.6**.
---

# EODHD Claude Skills

## Overview

Skill adapter enabling AI agents (Claude Code, Codex) to work with the [EODHD financial data API](https://eodhd.com/). Distributed as a Claude Code plugin (`EodHistoricalData/eodhd-claude-skills`). Version: **0.3.6**.

Primarily documentation: markdown reference files covering 72 API endpoints, 29 general guides, 8 subscription plans, analysis workflows, and a lightweight Python client.

## File Structure

```
.claude-plugin/
  marketplace.json              # Plugin manifest (name, version, skills list)
skills/eodhd-api/
  SKILL.md                      # Skill definition — triggers, workflow, supported endpoints
  references/
    general/                    # 29 general guides (symbol format, exchanges, rate limits, etc.)
    endpoints/                  # 72 individual endpoint docs (one .md per endpoint)
    subscriptions/              # 8 subscription plan docs (free → all-in-one-extended)
    workflows.md                # 4 analysis workflow patterns
  scripts/
    eodhd_client.py             # Python API client (stdlib-only, ~495 lines)
    market_cap_series.py        # Market cap time series helper
    test_investverte_*.py       # ESG endpoint test scripts (4 files)
  templates/
    analysis_report.md          # Structured output template
adapters/
  claude/eodhd-api.md           # Claude adapter guide
  codex/eodhd-api.md            # Codex adapter guide
```

## Key Components

| Component | Purpose | Location |
|-----------|---------|----------|
| SKILL.md | Skill definition (triggers, workflow, guardrails) | `skills/eodhd-api/SKILL.md` |
| eodhd_client.py | Stdlib-only Python client, 30+ endpoint support | `skills/eodhd-api/scripts/eodhd_client.py` |
| Endpoint docs | Per-endpoint API reference (params, response shape, examples) | `skills/eodhd-api/references/endpoints/` |
| General guides | Cross-cutting topics (auth, symbol format, rate limits, fundamentals) | `skills/eodhd-api/references/general/` |
| Subscription docs | Per-plan feature breakdowns (7 plans + free tier) | `skills/eodhd-api/references/subscriptions/` |
| Workflows | 4 analysis recipe patterns | `skills/eodhd-api/references/workflows.md` |
| Report template | Structured output format for analysis results | `skills/eodhd-api/templates/analysis_report.md` |
| Claude adapter | Claude-specific prompting patterns and setup | `adapters/claude/eodhd-api.md` |
| Codex adapter | Codex-specific execution flow and examples | `adapters/codex/eodhd-api.md` |

## Features

| Feature | Description | Entry Point |
|---------|-------------|-------------|
| Plugin distribution | Claude Code plugin system installation | `.claude-plugin/marketplace.json` |
| API client | 30+ endpoint Python CLI (stdlib-only, JSON stdout) | `skills/eodhd-api/scripts/eodhd_client.py` |
| Endpoint reference | 72 individual endpoint docs with params/response/examples | `skills/eodhd-api/references/endpoints/README.md` |
| General guides | Auth, symbol format, exchanges, rate limits, fundamentals | `skills/eodhd-api/references/general/README.md` |
| Analysis workflows | Historical+fundamentals, screener, event window, macro overlay | `skills/eodhd-api/references/workflows.md` |
| Multi-agent adapters | Claude and Codex environment setup guides | `adapters/` |
| Subscription reference | Per-plan feature/limit breakdowns | `skills/eodhd-api/references/subscriptions/README.md` |

## Key References

- `skills/eodhd-api/references/general/exchanges.md` — canonical list of supported exchange codes
- `skills/eodhd-api/references/general/symbol-format.md` — ticker format rules
- `skills/eodhd-api/references/general/rate-limits.md` — API quotas, Marketplace limits
- `skills/eodhd-api/references/general/update-times.md` — data update schedules
- `skills/eodhd-api/references/endpoints/README.md` — endpoint index

## Development Setup

1. Clone: `git clone https://github.com/EodHistoricalData/eodhd-claude-skills.git`
2. Set token: `export EODHD_API_TOKEN="your_token_here"`
3. Test client: `python skills/eodhd-api/scripts/eodhd_client.py --endpoint exchanges-list`

Or install as Claude Code plugin:
```bash
/plugin marketplace add EodHistoricalData/eodhd-claude-skills
/plugin install eodhd-api@eodhd-claude-skills
```

## Testing

```bash
# Verify client works (no external deps needed)
python skills/eodhd-api/scripts/eodhd_client.py --endpoint exchanges-list

# Test specific endpoint
python skills/eodhd-api/scripts/eodhd_client.py --endpoint eod --symbol AAPL.US --from-date 2025-01-01 --to-date 2025-01-31

# ESG endpoint tests
python skills/eodhd-api/scripts/test_investverte_list_sectors.py
python skills/eodhd-api/scripts/test_investverte_view_company.py
```

No test framework — validation is manual against live API.

## Deployment

No runtime deployment. Distributed via GitHub releases.

- Version bumps: edit `version` in `.claude-plugin/marketplace.json`
- Auto-release: `.github/workflows/release.yml` triggers on version bump
- Users install/update via Claude Code plugin system

## Conventions

### Documentation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EodHistoricalData/eodhd-claude-skills](https://github.com/EodHistoricalData/eodhd-claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
