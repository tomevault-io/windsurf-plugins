---
trigger: always_on
description: This file is intended for AI coding agents (Claude Code, OpenClaw, Codex, Copilot, Cursor, etc.) working in this repository.
---

# AGENTS.md

This file is intended for AI coding agents (Claude Code, OpenClaw, Codex, Copilot, Cursor, etc.) working in this repository.

## What This Repo Is

**FirstData** is a structured knowledge base of global authoritative open data sources. It is a **pure data repository** — no application code, no runtime logic.

Your job here is to **create or edit JSON metadata files** that describe real-world data sources (government databases, international organizations, academic datasets, etc.).

## Validation

Dependencies are managed with [uv](https://docs.astral.sh/uv/). Run the following before submitting:

```bash
# Install dependencies (first time only)
uv sync

# Run all validation checks
make check

# Or run checks individually:
make validate       # Validate JSON schema compliance
make check-ids      # Check for duplicate IDs
make check-domains  # Check domain naming consistency
```

A GitHub Action runs these checks automatically on every PR. PRs that fail validation cannot be merged.

## The Only Thing You Need to Know: The JSON Schema

Every file under `firstdata/sources/` must conform to `firstdata/schemas/datasource-schema.json`.

### Required Fields

```json
{
  "id": "worldbank-open-data",
  "name": {
    "en": "World Bank Open Data",
    "zh": "世界银行开放数据"
  },
  "description": {
    "en": "...",
    "zh": "..."
  },
  "website": "https://www.worldbank.org",
  "data_url": "https://data.worldbank.org",
  "api_url": "https://api.worldbank.org/v2/",
  "authority_level": "international",
  "country": null,
  "domains": ["economics", "health", "education"],
  "geographic_scope": "global",
  "update_frequency": "quarterly",
  "tags": ["world bank", "development", "gdp", "poverty", "世界银行"],
  "data_content": {
    "en": ["GDP and national accounts", "Poverty and inequality indicators"],
    "zh": ["GDP和国民账户", "贫困和不平等指标"]
  }
}
```

### Field Rules

| Field                | Allowed Values / Constraints                                                                                                    |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `id`               | Lowercase, hyphens only. Must be globally unique. Pattern:`^[a-z0-9-]+$`                                                      |
| `name.en`          | Required. Add `zh` and `native` when applicable                                                                             |
| `description.en`   | Required. Add `zh` when applicable                                                                                            |
| `website`          | Top-level org homepage                                                                                                          |
| `data_url`         | Must point directly to the data access/download page, NOT the homepage                                                          |
| `api_url`          | API docs or endpoint URL. Use `null` if no API exists                                                                         |
| `authority_level`  | `government` · `international` · `research` · `market` · `commercial` · `other`                                |
| `country`          | ISO 3166-1 alpha-2 (e.g.`"CN"`, `"US"`). **Must be `null`** when `geographic_scope` is `global` or `regional` |
| `domains`          | Array of strings, at least one. **MUST use lowercase** (e.g., `"economics"` not `"Economics"`). See [DOMAINS.md](firstdata/schemas/DOMAINS.md) for standard domain list |
| `geographic_scope` | `global` · `regional` · `national` · `subnational`                                                                   |
| `update_frequency` | `real-time` · `daily` · `weekly` · `monthly` · `quarterly` · `annual` · `irregular`                         |
| `tags`             | Mixed Chinese/English keywords for semantic search. Include synonyms and data type names                                        |
| `data_content`     | Optional but recommended. Lists of strings describing what data is available                                                    |

## Where to Place New Files

```
firstdata/sources/
├── china/{domain}/{id}.json                          # Chinese gov & institutions
├── international/{domain}/{id}.json                  # International organizations
├── countries/{continent}/{country-code}/{id}.json    # National official sources
├── academic/{discipline}/{id}.json                   # Academic/research databases
└── sectors/{ISIC-code}-{name}/{id}.json              # Industry datasets
```

**Examples:**

- China customs data → `sources/china/economy/trade/customs.json`
- WHO health data → `sources/international/health/who.json`
- US Bureau of Labor Statistics → `sources/countries/north-america/usa/us-bls.json`
- PubMed → `sources/academic/health/pubmed.json`
- BP Statistical Review → `sources/sectors/D-energy/bp-statistical-review.json`

## Do Not Touch (Auto-Generated or Protected)

The following files are maintained automatically by CI/scripts. **AI agents must NOT modify them manually:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MLT-OSS/FirstData](https://github.com/MLT-OSS/FirstData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
