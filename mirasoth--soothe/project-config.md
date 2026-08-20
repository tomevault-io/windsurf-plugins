---
trigger: always_on
description: > Goal-driven orchestration framework for 24/7 autonomous agents. Extends deepagents with planning, durability, and remote agent interop.
---

# Soothe Development Guide

> Goal-driven orchestration framework for 24/7 autonomous agents. Extends deepagents with planning, durability, and remote agent interop.

---

## ⚠️ CRITICAL RULES

### 1. Implementation Guides (IGs)
- **Substantial work**: Create IG in `docs/impl/` (`IG-XXX-brief-title.md`) to track scope
- **Minor changes**: No verbose IG—use commit/PR context, or minimal stub only

### 2. Config Sync
When editing `config/nano.template.yml`, MUST also update `config/develop/nano.yml` with matching structure.
When editing any `config/*.template.yml`, also sync the packaged copies under
`packages/soothe-daemon/src/soothe_daemon/setup/templates/` (used by `soothed setup`).

### 3. Ecosystem First
Check `langchain-core`, `langchain-community`, `deepagents` before implementing anything:
- Tools: `BaseTool`, `@tool`
- Subagents: `SubAgent`, `CompiledSubAgent`
- MCP: `langchain-mcp-adapters`
- Memory: `deepagents.MemoryMiddleware`

### 4. Test Location
Tests go in package directories: `packages/<pkg>/tests/unit/` or `tests/integration/` — NOT root `tests/`.

### 5. Verification Required
Run `./scripts/verify_finally.sh` before ANY commit. Zero lint errors, all tests pass.

### 6. After Code Impl: Cleanse → Verify → Fix (MUST)
After implementing (or changing) code—and before marking work done (commit, PR, or handoff)—you MUST apply this sequence every time:

1. **Ask user before cleansing** — for each implementation finished, ask whether to cleanse legacy code, backward compatibility shims, and dead code related to the change.
2. **Cleanse related legacy and dead code** — if approved, remove superseded helpers, unused exports, duplicate parallel paths, backward compat shims, and stale tests/docs tied to the change. Do **not** change existing functionality while cleansing; cleanup is deletion/consolidation only, not behavior rewrites.
3. **Run verification** — `./scripts/verify_finally.sh`
4. **Fix all errors** — lint, format, tests, vulture; do not stop with a failing verify. Re-cleanse if fixes leave new dead code, then re-run verify until green.

### 7. Terminology
- NEVER use "layer N" — use concrete names (CoreAgent, StrangeLoop, GoalEngine)
- NEVER expose IG-XXX/RFC-XXX in user-facing text (logs, CLI, errors)—internal only
- IG-XXX/RFC-XXX references are allowed ONLY in internal code: docstrings, comments, and internal documentation. They must never appear in runtime strings visible to users.
- Only `docs/specs/` (RFCs) and `docs/impl/` (IGs) are allowed for active reference.
- Archived content in `docs/archive/` (drafts, completed analysis) is for historical reference only.
- When writing log messages, error text, CLI output, config field descriptions, or any user-visible string, omit all IG-/RFC- identifiers.

### 7b. Package Boundaries (MUST)

Soothe is a **one-way dependency DAG**. Before adding code, imports, or types,
place them in the correct **monorepo-owned** package. **Never reverse an arrow.**
Enforcement for owned packages: `scripts/check_module_import_boundaries.sh`
(wired into `./scripts/verify_finally.sh`).

**This monorepo owns** `soothe`, `soothe-autopilot`, `soothe-daemon`,
`soothe-cli`, and `soothe-sdk`. Submodules (`client/*`) are **consumed as
code** — do **not** format, lint, test, or release them from this repo.
`soothe-nano` and `soothe-deepagents` are **PyPI dependencies** (maintain/release
in their own repositories).

> `soothe-sdk` keeps its own `VERSION` file (1.x line) because
> `soothe-nano` (PyPI) depends on `soothe-sdk>=1.0.7`. All other owned
> packages use the root `VERSION` file (0.x line).

#### Dependency DAG (allowed direction only)

```text
soothe-sdk            ← shared contracts (monorepo; leaf)               ← OWNED
soothe-deepagents     ← deepagents fork (PyPI; leaf)
        ↓
soothe-nano           ← Coding CoreAgent (PyPI)
        ↓
soothe                ← host: StrangeLoop, CE, runner                   ← OWNED
        ↓
soothe-autopilot      ← goal orchestration: Autopilot, rails, verify    ← OWNED
        ↓
soothe-daemon         ← soothed process (channels, cron)                ← OWNED
        ↑
soothe-client-python  ← WebSocket transport (submodule)
        ↑
soothe-cli            ← Typer + Textual TUI                             ← OWNED
```

#### Placement (where new code goes)

| Concern | Package |
|---------|---------|
| Shared events, wire, display, plugin contracts, protocols | `soothe-sdk` (monorepo; `packages/soothe-sdk`) |
| Coding CoreAgent, skills/MCP/backends used in-proc | `soothe-nano` (PyPI; mirasoth/soothe-nano) |
| StrangeLoop, Context Engine, identity, host runner | `soothe` |
| Autopilot (goal scheduling, dispatch, monitor, rails, verify, notify) | `soothe-autopilot` |
| Process lifecycle, channels, HTTP/WS server, admin IO, cron | `soothe-daemon` |
| Human CLI / TUI | `soothe-cli` |
| Language WS clients | `client/*` (external submodules) |

#### Import allow / deny (MUST) — monorepo-owned packages

| Package | May import | Must NOT import |
|---------|------------|-----------------|
| `soothe-sdk` | (leaf: `pydantic`, `langchain-core` only) | `soothe`, `soothe_autopilot`, `soothe_daemon`, `soothe_cli` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mirasoth/soothe](https://github.com/mirasoth/soothe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
