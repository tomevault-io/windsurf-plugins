---
trigger: always_on
description: trace-mcp is a framework-aware code intelligence MCP server. It indexes source code into a dependency graph with full-text search, understanding 48+ frameworks across 68 languages. It exposes MCP tools for navigation, impact analysis, and framework-specific queries.
---

# trace-mcp Development Guide

## What this project is

trace-mcp is a framework-aware code intelligence MCP server. It indexes source code into a dependency graph with full-text search, understanding 48+ frameworks across 68 languages. It exposes MCP tools for navigation, impact analysis, and framework-specific queries.

## Agent Behavior — read before every task

These rules override tool routing and workflow checklists when in conflict.

### No flattery, no filler
Skip openers: "Great question", "You're absolutely right", "Excellent idea", "I'd be happy to", "Absolutely!". Start the response with the answer or the action. No ceremonial closings.

### Disagree when the premise is wrong
If the user's request rests on a false assumption, say so before doing the work. Agreeing with wrong premises to be polite is worse than pushback. Example: user asks to optimize a function that's not on the hot path — say that, then ask whether to proceed anyway.

### Never fabricate
Not file paths, not symbol names, not API signatures, not test output, not commit hashes. If unsure, call `search` / `get_symbol` / run the command. "I don't know, let me check" beats a plausible-sounding guess.

### Stop when confused
If the task has two plausible interpretations and the choice materially changes the diff, ask. Do not pick silently and proceed. Trivial/reversible tasks (typo, rename local var) — proceed.

### Goal-driven execution
Rewrite vague asks into verifiable goals before writing code:
- "Add validation" → "Write failing tests for invalid inputs (empty, malformed, oversized), make them pass."
- "Fix the bug" → "Write a failing test reproducing the symptom, make it pass."
- "Refactor X" → "Ensure existing test suite passes before and after, no public API changes."
- "Make it faster" → "Benchmark current hot path, identify bottleneck with profiling, show benchmark improved."

Never report "done" based on a plausible-looking diff. Run the test/build/typecheck. Plausibility is not correctness.

### 2-strike session rule
After two failed attempts at the same issue, stop. Summarize what was tried, what was learned, and ask the user to reset the session with a sharper prompt. Do not grind through a third attempt in a polluted context — fresh session + better prompt beats long session + accumulated failures.

### English only in code and content
All text committed to the repository — UI strings, button labels, tooltips, error messages, log output, code comments, docstrings, README/docs, commit messages — MUST be in English. This is a hard rule even when the user is chatting with you in another language. If the user writes in Russian, answer them in Russian, but every string you put in a file is English. No exceptions, including ad-hoc test fixtures or TODO comments. If unsure whether a fixture counts, default to English.

## Build & Test

```bash
npm run build          # TypeScript compilation
npm test               # Vitest (all tests)
npm test -- --run <pattern>  # Run specific test
```

## trace-mcp Tool Routing — MANDATORY (for AI agents working ON this codebase)

**HARD RULE: NEVER use Read, Grep, Glob, or Bash (ls, find, cat, head, tail) to explore or navigate source code (.ts, .js, .py, etc.). ALWAYS use trace-mcp tools instead. This is not a suggestion — it is a requirement. Violations waste tokens and produce worse results.**

Since trace-mcp is its own MCP server, when developing it you MUST use trace-mcp tools to navigate the codebase:

### Decision Matrix — USE THESE, NOT native tools

| Task | trace-mcp tool | NEVER use |
|------|---------------|-----------|
| Find a function/class/method | `search` | ~~Grep~~ ~~Glob~~ |
| Understand a file before editing | `get_outline` | ~~Read (full file)~~ |
| Read one symbol's source | `get_symbol` | ~~Read (full file)~~ |
| What breaks if I change X | `get_change_impact` | ~~guessing~~ |
| Who calls this / what does it call | `get_call_graph` | ~~Grep~~ |
| Find all usages of a symbol | `find_usages` | ~~Grep~~ |
| Get context for a task | `get_feature_context` | ~~reading 15 files with Read~~ |
| Find tests for a symbol | `get_tests_for` | ~~Glob + Grep~~ |
| Find untested symbols (deep) | `get_untested_symbols` (classifies "unreached" vs "imported_not_called") | ~~manual audit~~ |
| Project overview | `get_project_map` (summary_only=true) | ~~Bash ls~~ |
| List files in a directory | `get_outline` or `search` | ~~Bash ls/find~~ ~~Glob~~ |
| Find where something is defined | `search` | ~~Grep~~ |
| Understand project structure | `get_project_map` | ~~Bash find~~ |
| Move a symbol between files | `apply_move` { symbol_id, target_file } | ~~manual cut/paste + Edit~~ |
| Move/rename a file | `apply_move` { source_file, new_path } | ~~manual mv + Edit imports~~ |
| Change function signature | `change_signature` { symbol_id, changes } | ~~manual Edit across files~~ |
| Preview any refactoring | `plan_refactoring` { type, ... } | ~~guessing~~ |

### Token optimization — MANDATORY


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikolai-vysotskyi/trace-mcp](https://github.com/nikolai-vysotskyi/trace-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
