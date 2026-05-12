---
trigger: always_on
description: This repository is the **COC (Cognitive Orchestration for Codegen) setup** for building with the Kailash SDK — providing agents, skills, rules, and hooks for Kailash SDK development. All projects using this setup inherit these capabilities through the `.claude/` directory.
---

# Kailash COC Claude (Python)

This repository is the **COC (Cognitive Orchestration for Codegen) setup** for building with the Kailash SDK — providing agents, skills, rules, and hooks for Kailash SDK development. All projects using this setup inherit these capabilities through the `.claude/` directory.

## Absolute Directives

These override ALL other instructions. They govern behavior before any rule file is consulted.

### 0. Foundation Independence — No Commercial Coupling

Kailash Python SDK is a **Terrene Foundation project**. It is fully independent. There is NO relationship between Kailash Python SDK and any commercial product, proprietary codebase, or commercial entity. Do not reference, compare with, or design against any proprietary product. Do not use language like "open-source version of X" or "Python port of Y." Kailash Python SDK IS the product — not a derivative of anything. See `rules/independence.md` for full policy.

### 1. Framework-First

Never write code from scratch before checking whether the Kailash frameworks already handle it.

- Instead of direct SQL/SQLAlchemy/Django ORM → check with **dataflow-specialist**
- Instead of building API endpoints, web services, HTTP servers manually → check with **nexus-specialist**
- Instead of custom MCP server/client → check with **mcp-specialist**
- Instead of custom agentic platform → check with **kaizen-specialist**
- Instead of custom governance/access control → check with **pact-specialist**

### 2. .env Is the Single Source of Truth

All API keys and model names MUST come from `.env`. Never hardcode model strings like `"gpt-4"` or `"claude-3-opus"`. Root `conftest.py` auto-loads `.env` for pytest.

See `rules/env-models.md` for full details.

### 3. Implement, Don't Document

When you discover a missing feature, endpoint, or record — **implement or create it**. Do not note it as a gap and move on. The only acceptable skip is explicit user instruction.

See `rules/e2e-god-mode.md` and `rules/zero-tolerance.md` for enforcement details.

### 4. Zero Tolerance

Pre-existing failures MUST be fixed, not reported. Stubs are BLOCKED. Naive fallbacks are BLOCKED. SDK bugs get GitHub issues, not workarounds. See `rules/zero-tolerance.md`.

### 5. Recommended Reviews

- **Code review** (reviewer) after file changes — RECOMMENDED — see `rules/agents.md`
- **Security review** (security-reviewer) before commits — strongly recommended — see `rules/agents.md`
- **Real infrastructure recommended** in Tier 2/3 tests — see `rules/testing.md`

### 6. LLM-First Agent Reasoning

When building AI agents: **the LLM does ALL reasoning. Tools are dumb data endpoints.** No if-else routing, no keyword matching, no regex classification in agent decision paths. The LLM IS the router, classifier, extractor, and evaluator. Deterministic logic is BLOCKED unless the user explicitly opts in. See `rules/agent-reasoning.md` for the full rule and detection patterns.

## Workspace Commands

Phase commands replace the manual copy-paste workflow. Each loads the corresponding instruction template and checks workspace state.

| Command      | Phase | Purpose                                                    |
| ------------ | ----- | ---------------------------------------------------------- |
| `/analyze`   | 01    | Load analysis phase for current workspace                  |
| `/todos`     | 02    | Load todos phase; stops for human approval                 |
| `/implement` | 03    | Load implementation phase; repeat until todos done         |
| `/redteam`   | 04    | Load validation phase; red team with MCP tools             |
| `/codify`    | 05    | Load codification phase; create agents & skills            |
| `/release`   | —     | SDK release: PyPI publishing, docs deploy, CI (standalone) |
| `/ws`        | —     | Read-only workspace status dashboard                       |
| `/wrapup`    | —     | Write session notes before ending                          |
| `/journal`   | —     | View, create, or search project journal entries            |

**Workspace detection**: Hooks automatically detect the active workspace and inject context. `session-start.js` shows workspace status on session start (human-facing). `user-prompt-rules-reminder.js` injects a 1-line `[WORKSPACE]` summary into Claude's context every turn (survives context compression).

**Session continuity**: Run `/wrapup` before ending a session to write `.session-notes`. The next session's startup reads these notes and shows workspace progress automatically.

## Rules Index

| Concern                                         | Rule File                       | Scope                                                       |
| ----------------------------------------------- | ------------------------------- | ----------------------------------------------------------- |
| **Foundation independence**                     | `rules/independence.md`         | **Global — overrides all**                                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [terrene-foundation/kailash-coc-claude-py](https://github.com/terrene-foundation/kailash-coc-claude-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
