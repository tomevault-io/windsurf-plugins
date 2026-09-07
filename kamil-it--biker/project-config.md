---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Agent Policy

**Before implementing any multi-step task**, always call `mcp__ruflo__hooks_pre-task` with the task ID and description. It returns agent role suggestions — spawn those agents via `mcp__ruflo__agent_spawn` before writing code. Example flow:

1. `mcp__ruflo__hooks_pre-task` — get agent suggestions for this task
2. `mcp__ruflo__agent_spawn` — spawn one agent per suggested role (e.g. `backend-impl`, `tester`)
3. `mcp__ruflo__task_create` — register the task
4. Implement, then `mcp__ruflo__task_complete` when done

## ECC Skill Routing

Structured workflows for common task types. Use these skills in `.claude/skills/` when starting code changes.

**See `.claude/ECC_INTEGRATION_GUIDE.md` for complete documentation and how to download all 449 ECC skills.**

| Task | Skill | Use When |
|------|-------|----------|
| **Implement new endpoint** | `sparc-code` | Adding POST /v1/bike/* or /v1/equipment/* endpoint |
| **Add tests** | `sparc-tester` | After implementation, before PR; writes smoke tests in backend/scripts/test_search.py |
| **Security audit** | `sparc-security-review` | New endpoint, new finder module, or API integration (validates input, prevents prompt injection, checks error handling) |
| **Capture pattern** | `memory-persist` | After successful feature completion; saves reusable pattern to Obsidian vault for future tasks |

**Example workflow:**
```
User: "Add Decathlon offer endpoint"

1. Invoke /sparc:code
   → Specification → Pseudocode → Implementation → Testing phases
   → Creates app/bike_offer_decathlon_finder.py + POST /v1/bike/decathlon route

2. Invoke /sparc:tester
   → Adds test to backend/scripts/test_search.py
   → Verifies smoke test passes, cache works, error handling

3. Invoke /sparc:security-review
   → Validates input, prevents prompt injection, checks error responses
   → Runs pen-tests with cURL examples

4. Invoke /memory:persist
   → Saves "Offer finder pattern: single web_search + cache + fallback" to Obsidian
   → Next time: search vault → reuse template → save 1.5 hours

5. Create PR for review
```

**Skills in `.claude/skills/`:**
- `sparc-code.md` — Structured implementation phases + templates
- `sparc-tester.md` — TDD workflow + smoke test templates
- `sparc-security-review.md` — Security checklist + pen-test examples
- `memory-persist.md` — Obsidian vault integration for pattern capture
- **All 449 ECC skills** — Run `python fetch_ecc_skills.py` to download (see guide for details)

**ECC Overview:** 449 production-ready skills for backends (FastAPI, Django, etc.), frontends (React, Vue, etc.), testing, security, DevOps, and specialized domains. See `.claude/ECC_INTEGRATION_GUIDE.md` for complete reference.

## Memory & Persistence

Durable, cross-session memory for this project lives in a single human-readable Obsidian vault — **not** ruflo's `memory_*` / AgentDB stores. The vault is at `obsidian/bike-memory/` (gitignored, including its bearer token) with notes under a `memory/` folder. It is served by the `obsidian` MCP server (the "MCP Connector" plugin, `http://127.0.0.1:27200/mcp`) using local Transformers.js embeddings (`Xenova/all-MiniLM-L6-v2`, no API key).

**To recall:** `mcp__obsidian__search_vault_smart` (semantic) or `search_vault_simple` (keyword).
**To store:** `mcp__obsidian__create_vault_file` (path like `memory/<topic>.md`, with YAML frontmatter + tags).
**To read/update:** `get_vault_file`, `append_to_vault_file`, `patch_vault_file`, `list_vault_files`.

Prefer these over `mcp__ruflo__memory_store` / `memory_search` — the vault is the source of truth so everything stays in one syncable, greppable, Obsidian-browsable store. **Requires the Obsidian app running** with the MCP Connector plugin enabled; if the `obsidian` server is unavailable, say so rather than silently falling back to the ruflo DB.

## Backlog

Tasks are tracked in `/backlog/`. Naming convention:

- `TODO_<ID>_<TASK_NAME>.md` — task not yet started
- `DONE_<ID>_<TASK_NAME>.md` — completed task (rename the file, don't delete it)
- `TODO_ISSUE_<ID>_<NAME>.md` — reported bug/issue, not yet fixed
- `DONE_ISSUE_<ID>_<NAME>.md` — fixed issue (rename the file, don't delete it)

Layout:

```
backlog/            active work — TODO_* files live here
backlog/done/       merged tasks (DONE_*)
backlog/blocked/    implemented but unverifiable — still TODO_*
```

`backlog/` itself holds **only what is actionable now**. Scan it for the next task; the two subfolders are archives, not queues.

When picking up a task: read its file, implement, then rename `TODO_` → `DONE_` **and move it to `backlog/done/`**.
When creating a new task: ask clarifying questions first, then write the file.

### Completed tasks — `backlog/done/`

A task moves here when **its PR has merged to `main`**. Merged is the bar: finished code with an open PR is not done and stays in `backlog/`. Keep the files — they are the record of what was built and why. Update `backlog/done/README.md` when adding one.

### Blocked tasks — `backlog/blocked/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kamil-IT/biker](https://github.com/Kamil-IT/biker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
