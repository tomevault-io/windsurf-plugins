---
trigger: always_on
description: > **Working on this repository?** Coding agents and maintainers should read
---

# Using `sql2json` with AI Agents and LLMs

> **Working on this repository?** Coding agents and maintainers should read
> `WORKFLOW.md` and `WORKFLOW.local.md` first. They define the public-safe
> branch/PR lifecycle, francisco-boards usage, validation gates, and release
> discipline for this repo (`S2J`, base branch `master`). The rest of this file
> explains how agents should use `sql2json` as a CLI/library.

`sql2json` runs SQL queries via SQLAlchemy and outputs JSON or CSV to stdout. It is invoked as a CLI tool or imported as a Python package — no framework coupling, no MCP server required.

> **Invocation:** examples below use the `sql2json` command, available **since v0.2.1**. On `0.2.0` and earlier — or when the package's scripts directory is not on `PATH` — substitute the equivalent `python -m sql2json ...` form (on Windows, `py -m sql2json ...`).

---

## Install & upgrade (for agents)

Install `sql2json` as an isolated tool, **with the database drivers bundled** so
queries against PostgreSQL/MySQL work without a follow-up step:

```bash
uv tool install "sql2json[postgres,mysql]"
# or
pipx install "sql2json[postgres,mysql]"
```

Why these methods matter in a sandbox:

- They work on **externally-managed** environments (Manjaro/Arch, Debian 12+,
  Ubuntu 23.04+, Homebrew Python), where a plain `pip install` is refused with
  `error: externally-managed-environment` ([PEP 668](https://peps.python.org/pep-0668/)).
- They expose `sql2json` on `PATH` (in `~/.local/bin`; run `pipx ensurepath` if
  it is missing) without mutating any project environment.

**Quote the extras.** In bash/zsh the brackets are glob characters, so the spec
must be quoted — `"sql2json[postgres,mysql]"` — or the shell expands it and the
install fails before the installer runs. In PowerShell use single quotes:
`'sql2json[postgres,mysql]'`.

**Selecting drivers:** `[postgres]` (psycopg2) or `[mysql]` (pymysql) for a
single database; bare `sql2json` is **SQLite only** — connecting to
Postgres/MySQL without the extras raises `ModuleNotFoundError: psycopg2` /
`pymysql`. For other databases (e.g. MS SQL Server's `pyodbc`), install the
driver alongside: `uv tool install "sql2json" --with pyodbc`.

If `sql2json` already exists but lacks a driver, reinstall with the extras
(`uv tool install "sql2json[postgres,mysql]" --force`) or inject it
(`pipx inject sql2json psycopg2-binary pymysql`).

**Inside a project/venv** (library use), add it as a dependency instead, keeping
the extras: `uv add "sql2json[postgres,mysql]"` or, in an activated venv,
`pip install "sql2json[postgres,mysql]"`.

**Upgrade** (carry the extras so drivers stay installed):

```bash
uv tool upgrade sql2json                              # or: uv tool install "sql2json[postgres,mysql]" --force
pipx upgrade sql2json
pip install --upgrade "sql2json[postgres,mysql]"      # inside a venv
```

Check the installed version with `uv tool list`, `pipx list`, or
`python -c "import importlib.metadata as m; print(m.version('sql2json'))"`.

---

## Strategy for Agents

Map a natural language request to these parameters:

1. **Identify the connection** (`--name`): which database to use.
2. **Select the query** (`--query`): prefer a named query from `config.json`, using connection-scoped queries when available; otherwise use raw inline SQL or a path to a `.sql` file prefixed with `@`.
3. **Resolve named-query precedence**: for `--name <conn> --query <name>`, sql2json checks `connection_queries.<conn>.<name>` first, then falls back to `queries.<name>`, then treats the value as raw SQL or `@file` when no named query exists.
4. **Supply parameters**: date variables and SQL bind parameters as extra `--key value` flags.
5. **Shape the output**: use `--first`, `--key`, `--value`, `--wrapper`, `--jsonkeys` to transform results.

---

## Discovery — orient before querying

Before calling a query, an agent can inspect what is configured:

```bash
# List available database connections
sql2json --list-connections --config /path/to/config.json
# → ["default", "mysql", "reporting"]

# List available named queries, grouped by scope
sql2json --list-queries --config /path/to/config.json
# → {"global": ["default", "sales_monthly"], "connections": {"mysql": ["table_sizes"], "reporting": ["total_users"]}}

# Request the old flat global-query list when integrating with legacy callers
sql2json --list-queries legacy --config /path/to/config.json
# → ["default", "sales_monthly"]
```

`--list-connections` prints a JSON array to stdout and exits 0. `--list-queries` prints the scoped discovery object by default; `--list-queries legacy` prints the old flat global query array. If `--config` is omitted the tool uses its normal config lookup order.

When selecting a named query for a connection, prefer a query listed under `connections.<connection>`; if none matches, use the matching name from `global`. Runtime lookup follows the same precedence: scoped query first, global query fallback, then raw SQL/`@file` behavior.

---

## Config file lookup order

When `--config` is not supplied, the tool searches in this order:

1. `./sql2json.json` (current working directory)
2. `./.sql2json/config.json` (current working directory)
3. `~/.sql2json/config.json` (user home directory)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fsistemas/sql2json](https://github.com/fsistemas/sql2json) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
