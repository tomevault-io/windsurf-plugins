---
trigger: always_on
description: This file provides shared project guidance for agent runtimes working with this repository.
---

# AGENTS.md

This file provides shared project guidance for agent runtimes working with this repository.

## Repository Overview

This repository is **claude-code-codex-bridge**.

It provides the `cc-codex-bridge` CLI, a standalone tool that bridges a local Claude Code setup into Codex-compatible artifacts without creating a second hand-maintained system.

## Architecture

### Repository Structure

```text
claude-code-codex-bridge/
├── .claude/
│   └── docs/
│       ├── analysis/
│       ├── decisions/
│       ├── learnings/
│       ├── patterns/
│       ├── plans/
│       └── research/
├── .github/
│   └── workflows/
├── docs/
│   ├── agent-skills-standard.md
│   ├── claude-code-mcp-reference.md
│   ├── codex-cli-reference.md
│   └── mcp-bridge-mapping.md
├── src/
│   └── cc_codex_bridge/
│       ├── __main__.py
│       ├── cli.py
│       ├── discover.py
│       ├── reconcile.py
│       ├── translate_agents.py
│       ├── translate_skills.py
│       └── ...
├── tests/
├── AGENTS.md
├── CHANGELOG.md
├── CLAUDE.md
├── FOLLOW_UPS.md
├── IDEAS.md
├── LICENSE
├── README.md
└── pyproject.toml
```

### Package Layout

- Runtime code lives under `src/cc_codex_bridge/`.
- Tests live under `tests/`.
- The installable CLI command is `cc-codex-bridge`.
- The Python module entrypoint is `python3 -m cc_codex_bridge`.

### Runtime Dependencies

The `claude` CLI must be available on PATH. The bridge shells out to `claude plugins list --json` to determine which plugins are enabled. Without it, `discover()` raises a `DiscoveryError` and the bridge cannot function.

### Runtime Contract

The bridge reads local Claude Code state and produces Codex-compatible outputs such as:

- `CLAUDE.md` as the `@AGENTS.md` shim
- `~/.codex/agents/*.toml` (global agent files, tracked in global registry)
- `.codex/agents/*.toml` (project-local agent files)
- `~/.codex/skills/*`
- `~/.cc-codex-bridge/projects/<hash>/state.json` (per-project bridge state)
- `~/.cc-codex-bridge/registry.json` (global ownership registry)

Do not treat generated `.codex/*` or generated Codex skill directories as hand-authored source.

## Development

### Setup

Install in editable mode:

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install --upgrade pip
python3 -m pip install -e ".[dev]"
```

### Testing

**Always run tests through the project venv.** The package depends on PyYAML and other libraries that are not available in the system Python. Running bare `pytest` without the venv will fail with `ModuleNotFoundError`.

Run the full test suite after code changes:

```bash
source .venv/bin/activate && pytest tests -q
```

Run coverage:

```bash
source .venv/bin/activate && pytest --cov=cc_codex_bridge --cov-report=term-missing tests -q
```

### Packaging

- Keep the `src/` layout intact.
- Keep imports using the `cc_codex_bridge` package path.
- Keep the console script entrypoint in `pyproject.toml` aligned with the package layout.

## Development Model

This project is AI-written and AI-maintained. The human (Vlad) sets direction, makes architectural decisions, and reviews work. Claude Code agents do the implementation, testing, analysis, and maintenance. "Single maintainer" does not mean capacity-constrained — it means single human decision-maker with AI execution capacity. Do not assume limited implementation bandwidth when reasoning about priorities or feasibility.

No agent carries context between sessions — every agent reads the code cold. This has practical implications:

- **Prefer single canonical implementations** over duplicated patterns. An agent will copy whichever pattern it encounters first; if two conventions exist for the same thing, drift is inevitable.
- **Constants, types, and named helpers are discovery mechanisms.** They are more valuable here than in a human-authored codebase — they are the primary way agents find the "right" way to do something.
- **Consolidating duplicated logic is drift prevention**, not polish. Treat it accordingly when prioritizing work.

## Domain References

The bridge translates between two ecosystems. Authoritative reference documents live in `docs/`:

- `docs/agent-skills-standard.md` — the open Agent Skills standard (from [agentskills.io](https://agentskills.io/)): skill directory structure, `SKILL.md` format, frontmatter fields, progressive disclosure, client implementation contract, and script conventions.
- `docs/codex-cli-reference.md` — Codex CLI specifics (from [developers.openai.com/codex](https://developers.openai.com/codex/) and the [Codex source](https://github.com/openai/codex)): instructions discovery (`AGENTS.md`), skill discovery hierarchy, agent role configuration, agent file auto-discovery, and Claude Code vs Codex comparison.
- `docs/claude-code-mcp-reference.md` — Claude Code MCP server configuration format: `~/.claude.json` structure, per-project scoping, `.mcp.json` project-shared format, stdio/HTTP/SSE transports, and `headersHelper`/`oauth` fields.
- `docs/mcp-bridge-mapping.md` — MCP bridge mapping rules: how each CC MCP field maps to Codex `config.toml` format, transport-specific translation, bearer token extraction, and unsupported feature handling.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vladolaru/claude-code-codex-bridge](https://github.com/vladolaru/claude-code-codex-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
