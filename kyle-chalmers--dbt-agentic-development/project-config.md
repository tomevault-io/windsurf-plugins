---
trigger: always_on
description: > IMPORTANT: Everything in this repo is public-facing, so do not place any sensitive info here and make sure to distinguish between what should be internal-facing info (e.g. secrets, PII, recording guides/scripts), and public-facing info (instructions, how-to guides, actual code utilized). If there is information that Claude Code and other AI tools needs across sessions but should not be published, put it in the `.internal/` folder which is ignored by git per the `.gitignore`.
---

# AI Agent Instructions: dbt Agentic Development

> IMPORTANT: Everything in this repo is public-facing, so do not place any sensitive info here and make sure to distinguish between what should be internal-facing info (e.g. secrets, PII, recording guides/scripts), and public-facing info (instructions, how-to guides, actual code utilized). If there is information that Claude Code and other AI tools needs across sessions but should not be published, put it in the `.internal/` folder which is ignored by git per the `.gitignore`.

## Project Overview

This is a demo repository for a **KC Labs AI YouTube video** — a practical setup tutorial showing how to use AI coding assistants with dbt, from installing dbt through building convention-aware, lineage-informed models.

**Audience**: Analytics engineers, data professionals, and developers using dbt
**Demo Subject**: Full setup guide for dbt Agent Skills + dbt MCP Server with Claude Code
**Demo Project**: jaffle_shop (dbt's canonical demo project) with DuckDB adapter

> **Claude Code**: If `.internal/OWNER_CONFIG.md` exists, read it at the start of each session and use those concrete values (org URLs, resource names, emails) for all commands.
>
> **Viewers cloning this repo**: Create your own `.internal/OWNER_CONFIG.md` with your personal values (DevOps org, project, email). Then follow the README setup steps to install dbt and the AI tooling.

## Available Tools

### dbt CLI

- **Run models**: `dbt run` (all models) or `dbt run --select model_name`
- **Run tests**: `dbt test` (all tests) or `dbt test --select model_name`
- **Generate docs**: `dbt docs generate`
- **Serve docs**: `dbt docs serve` (opens browser with DAG visualization)
- **List resources**: `dbt ls` (list all models, tests, sources)
- **Compile SQL**: `dbt compile --select model_name` (render Jinja without executing)
- **Adapter**: DuckDB (runs locally, zero database setup)

### dbt Agent Skills

Installs dbt-specific conventions and patterns into Claude Code's CLAUDE.md:

```bash
npx skills add dbt-labs/dbt-agent-skills
```

What it provides:
- Naming conventions (e.g., `stg_[source]__[entity]`, `int_[entity]_[verb]`, `fct_[entity]`, `dim_[entity]`)
- ref() and source() usage patterns
- Test patterns (unique, not_null, relationships, accepted_values)
- YAML schema file conventions
- Model organization (staging → intermediate → marts)

### dbt MCP Server

Connects Claude Code to live dbt project metadata:

```bash
claude mcp add dbt -e DBT_PROJECT_DIR=$(pwd) -e DBT_PATH=$(which dbt) -- uvx dbt-mcp
```

What it provides:
- Live DAG lineage (which models depend on which)
- Column-level schema info (names, types, descriptions)
- Existing test coverage (what's already tested)
- Source definitions and freshness
- Project configuration and variables
- dbt CLI tools (run, test, compile, list, build, show, parse)
- Codegen tools (generate model YAML, source YAML, staging models)

> **Important:** If the MCP server only exposes docs tools, it can't find the dbt project or binary. Set both `DBT_PROJECT_DIR` (absolute path to the directory containing `dbt_project.yml`) and `DBT_PATH` (output of `which dbt`) in `.mcp.json`. Both are required for the CLI and codegen tools to load — the server does not auto-discover either.

#### CLI vs MCP: Output Comparison

| Capability | dbt CLI | dbt MCP Server |
| ---------- | ------- | -------------- |
| Run / test / build | Streams full log — per-model status, timing, PASS/WARN/ERROR counts | Returns `"OK"` |
| List resources | `dbt ls` → flat FQN strings | `mcp__dbt__list` → same flat FQN strings |
| Compile SQL | `dbt compile` → **prints rendered SQL** | `mcp__dbt__compile` → returns `"OK"` |
| Preview data | `dbt show` → formatted table | `mcp__dbt__show` → structured JSON |
| Column schemas + tests | Requires parsing `target/manifest.json` (partial) | `get_node_details_dev` → full structured JSON per node |
| DAG lineage | `dbt ls --output json` → flat NDJSON, reconstruct graph manually | `get_lineage_dev` → nested parent/child graph |
| dbt docs search | No CLI equivalent | `search_product_docs` |

#### When to Use CLI vs MCP

Both are available at all times. Choose based on what gives better results:

| Use CLI when… | Use MCP when… |
| ------------- | ------------- |
| Inspecting compiled SQL (`dbt compile` prints it; MCP just returns `"OK"`) | Querying lineage (`get_lineage_dev` returns a nested graph; CLI returns a flat list) |
| Diagnosing run/test failures (CLI streams per-model status, timing, PASS/WARN/ERROR counts; MCP returns `"OK"`) | Looking up column schemas, data types, or test coverage for a specific model (`get_node_details_dev`) |
| Listing resources — output is identical either way | Searching dbt product docs (`search_product_docs` — no CLI equivalent) |

#### MCP Fallback


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyle-chalmers/dbt-agentic-development](https://github.com/kyle-chalmers/dbt-agentic-development) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
