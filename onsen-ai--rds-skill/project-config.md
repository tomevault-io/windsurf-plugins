---
trigger: always_on
description: Query any AWS Aurora PostgreSQL cluster via IAM authentication and psycopg2. Use whenever the user mentions RDS, Aurora, PostgreSQL, Aurora PgSQL, database queries, schema exploration, table metadata, column listing, data profiling, or wants to explore Aurora database objects. Also use for local analytics on previously saved query results. Works with any Aurora PostgreSQL cluster with IAM database authentication enabled. Always use this skill for any Aurora PostgreSQL task, even simple SELECT qu
---


# Aurora PostgreSQL Skill

Read-only Aurora PostgreSQL exploration and business analysis via IAM authentication. No passwords or secrets required — your AWS IAM identity is your credential. Cross-platform (Mac + Windows). Works with any AI coding agent.

All scripts are in `${CLAUDE_SKILL_DIR}/scripts/` and require Python 3, AWS CLI, and psycopg2.

## Python Command

Read `~/.rds-skill/config.json` and use the `"python"` key as the Python command.
If config doesn't exist yet, try `python3 --version` first, falling back to `python --version`.
Throughout this document, `PYTHON` means the detected Python command.

## First-Time Setup

**You cannot run the setup wizard directly** — it requires interactive terminal input.

Check if `~/.rds-skill/config.json` exists:
- **If it exists:** Read it to discover the configured connections (the `connections` map and the `default` key).
- **If it doesn't exist:** Tell the user to run the setup wizard in their terminal:

> Run this in your terminal to configure the Aurora connection:
> ```
> python3 scripts/setup.py
> ```
> (On Windows, use `python` instead of `python3`)

Wait for the user to confirm setup is complete before running any queries.

## Connection Selection

The skill supports **multiple named connections** (e.g. prod, staging, local). Config shape:

```json
{
  "default": "prod",
  "connections": {
    "prod": { "host": "...", "database": "main", "db_user": "...", "region": "eu-west-1", "profile": "...", "write_mode": "reject" },
    "staging": { "...": "..." }
  },
  "python": "/usr/bin/python3"
}
```

- Every script accepts `--connection NAME` to pick a specific connection.
- Without `--connection`, scripts use the `default` connection.
- To list connections: `PYTHON ${CLAUDE_SKILL_DIR}/scripts/setup.py --list`
- To switch defaults: `PYTHON ${CLAUDE_SKILL_DIR}/scripts/setup.py --set-default NAME`
- To remove one: `PYTHON ${CLAUDE_SKILL_DIR}/scripts/setup.py --remove NAME`
- To add another: re-run `PYTHON ${CLAUDE_SKILL_DIR}/scripts/setup.py` (interactive — instruct the user to run it themselves).

When the user mentions "prod" / "staging" / a specific cluster name, map that to the matching connection and pass `--connection NAME` on every script invocation. If they don't specify, use the default and mention which one you're using.

## Quick Reference

| Task | Script | When to use | Key Args |
|------|--------|-------------|----------|
| **Run SQL** | `query.py` | Any free-form read-only query | `"SELECT ..."` or `--sql-file=PATH` |
| **List schemas** | `schemas.py` | Starting point — see what schemas exist with table counts | |
| **List tables** | `tables.py` | Browse tables, check row counts and sizes before querying | `--schema=NAME` |
| **List columns** | `columns.py` | Understand column types, nullability, indexes | `--schema=NAME --table=NAME` |
| **Search objects** | `search.py` | Find tables or columns when you don't know the exact name | `--pattern=TEXT` |
| **Sample data** | `sample.py` | Quick peek at actual values — always do this before writing queries | `--schema=NAME --table=NAME` |
| **Data profile** | `profile.py` | Per-column stats (nulls, cardinality, min/max/avg) | `--schema=NAME --table=NAME` |
| **Local analytics** | `analyze.py` | Analyze saved results locally without hitting Aurora | `FILE --describe` |

### Common options (all RDS scripts)

| Option | Description |
|--------|-------------|
| `--format=txt\|csv\|json` | Terminal display format (default: txt) |
| `--save-format=txt\|csv\|json` | File save format (default: csv) |
| `--save=PATH` | Save to a specific file path |
| `--no-save` | Don't auto-save to ~/rds-exports/ |
| `--save-sql` | Save the SQL query as a .sql file alongside results |
| `--sql-file=PATH` | Read SQL from a file (query.py only) |
| `--connection=NAME` | Pick a named connection from `~/.rds-skill/config.json` (defaults to the saved `default`) |
| `--profile=NAME` | Override AWS profile |
| `--host=HOST` | Override cluster endpoint |
| `--database=NAME` | Override database |
| `--db-user=NAME` | Override database user |
| `--timeout=N` | Max wait seconds (default: 120) |
| `--max-rows=N` | Max rows to fetch (default: 1000) |

## Output and File Saving

All query results are **automatically saved** to `~/rds-exports/query-{timestamp}.csv`.
The terminal shows an aligned txt preview (first 200 rows). The saved file defaults to CSV for spreadsheet compatibility.

This means you always have:
- **Inline preview** (200 rows in txt format) — enough to understand the data shape and answer quick questions
- **Full CSV on disk** — for deeper analysis with `analyze.py` or for the user to open in a spreadsheet


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onsen-ai/rds-skill](https://github.com/onsen-ai/rds-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
