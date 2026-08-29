---
trigger: always_on
description: Nauro is a versioned project context system for AI coding agents. This is a `uv` workspace monorepo with two packages:
---

# CLAUDE.md — Nauro (monorepo)

@AGENTS.md

Nauro is a versioned project context system for AI coding agents. This is a `uv` workspace monorepo with two packages:

| Package | Path | Python | Purpose |
|---|---|---|---|
| `nauro` | `packages/nauro/` | 3.10+ | CLI + local MCP server (stdio). Reads/writes `~/.nauro/projects/` |
| `nauro-core` | `packages/nauro-core/` | 3.10+ | Shared pure-Python logic: parsing, validation, context assembly, constants. No I/O; compute-only deps. |

Each package has its own `pyproject.toml` and test suite; `packages/nauro/` also carries a package-level `CLAUDE.md` (nauro-core does not). The remote MCP server (`mcp-server/`) lives in a separate private repository.

## The one architectural fact that matters

The project store lives at `~/.nauro/projects/<project-id>/` — **not** inside any repo. This is the core design decision. A per-repo store would break cross-repo context, which is the problem Nauro exists to solve. The registry at `~/.nauro/registry.json` is keyed by project id (ULID); each entry carries the project name as metadata plus one or more associated repo paths on the machine.

```
~/.nauro/
  registry.json                  # id-keyed entries: name, mode, repo paths
  projects/
    <project-id>/
      project.md                 # stable: goals, non-goals, users, constraints
      state_current.md           # volatile: current sprint, blockers, recent completions
      state_history.md           # append-only history of completed work
      stack.md                   # tech choices with rationale and rejected alternatives
      open-questions.md          # append-only unresolved threads
      decisions/
        001-title.md             # one file per decision, sequential, immutable
      snapshots/
        v001.json                # full store capture at a point in time
```

All files are freeform markdown. No database. No JSON for content — JSON only for `registry.json` and snapshots.

## Cross-package architecture

```
~/.nauro/projects/<id>/          Local store (flat markdown + JSON snapshots)
        │
        ├── nauro CLI              reads/writes directly
        ├── local MCP (stdio)      reads/writes directly, spawned by Claude Code
        │
        └── nauro sync ──────►  S3 bucket (remote storage)
                                     │
                                     └── remote MCP (Lambda, separate repo) reads/writes via S3
```

## Config and credentials

User config lives at `~/.nauro/config.json` (written by `nauro auth login` and other feature-specific commands; inspect with `nauro config get/list/unset`, which resolve top-level keys only):
- `auth` object — credentials from `nauro auth login`, nested under one top-level `auth` key (so inspect with `nauro config get auth`, not a dotted path): `access_token` is the Auth0 bearer sent to the presign sync endpoints; `refresh_token` mints a fresh access token when the bearer expires; `sub` is the raw JWT subject id (identity; the block also persists `sanitized_sub` and `user_id`)
- Auth0 domain, client ID, API URL, and audience ship as defaults in `nauro/auth.py`; env vars (`NAURO_AUTH0_*`, `NAURO_API_URL`) or config keys override (domain + client_id must be set as a pair)
- `NAURO_HOME` env var overrides `~/.nauro/` for testing

## Stack

- CLI: Python 3.10+, Typer
- MCP server: local stdio transport (FastMCP), spawned by the MCP client
- Storage: flat markdown + JSON snapshots
- Templating: f-strings and Python string templates — no Jinja2

## CLI commands

Principal commands (run `nauro --help` for the full surface):

- `nauro init <name>` — register a new project in `~/.nauro/`, scaffold the store, associate repo paths
- `nauro adopt` — adopt an existing repo: register it, wire MCP across surfaces, install the `/nauro-adopt` skill (`--with-skills` / `--with-subagents` add the opt-in skills and bundled subagents)
- `nauro attach <project_id>` — associate the current repo with an existing cloud project
- `nauro link` — promote a local-only project to cloud
- `nauro note <text>` — append a decision (default) or question (if text ends with `?` or `--question` flag)
- `nauro sync` — capture a snapshot, regenerate `AGENTS.md` in all associated repos
- `nauro log` — list recent snapshots with metadata
- `nauro status` — capability table for the current project (active surfaces, absolute store path)
- `nauro doctor` — report deterministic store-integrity defects (unparseable decision files, dangling or cyclic supersession refs, status contradictions) plus repairable supersede backref orphans; report-only, always exits 0
- `nauro repair` — flip the single unambiguous supersede backref orphan after interactive confirmation; every other shape is reported with guidance and left alone
- `nauro graph` — render the decision graph to one self-contained HTML file in the store directory and open it
- `nauro serve` — start the local MCP server (stdio transport)
- `nauro import --memory-bank <path>` — migrate a Cline/Roo Code Memory Bank
- `nauro import --adr <path>` — migrate Architecture Decision Records


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nauro-AI/nauro](https://github.com/Nauro-AI/nauro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
