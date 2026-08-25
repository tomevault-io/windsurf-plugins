---
trigger: always_on
description: <!-- OPENCODE-MEMORY:START -->
---

<!-- OPENCODE-MEMORY:START -->
<!-- Managed block - do not edit manually. Updated by: nano-brain skill -->

## Memory System (nano-brain)

This project uses **nano-brain** for persistent context across sessions. Agents talk to the daemon via the registered MCP server `nano-brain` (streamable HTTP at `/mcp`).

### Quick Reference

All operations are MCP tool calls. Every tool takes a `workspace` (SHA-256 hash; resolve once via `memory_workspaces_resolve` with `{path: "<project root>"}`).

| I want to... | MCP tool | Required args |
|---|---|---|
| Recall past work on a topic | `memory_query` | `workspace`, `query` |
| Find exact error/function name | `memory_search` | `workspace`, `query` |
| Explore a concept semantically | `memory_vsearch` | `workspace`, `query` |
| Save a decision for future sessions | `memory_write` | `workspace`, `content` (+ `tags`, `title`) |
| Catch up at session start | `memory_wake_up` | `workspace` |
| Fetch one doc by ID/path | `memory_get` | `workspace`, `path` |
| Check daemon health | `memory_status` | (none) |
| List all registered workspaces | `memory_workspaces_list` | (none) |

### Session Workflow

**Start of session:** List workspaces first, then resolve and query.

```
memory_workspaces_list()                    // → list all workspaces with paths
memory_workspaces_resolve(path="<path>")   // → workspace hash (if you know the path)
memory_wake_up(workspace=<hash>, limit=8)
memory_query(workspace=<hash>, query="<task topic>")
```

**If you don't know the workspace path:**
```
memory_workspaces_list()  // → find workspace by name or path
```

**End of session:** Save key decisions, patterns discovered, and debugging insights.

```
memory_write(
  workspace=<hash>,
  content="## Summary\n- Decision: ...\n- Why: ...\n- Files: ...",
  tags=["summary", "decision"],
  collection="memory"
)
```

### Code Intelligence Tools

Symbol-level analysis (requires the workspace to be indexed by the daemon's watcher — check `memory_status.queue_pending` if results are empty).

| I want to... | MCP tool |
|---|---|
| Find 1-hop callers/callees of a symbol | `memory_graph` |
| Assess risk of changing a symbol (reverse impact BFS) | `memory_impact` |
| Find what a symbol depends on (forward traversal) | `memory_impact` (direction="out") |
| Trace forward call chain from an entry point | `memory_trace` |
| Find a symbol by name/kind across the workspace | `memory_symbols` |

### When to Search Memory vs Codebase vs Code Intelligence

- **"Have we done this before?"** → `memory_query` (searches past sessions + decisions)
- **"Where is this in the code?"** → grep / ast-grep (searches current files)
- **"How does this concept work here?"** → both (memory for past context + grep for current code)
- **"What calls this function?"** → `memory_graph(node="<name>", direction="in")`
- **"What breaks if I change X?"** → `memory_impact(node="<name>", max_depth=2)`
- **"What does X depend on?"** → `memory_impact(node="<name>", direction="out")`
- **"Walk the call chain from entry point X"** → `memory_trace(node="<name>", max_depth=5)`

See `.opencode/skills/nano-brain/SKILL.md` for the full reference (all MCP tools, recipes, troubleshooting).

<!-- OPENCODE-MEMORY:END -->

## RRI-T Test Instance

For RRI-T testing (skill: `rri-t-testing`), use a **separate nano-brain instance on port 8899** to avoid clashing with the default 3100 server (another process in this container uses 3100).

- **Custom config**: `/tmp/nano-brain-custom/config.yml` (port 8899, isolated logs/summaries dir)
- **Launch**:
  ```bash
  NANO_BRAIN_CONFIG=/tmp/nano-brain-custom/config.yml ./nano-brain serve
  ```
- **Health check**: `curl -s http://localhost:8899/api/status`
- **Precedence**: `--config` flag > `NANO_BRAIN_CONFIG` env > `~/.nano-brain/config.yml` (default)

Never run RRI-T against the default 3100 instance — it pollutes production memory and conflicts with the sibling process.

<!-- BEHAVIORAL-GUIDELINES:START -->
# Behavioral Guidelines (Always Apply)

Reduce common LLM coding mistakes. Apply to every task regardless of scope.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nano-step/nano-brain](https://github.com/nano-step/nano-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
