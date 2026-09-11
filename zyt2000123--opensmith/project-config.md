---
trigger: always_on
description: This file is the working brief for Codex or any coding agent operating in this repository.
---

# AGENTS.md

This file is the working brief for Codex or any coding agent operating in this repository.

**Trust code over docs.** `docs/` has not been kept in sync with refactors. When
this file and the code disagree, the code wins — and fix this file. Count things
by the consumer's criterion, never by directory listing: `engine/tool/registry.py`
accepts a tool only if it defines `TOOL_META` + `execute`; a skill counts only if
its directory holds a top-level `SKILL.md`.

## 1. What This Project Is

Agent-Smith is a local-first personal assistant Agent workbench that runs in the terminal.

- Smith is the single, always-on Agent
- Smith uses the skill system to switch workflows per task type
- No sub-agents, no multi-agent routing — one Agent, one conversation, accumulating memory over time

One-line:

> Agent-Smith is a local-first Agent workbench. Smith is your single resident
> assistant — it keeps context, accumulates memory, and switches workflows via skills.

## 2. Current Priority

The current priority is the terminal workbench experience:

1. Ink shell (`shell/`) is the sole entry point and provides the rich terminal UI
2. Skill-based workflow switching works for different task types
3. Memory accumulation across sessions

## 3. Agent Architecture

```
User ──▶ Ink shell ──HTTP + SSE──▶ server ──▶ engine
                                                │
              identity_catalog (agents/identities/*.yaml)
       keyword score match, priority breaks ties ──▶ RouteDecision
                                                │
                        ┌───────────────────────┴──────────────────┐
                        ▼                                          ▼
                 no pipeline matched                     pipeline route matched
                 direct ReAct loop                   (agents/pipelines/<id>.yaml)
                                                                   │
                                  skill chain nodes, one gate per node
                                  gate fails ──▶ backtrack to an earlier node
```

Declared routes: identity `smith` (default) has `git` (no pipeline); identity
`coding` has `requirements-research`, `tdd-development`, `code-review`, each →
the same-named pipeline. Ordinary coding requests intentionally stay direct
ReAct (see `agents/identities/coding.yaml` instructions). A pipeline whose
skills are not installed falls back to direct ReAct for the whole run, and
ROUTE_DECIDED reports the fallback (no pipeline) rather than the skipped chain.

Prompt assembly (`engine/context/assembler.py`) stacks 16 trust-tagged layers:
Agent Role / Style / Workflow, Tool Usage Policy, Available Tools, Available
Skills, Learned User Context, Global Instructions, Project Instructions,
Identity Guidance, Evaluation Safety Guidance (conditional), Output Style,
Memory Governance, Durable Memory, Runtime Context, Engine Runtime Control.

Memory has two rendered views: `context.md` (user collaboration) and
`memory/durable.md` (project memory). Both are budget-capped and injected in
full. `memory/recent.jsonl` is the evidence log; there is no query-time memory
retrieval, FTS index, embeddings, or episode layer.

## 4. Product Language

Use: "Smith", "Agent", "skill", "session", "memory", "tool", "template"

Avoid: "sub-agent", "employee", "digital employee", "hire"

## 5. Architecture Boundaries

Four layers, one-way dependencies (verified by import graph, not by convention):

```
server/ → engine/ → common/
          ↑
        agents/   (loaded at runtime, never imported)
```

Plus `shell/` as the terminal frontend (Ink/React, calls server over HTTP).

| Layer | Directory | Source lines | Responsibility |
|---|---|---|---|
| Infrastructure | `common/` | 269 | Paths, SQLite connection, YAML read/write. Zero business logic. |
| Execution | `engine/` | 16.5k | Agent framework: LLM, pipeline + ReAct, memory, skills, tools, safety, observability. Zero platform knowledge. |
| Content | `agents/` | 4.0k | Smith identity seed, pipelines, gates, tools, skills, safety rules. Pure content. |
| Platform | `server/` | 5.9k | FastAPI. Orchestration, session/agent lifecycle, 34 HTTP endpoints. |
| Terminal UI | `shell/` | 9.3k TS | Ink shell. Calls server over HTTP, auto-starts the backend. |

Rules:

- `engine/` must not know FastAPI, HTTP, or agent instance management
- `agents/` imports nothing from other layers — the tool registry loads its `.py`
  files via `exec_module`, so the contract is `TOOL_META` + `execute`, not types.
  A path constant cannot be shared into it; expect duplicated path derivation.
- `server/app/routers/` stays thin — extract params, call service, return result
- `server/app/` is the FastAPI application package; keep this conventional layout
- `agents/smith/` is where Smith's built-in identity seed lives
- New capabilities → add skills, not new agents

`common/paths.py` is the single source of truth for the runtime data root
(`~/.agent-smith`, enforced `0o700`/`0o600`). `engine/safety/tool_guard.py`
anchors its non-bypassable platform-write protection on it.

## 6. Files That Matter

| Area | Key Files |
|---|---|
| Terminal entry | `shell/bin/smith.js` → `shell/src/index.tsx` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zyt2000123/openSmith](https://github.com/zyt2000123/openSmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
