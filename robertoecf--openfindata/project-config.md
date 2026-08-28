---
trigger: always_on
description: This file is for coding agents working in this repository. Keep it practical:
---

# AGENTS.md: Dados Financeiros Abertos

This file is for coding agents working in this repository. Keep it practical:
follow the project conventions, avoid speculative dependencies, and produce
reproducible data work.

For Claude Code / Cursor harness specifics (worktrees, ship routing), see
[`CLAUDE.md`](CLAUDE.md).

## Agent skills

Workflow docs live under `docs/agents/`. Keep this section as pointers:

- Cold-start orientation map: [`docs/agents/orientation.md`](docs/agents/orientation.md)
- Domain docs consumption: [`docs/agents/domain.md`](docs/agents/domain.md)
- Quality gates and preflight: [`docs/agents/quality.md`](docs/agents/quality.md)
- MCP Trust review gate: [`docs/agents/mcp-trust-review.md`](docs/agents/mcp-trust-review.md)
- MCP Trust reviewer skill: [`.claude/skills/mcp-trust-reviewer/SKILL.md`](.claude/skills/mcp-trust-reviewer/SKILL.md)
- Ship parent workflow: [`docs/agents/openfindata-ship/SKILL.md`](docs/agents/openfindata-ship/SKILL.md)

Harness-global skills (adversarial-review, deslop, handoff, tdd, …) are not
duplicated in this repo; use the installed host skills.

## Project baseline

- Implementation checkout: a dedicated **worktree**, never the root checkout
  and never `main`. Root/`main` are inspect-only (see `CLAUDE.md`).
- Project name: Dados Financeiros Abertos.
- Distribution/package slug: `openfindata`.
- Import package and CLI remain `findata` for compatibility.
- Scope: Python library + REST API + MCP server + CLI for Brazilian public
  financial data.
- Prefer public, reproducible sources. Do not introduce API keys, tokens, or
  private credentials into code, tests, docs, examples, or generated artifacts.
- Keep repo-facing Markdown disciplined and functional. Avoid decorative emoji.
- Do not commit generated caches, local virtualenvs, temporary chart outputs, or
  binary artifacts other than the generated registry SQLite already owned by the
  project.

## Quality gates

Before a code change is considered ready, run the smallest relevant check first,
then the full gate from the **worktree** before merging or release work:

```bash
bash scripts/ship/preflight.sh
```

Expanded equivalent (same interpreter resolver as preflight: worktree
`.venv`, then repo-root `.venv`, then `python3`):

```bash
# PY=$(first existing: .venv/bin/python | <repo-root>/.venv/bin/python | python3)
"$PY" -m ruff format --check src/ tests/ scripts/
"$PY" -m ruff check src/ tests/ scripts/
"$PY" -m mypy src/findata
"$PY" -m pytest tests/ -q
```

Ruff owns the Biome-like formatter/lint baseline and the ESLint-like AI
guardrails configured in `pyproject.toml`. Details:
[`docs/agents/quality.md`](docs/agents/quality.md).

For documentation-only edits, at least run:

```bash
git diff --check
```

## Source integration pattern

- New source modules live under `src/findata/sources/<source>/`.
- Add route, CLI, tests, docs, and mocked HTTP coverage together when exposing a
  new public dataset.
- Use `findata.http_client.get_json` / `get_bytes` for network access.
- Unit tests should not hit live APIs. Use `respx`; mark live checks as
  `@pytest.mark.integration`.
- Keep `mypy --strict` clean for new code.

## Base dos Dados / BigQuery usage

Base dos Dados is a supported free logged-in source. Treat it differently from
commercial-entitlement APIs such as ANBIMA's authenticated developer products:
SQL, Python and R access are free/self-serve, but BigQuery still requires the
operator's Google login and a billing project.

- Do not embed Google credentials, service-account JSON, refresh tokens, or
  project-specific secrets in code, tests, docs, examples, or generated
  artifacts.
- Prefer the local env var for interactive work:

```bash
export FINDATA_BD_BILLING_PROJECT_ID="<google-cloud-project-id>"
```

- The project also accepts `BASE_DOS_DADOS_BILLING_PROJECT_ID` and
  `GOOGLE_CLOUD_PROJECT` as fallbacks.
- Use the Project ID, not the display name.
- BigQuery can bill by bytes processed. Base dos Dados access is free, and
  BigQuery has a free monthly quota, but queries may still consume billable
  quota. Start with tiny `LIMIT` queries before broad scans.
- Prefer the `findata` CLI wrapper for local checks:

```bash
.venv/bin/findata basedosdados sql br_bd_diretorios_brasil municipio --limit 5
.venv/bin/findata basedosdados query \
  'SELECT id_municipio, nome FROM `basedosdados.br_bd_diretorios_brasil.municipio` LIMIT 5'
```

- For user-facing or PR evidence, report the billing project used only when it
  is already known to the user or explicitly provided in the task. Do not expose
  private credential paths.

## Graphics and chart generation standards

Agents may use this repo to generate exploratory charts for users. Treat the
Chart Lab (`/charts`, `src/findata/web/templates/charts.html`,
`src/findata/web/static/chart-explorer.js`) as the canonical visual and
informational reference, but not as a mandatory renderer.

Short runbook for any one-off chart request:

1. Generate an audit data file next to the visual artifact, preferably tidy CSV.
2. Generate a clear visual artifact (SVG, PNG, HTML, or Lightweight Charts HTML)
   with the same minimum information block used by Chart Lab.
3. Save or print the script/route used, data path, visual output path, and
   renderer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertoecf/OpenFinData](https://github.com/robertoecf/OpenFinData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
