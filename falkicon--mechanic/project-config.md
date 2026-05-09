---
trigger: always_on
description: Technical reference for AI agents working on the Mechanic project.
---

# Mechanic - Agent Documentation

Technical reference for AI agents working on the Mechanic project.

---
## The year is 2026. Your Year clock may be off!


## MCP Server (Primary)

Mechanic exposes all 53 commands as MCP tools. **ALWAYS use MCP tools directly** - this is the fastest and most reliable way to interact with the ecosystem.

### Key MCP Tools

| Tool | Description |
|------|-------------|
| `addon.output` | Get all errors, tests, console logs (after user confirms reload) |
| `addon.test` | Run Busted tests for an addon |
| `addon.lint` | Run Luacheck linter |
| `sandbox.test` | Run tests offline (no WoW needed) |
| `api.search` | Search WoW APIs (offline) |

### After In-Game Changes

**IMPORTANT**: Do NOT call `addon.output` immediately after code changes. The timing between reload and SavedVariables sync is unpredictable.

1. **Ask** the user to `/reload` in WoW
2. **Wait** for the user to confirm the reload is complete
3. **Then** call the `addon.output` MCP tool with `agent_mode=true` to get accurate results

---

## CLI Fallback (Users Only)

The `mech` CLI is for user interaction. AI agents should use MCP tools. Reference the CLI documentation only to understand the underlying command structure.

---

## Quick Reference

| Component | Path | Description |
|-----------|------|-------------|
| **WoW Addon** | `!Mechanic/` | In-game development hub |
| **Desktop Tool** | `desktop/` | Local companion (CLI + Dashboard) |
| **Specifications** | `PLAN/` | Phase plans and master spec |

---

## Project Structure

```
Mechanic/                   ← Git repo root
├── !Mechanic/              ← Bootstrap addon (loads first via ! prefix)
│   ├── !Mechanic.toc
│   ├── Bootstrap.lua
│   ├── MechanicQueue.lua
│   └── Libs/
├── Mechanic/               ← Main addon (full UI hub)
│   ├── Mechanic.toc
│   ├── Core.lua
│   ├── Utils.lua
│   ├── UI/
│   └── Libs/
├── desktop/                ← Mechanic Desktop
│   ├── pyproject.toml
│   ├── dashboard/          ← Web UI (vanilla HTML/JS)
│   ├── data/               ← SQLite database
│   ├── tests/              ← Pytest test suite (9 tests)
│   └── src/mechanic/
│       ├── cli.py          ← Click CLI entry point
│       ├── server.py       ← FastAPI + WebSocket
│       ├── watcher.py      ← SavedVariables file watcher
│       └── commands/       ← Command modules
│           ├── core.py       ← Base commands (sv.*, reload.*, etc.)
│           ├── development.py ← addon.validate, addon.lint, etc.
│           ├── release.py    ← version.bump, changelog.add, etc.
│           ├── locale.py     ← locale.validate, locale.extract
│           ├── atlas.py      ← atlas.scan, atlas.search
│           └── environment.py ← addon.create, addon.sync, libs.check
├── PLAN/                   ← Project-wide specs
├── AGENTS.md               ← This file
├── README.md
└── CHANGELOG.md
```

---

## ⚠️ CRITICAL: Development Standards

> **All new features MUST follow structured command principles.**

### Core Principles

1. **Commands First**: Every feature is a command with typed input/output schemas.
2. **Structured Results**: All commands return `CommandResult` with `success`, `data`, `error`.
3. **Actionable Errors**: Errors include `code`, `message`, and `suggestion` for recovery.
4. **Metadata for Trust**: Include `sources`, `reasoning`, and `confidence` where applicable.
5. **Headless Backend**: UI is a pure consumer of commands via `/api/execute` bridge.

### Command Template

```python
from afd import CommandResult, success, error
from afd.core.metadata import create_source
from pydantic import BaseModel, Field

class MyInput(BaseModel):
    param: str = Field(..., description="Input parameter")

class MyOutput(BaseModel):
    result: str

@server.command(
    name="feature.action",
    description="Description of what this command does",
    input_schema=MyInput,
    output_schema=MyOutput,
)
async def my_command(input: MyInput, context: Any = None) -> CommandResult[MyOutput]:
    src = create_source(type="file", id="my-source", title="Source Name")
    return success(
        data=MyOutput(result="value"),
        reasoning="Explanation of what happened",
        sources=[src],
        confidence=0.95
    )
```

---

## Command Reference

For the complete command reference with all 53 commands, see the **using-mechanic** skill:
`.claude/skills/using-mechanic/references/afd-commands.md`

### Command Categories (Summary)

| Category | Commands | File |
|----------|----------|------|
| Core | `sv.*`, `dashboard.*`, `server.*` | `core.py` |
| Development | `addon.validate`, `addon.lint`, `addon.format`, `addon.test`, `addon.deprecations` | `development.py` |
| Release | `version.bump`, `changelog.add`, `git.*`, `release.all` | `release.py` |
| Environment | `addon.create`, `addon.sync`, `libs.*`, `env.status`, `system.pick_file` | `environment.py` |
| Locale | `locale.validate`, `locale.extract` | `locale.py` |
| Atlas | `atlas.scan`, `atlas.search` | `atlas.py` |
| Lua | `lua.queue`, `lua.results` | `lua.py` |
| API | `api.search`, `api.info`, `api.list`, `api.queue`, `api.stats`, `api.populate`, `api.generate`, `api.refresh` | `api.py`, `apidefs.py` |
| Sandbox | `sandbox.generate`, `sandbox.status`, `sandbox.exec`, `sandbox.test` | `sandbox.py` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Falkicon/Mechanic](https://github.com/Falkicon/Mechanic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
