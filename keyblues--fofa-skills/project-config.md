---
trigger: always_on
description: FOFA cyberspace search engine skill — asset discovery, vulnerability mapping, threat intelligence, fingerprinting, and statistical aggregation.
---


# FOFA Search Skill

FOFA cyberspace search engine skill. Covers asset discovery, vulnerability mapping, threat intelligence, fingerprinting, and statistical aggregation.

## When to Activate This Skill

Activate this skill when the user's request involves any of:

- Searching for internet-facing assets (domains, IPs, services, ports)
- Vulnerability impact assessment / exposure checking
- Fingerprint identification (favicon hash, JARM, banner, certificate)
- Statistical analysis of internet asset distribution
- Threat intelligence (phishing detection, C2 identification, suspicious infrastructure)
- Cross-correlation / pivoting through shared infrastructure
- Any explicit mention of "FOFA", "网络空间搜索", "资产测绘", "fofa搜索"

Do **NOT** activate for:
- General web search queries (use web search tools instead)
- Non-cyberspace reconnaissance questions
- Questions solely about other search engines (Shodan, Censys) — unless user explicitly asks to cross-reference with FOFA

## Version Compatibility

- Skill version: **0.1.0** (must match `fofa-skills --version` output)
- Compatible with: Claude Code >= 1.0.0, Python >= 3.9
- If `--version` output doesn't match this SKILL.md's stated version, warn the user of potential inconsistency and suggest updating

## Important

All commands must be run from the project root (where `SKILL.md` resides).

## Prerequisites

- Python 3.9+ (stdlib only, zero pip dependencies)
- FOFA API KEY (32-char hex from [FOFA Personal Center](https://fofa.info))

### Verify Setup

```bash
python scripts/fofa_smart.py --version   # fofa-skills 0.1.0
python scripts/fofa_smart.py info        # verify key + account
```

### KEY Configuration

Lookup order:
1. Environment variable `FOFA_KEY`
2. `.env` file in project root: `FOFA_KEY=xxx`

If script returns `{"__fofa__": true, "error": true, "code": 1002}`:
- Ask user to get key from https://fofa.info
- Write to `.env`: `echo "FOFA_KEY=xxx" > .env`
- Retry the command

Optional env vars:
- `FOFA_DB_PATH` — SQLite path, default `./data/fofa_cache.db`
- `FOFA_BASE_URL` — API base URL, default `https://fofa.info/api/v1`

> **Note on `FOFA_ALLOW_FPOINTS=true`**: Even when set, AI must still notify the user before every paginated request that F-points will be consumed and wait for confirmation. This env var only bypasses the code-level hard block, not the AI-level confirmation obligation.

## Skill Boundaries & Refusal Rules

### Must Refuse
- **Unauthorized attack planning**: Queries clearly intended for planning attacks on specific targets without authorization context
- **Mass surveillance**: Bulk collection of personal data without legitimate security research context
- **Illegal activity**: Any query tied to unauthorized penetration testing or illegal operations

### Must Warn
- **Mass scanning**: Warn when query scope is extremely broad and may indicate indiscriminate scanning
- **Sensitive data**: Remind user of data protection obligations when results contain personal data (names, emails, credentials)
- **F-point consumption**: Always warn before any operation that costs F-points

### Behavioral Rules
- When uncertain which subcommand to use, prefer `stats` (free) over `search` for count-only questions
- When `search` returns no results, suggest alternative query patterns before giving up
- Always explain query parts in plain language — never raw-paste FOFA syntax to user
- When multiple queries are needed, batch them and explain the workflow
- When user is doing comprehensive reconnaissance, suggest cross-referencing with Censys/Shodan for validation (FOFA's strength is Chinese internet space; other engines may have better global coverage)

## Intent → Command Mapping

| User Intent | Command | Example |
|-------------|---------|---------|
| Find assets under a domain/IP | `search` | "What services does example.com have?" |
| Find exposed services/components | `search` | "Find exposed MySQL on the internet" |
| Vulnerability impact assessment | `search --full` | "What assets are affected by Log4j?" |
| Assets matching a fingerprint | `search` | "Sites with icon_hash xxx" |
| Distribution of a dimension | `stats` | "Port distribution of Apache" |
| Details of a specific IP/domain | `host` | "What's running on 1.1.1.1?" |
| Check account status | `info` | "How many F-points do I have?" |
| Re-view previous query results | `cache-read` | "Show me that search again" |
| Export results to file | `cache-export` | "Export results as CSV" |
| View operation history | `audit-log` | "What queries have been run?" |

### `search` vs `host`

- **`search`**: Find assets matching conditions → returns a list. Use when user asks "what assets match X?"
- **`host`**: Get full profile of one IP/domain → returns all services on that host. Use when user asks "what's running on this machine?"

Example: "Check 1.1.1.1" → `host` for full profile; `search -q 'ip="1.1.1.1"'` if it's just a filter condition.

### `search` vs `stats`

- **`search`**: When user wants a concrete asset list
- **`stats`**: When user only wants distribution counts (e.g., "how many per port"). Free, no F-point cost.

## Query Construction

Convert natural language to FOFA query:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keyblues/fofa-skills](https://github.com/keyblues/fofa-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
