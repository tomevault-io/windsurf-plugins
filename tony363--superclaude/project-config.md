---
trigger: always_on
description: - **Never run destructive or bulk-reset commands** (`git checkout -- <path>`, `git reset --hard`, `git clean -fdx`, `rm -rf`, etc.) unless the user explicitly instructs you to do so for that exact path.
---

# SuperClaude Repository Guidelines

## Command Safety Rules

- **Never run destructive or bulk-reset commands** (`git checkout -- <path>`, `git reset --hard`, `git clean -fdx`, `rm -rf`, etc.) unless the user explicitly instructs you to do so for that exact path.
- **Never use `git checkout`, `git restore`, or similar commands to revert tracked files** unless the user explicitly requests it. These commands can silently discard changes created by other agents.
- **Prefer targeted edits** over repo-wide operations. Do not reset or roll back large sections of the codebase to "undo" mistakes.
- **Treat uncertainties as denials.** When in doubt about whether a command is destructive, stop and ask the user.

## Web Search via Rube MCP

Built-in WebSearch is DISABLED. Use Rube MCP's LINKUP_SEARCH for all web searches:

```
mcp__rube__RUBE_MULTI_EXECUTE_TOOL({
  tools: [{ tool_slug: "LINKUP_SEARCH", arguments: { query: "...", depth: "deep", output_type: "sourcedAnswer" }}],
  session_id: "WEB-SESSION-001", memory: {}, sync_response_to_workbench: false,
  thought: "Searching for [topic]", current_step: "SEARCHING",
  current_step_metric: {completed: 0, total: 1, unit: "searches"}, next_step: "COMPLETE"
})
```

## Agent System

SuperClaude uses a **3-tier agent system** — filesystem-based personas in `agents/`. Agents are markdown files with YAML frontmatter, NOT instantiated classes. They serve as prompting blueprints that guide Claude's behavior.

### Tiers

| Tier | Path | Purpose | Count |
|------|------|---------|-------|
| **core** | `agents/core/` | Primary personas (architect, developer, guardian, optimizer, communicator) | 5 |
| **trait** | `agents/traits/` | Composable modifiers applied to any agent (security-first, test-driven, mcp-pal-enabled, etc.) | 7 |
| **extension** | `agents/extensions/` | Domain specialists (python-expert, rust-expert, ml-engineer, kubernetes-specialist, etc.) | 7 |

### Frontmatter Schema

```yaml
---
name: lowercase-hyphenated       # Required. Must match NAME_PATTERN: ^[a-z][a-z0-9-]*[a-z0-9]$
description: One-line purpose    # Required. Max 200 chars recommended.
tier: core|trait|extension       # Required.
category: string                 # Optional. Organization label.
triggers: [keyword, list]        # Recommended for core/extension. NOT for traits.
tools: [Read, Write, Edit, ...]  # Optional. Claude Code tool names.
traits: [security-first, ...]    # Optional. Compose traits into core/extension agents.
priority: 1-3                    # Optional (core/extension only). NOT for traits.
---
```

### Selection

Agent selection is handled by `sc-implement/scripts/select_agent.py` (standalone, JSON I/O). Scoring: 35% keywords + 25% domains + 20% task type + 10% files + 10% priority. Traits are applied additively; conflicts (e.g., performance-first vs security-first) are detected.

### Creating an Agent

1. Create `agents/{tier}/your-agent.md` with frontmatter matching the schema above
2. Add the agent body (markdown) with expertise, patterns, and behavioral instructions
3. Validate: `python scripts/validate_agents.py --verbose`

## Build & Test

```bash
# Bootstrap
python -m venv .venv && source .venv/bin/activate && pip install -e .[dev]

# Tests
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 pytest -m "not slow" tests/

# Lint
ruff check . && ruff format --check .

# Agent validation
python scripts/validate_agents.py --verbose
```

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **SuperClaude** (6382 symbols, 16783 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## When Debugging

1. `gitnexus_query({query: "<error or symptom>"})` — find execution flows related to the issue
2. `gitnexus_context({name: "<suspect function>"})` — see all callers, callees, and process participation
3. `READ gitnexus://repo/SuperClaude/process/{processName}` — trace the full execution flow step by step
4. For regressions: `gitnexus_detect_changes({scope: "compare", base_ref: "main"})` — see what your branch changed

## When Refactoring


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tony363/SuperClaude](https://github.com/Tony363/SuperClaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
