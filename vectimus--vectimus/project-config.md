---
trigger: always_on
description: Deterministic governance for AI coding tools and autonomous agents.  Intercepts every agent action, evaluates it against Cedar policies and returns allow/deny/escalate decisions before execution.  Open-source core with a commercial enterprise tier planned.
---

# AGENTS.md

## What is Vectimus?

Deterministic governance for AI coding tools and autonomous agents.  Intercepts every agent action, evaluates it against Cedar policies and returns allow/deny/escalate decisions before execution.  Open-source core with a commercial enterprise tier planned.

## Tech stack

- **Language:** Python 3.12+
- **Package manager:** uv (pyproject.toml, no setup.py)
- **Policy engine:** cedarpy >=4.8.0 (Cedar policy language bindings)
- **HTTP server:** FastAPI + uvicorn (optional, behind `pip install vectimus[server]`)
- **CLI:** click
- **Testing:** pytest + pytest-asyncio
- **Linting:** ruff
- **Logging:** structlog

## Development commands

```bash
uv sync --group dev          # Install all dependencies
pytest                       # Run tests
pytest tests/test_benchmark.py  # Run performance benchmarks
ruff check src/ tests/       # Lint
ruff format src/ tests/      # Format
vectimus test                # Test policies against sample events
vectimus init                # Generate hook configs for detected AI tools
```

## Project layout

```
policies/           # Cedar policy packs (11 domain-based dirs) — top-level for visibility
src/vectimus/
  engine/           # Core evaluation: evaluator, normaliser, models, Cedar schema, config, loader
  adapters/         # Thin hook translators for coding tools (Claude Code, Cursor, Copilot)
  integrations/     # Framework middleware/plugins (LangGraph, ADK, etc. — placeholder)
  server/           # Optional FastAPI server (behind vectimus[server])
  exporters/        # Audit log exporters (JSONL with file locking)
  cli/              # Click CLI commands (init, hook, test, status, pack, rule, observe, mcp, server)
tests/              # pytest tests
docs/               # Documentation
```

## CLI commands

| Command | Purpose |
|---------|---------|
| `vectimus hook --source <tool>` | Unified hook entry point for Claude Code, Cursor, Copilot |
| `vectimus init` | Detect tools, generate hook configs (merges with existing hooks) |
| `vectimus remove` | Remove Vectimus hooks from detected tools in this project |
| `vectimus test` | Test policies against sample events |
| `vectimus status` | Show configured tools, loaded policies, audit stats |
| `vectimus observe on/off/status` | Toggle observe mode (log only, no blocking) |
| `vectimus mcp allow/deny/list` | Manage MCP server allowlist |
| `vectimus rule list/show/disable/enable/overrides` | Manage individual rules |
| `vectimus pack list/enable/disable` | Manage policy packs |
| `vectimus server start` | Start the HTTP evaluation server |

## Writing standards

- No Oxford commas.
- No em dashes.
- No AI buzzwords (crucial, delve, landscape, leverage, pivotal, cutting-edge, game-changing, revolutionise).
- Vary sentence rhythm.  Mix short and long.
- Be specific.  No puffery.

## Code standards

- All Python code must have type hints.
- Docstrings on all public functions and classes.
- Keep functions small and focused.
- Use `pathlib.Path` for file paths, not string concatenation.  Windows compatibility matters.
- Line length limit: 100 characters (enforced by ruff).

## Key design decisions

**Fail closed.**  This is the single most important invariant in the codebase.  Only an explicit ALLOW from the policy engine should result in exit code 0.  Everything else (DENY, ESCALATE, errors, unknown values) must result in exit code 2 (deny).  When adding new code paths that handle decisions, always check for the allow case explicitly and deny everything else.  Never check only for DENY and let other values fall through to allow.

**Steer, don't just block.**  Every DENY must include a human-readable `reason` and a `suggested_alternative`.  A governance layer that only says "no" is a productivity killer.  One that says "not that way, try this instead" is a force multiplier.

**MVP is local-only.**  Command hooks evaluate locally via cedarpy.  No server, no daemon, no network, no attack surface by default.  The server module exists for team use cases and is activated explicitly behind `pip install vectimus[server]`.

**Observe before enforce.**  Observe mode logs decisions but always returns allow.  Teams run observe mode to see what would be blocked, review the audit log, then switch to enforcement.  This is how adoption works.

**MCP: allowlist, don't inspect.**  Vectimus intercepts MCP tool call requests, not server-side behaviour.  The highest-value control is server allowlisting (block calls to unapproved servers).  Input parameter inspection is defence in depth.  Docs are honest about what can and cannot be caught.

**Incident and standards-driven policies.**  Every policy rule must reference a real-world incident where possible or reference a control in a set of standards or recommendations like OWASP, etc.  Rules that exist "because best practice" are weak.  Rules that exist because a specific attack compromised thousands of developers are compelling.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vectimus/vectimus](https://github.com/vectimus/vectimus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
