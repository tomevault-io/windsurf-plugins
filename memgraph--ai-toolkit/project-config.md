---
trigger: always_on
description: Guidance for coding agents working in `memgraph/ai-toolkit`. Keep it in sync
---

# AGENTS.md

Guidance for coding agents working in `memgraph/ai-toolkit`. Keep it in sync
with the codebase — update it in the same PR as the change that makes part
of it stale.

## What this repo is

Memgraph AI Toolkit — a `uv` workspace of independently-versioned Python
packages for building AI/agent applications on Memgraph: core DB utilities,
framework integrations (LangChain, MCP, LightRAG), a document-to-graph
pipeline, a SQL-to-graph migration agent, and **Context Graph**, a family of
components that turn Claude Code / Codex agent sessions into a queryable
Memgraph graph.

## Repository layout

| Path | What it is |
|---|---|
| `memgraph-toolbox/` | Core Memgraph client/tooling. Dependency of nearly everything else here. |
| `integrations/langchain-memgraph/` | LangChain graph store, QA chain, toolkit. |
| `integrations/mcp-memgraph/` | MCP server exposing Memgraph to LLMs. |
| `integrations/lightrag-memgraph/` | LightRAG storage backends (KV/vector/doc-status/graph) on Memgraph. |
| `unstructured2graph/` | Chunks unstructured input (files/URLs/text) and hands chunks to LightRAG for entity extraction. Outside the Context Graph family but shares its testing conventions. |
| `agents/sql2graph/` | MySQL/Postgres → Memgraph migration agent. Has its own `uv.lock`/`.python-version`; run it with `cd agents/sql2graph && uv run main.py`. |
| `context-graph/` | The Context Graph family — see below. |
| `scripts/dev-memgraph.sh` | Local dev lifecycle: exploration Memgraph + isolated test Memgraph for the context-graph family and unstructured2graph. |
| `skills/release/SKILL.md` | Release process for every PyPI/Docker-published package. |

### The Context Graph family (`context-graph/`)

| Package | Role |
|---|---|
| `agent-context-graph` | Event hub. Normalizes runtime hooks / SDK activity into a shared Event Protocol and routes it to graph connectors. |
| `actions-graph` | Records tool calls/results/messages/subagent activity as `(:Action)`/`(:Agent)` nodes — observability, not memory. |
| `skills-graph` | Tracks Agent-Skills-spec `(:Skill)` usage per session. |
| `sessions-graph` | Owns `(:User)`/`(:Session)`, durable `(:Memory)` writes/recall, and session reconciliation into `(:Episode)` + extracted entities. |

Everything joins on a shared, idempotently-`MERGE`d `(:Session {session_id})`
node; only `sessions-graph` owns `(:User)` and `HAD_SESSION`.

Substantial design work in this family is tracked as GitHub issues labeled
`wayfinder:map` (title `Map: ...`), broken into `wayfinder:grilling` /
`wayfinder:research` / `wayfinder:prototype` / `wayfinder:task` child issues,
using this repo's `/grilling`, `/domain-modeling`, and `/research` skills.
Check open maps before starting non-trivial work here:
```bash
gh issue list --label wayfinder:map --state open
```

## Environment & setup

- Python `>=3.10`, dependency/workspace manager is `uv`. Workspace members are
  listed in the root `pyproject.toml` under `[tool.uv.workspace]`.
- Install a package editable with its test extras, e.g.:
  ```bash
  uv pip install -e memgraph-toolbox"[test]"   # quote extras on zsh/macOS
  ```
- Working from a nested worktree under this checkout (e.g.
  `.claude/worktrees/<name>/`)? Run `uv venv` there first. With no local
  `.venv`, `uv pip install -e ...`/`uv run --package ...` walk up and
  silently reuse — and mutate — the outer checkout's shared `.venv`,
  repointing its editable installs at whichever worktree you happen to run
  from. That's a real collision if another worktree or session relies on
  that same shared environment.
- Run a workspace package's own suite via `uv run --package <name> --extra test pytest ...`
  (see `.github/workflows/tests.yaml` for the exact invocation per package,
  including which extras each one needs — e.g. `skills-graph`, `actions-graph`,
  and `sessions-graph` all also need `--extra agent-context-graph`, and
  `sessions-graph` additionally needs `--extra reconciliation`).

## Common commands

**Lint / format** (matches `.github/workflows/lint.yaml`):
```bash
ruff check .
ruff format --check .
# to fix locally:
ruff check --fix . && ruff format .
```
`.pre-commit-config.yaml` runs the same ruff hooks plus
trailing-whitespace/end-of-file-fixer/check-yaml/check-toml/large-file/merge-conflict checks.

**Tests, non-context-graph packages** — start a plain Memgraph container and
run pytest in the package directory (see root `README.md` "Developing
Locally" and `tests.yaml` for per-package extras/env vars, e.g.
`langchain-memgraph` and `memgraph-toolbox`'s evaluation extras need
`OPENAI_API_KEY`).

**Tests, Context Graph family + unstructured2graph** — these test against a
*real* Memgraph, not mocks (see Testing policy below). Don't hand-start a
stray container for these; `scripts/dev-memgraph.sh` owns the disposable test
instance so the automated suite can never collide with or wipe your
exploration data:
```bash
./scripts/dev-memgraph.sh test                  # all packages
./scripts/dev-memgraph.sh test sessions-graph    # one package
./scripts/dev-memgraph.sh test-down              # reclaim the test container when done
```
The same script also drives a disposable *exploration* Memgraph
(`up`/`hooks-local`/`inspect`/`reconcile`/`down`) for dogfooding against your

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [memgraph/ai-toolkit](https://github.com/memgraph/ai-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
