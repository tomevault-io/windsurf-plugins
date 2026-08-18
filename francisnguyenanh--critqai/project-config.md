---
trigger: always_on
description: > **CritiqAI** is a multi-agent peer-review simulator built with Google ADK.
---

# AGENTS.md — CritiqAI

> **CritiqAI** is a multi-agent peer-review simulator built with Google ADK.
> It assigns AI personas to challenge student essays using Socratic questioning —
> never revealing answers, only probing reasoning.
>
> **Core pitch:** *Using AI to teach students NOT to depend on AI.*

---

## Architecture Overview

```text
                         ┌─────────────────────────────────────────────────────┐
                         │                   CritiqAI System                   │
                         │                                                     │
    Student/Teacher ────►│  FastAPI Web UI  ◄──────►  DebateSessionManager     │
                         │  (web_app.py)              (session_manager.py)     │
                         └──────────────────────────────────┬──────────────────┘
                                                            │
                   ┌────────────────────────────────────────▼────────────────────────────────────┐
                   │                      Google ADK Agent Pipeline (6 agents)                  │
                   │                                                                             │
                   │   ┌────────────┐     ┌────────────────┐                                    │
                   │   │ Summarizer │────►│ Persona        │                                    │
                   │   │  Agent     │     │ Selector Agent │                                    │
                   │   └──────┬─────┘     └───────┬────────┘                                    │
                   │          │ essay_summary      │ personas[]                                 │
                   │          └────────────────────▼                                            │
                   │                        ┌─────────────┐   generate    ┌──────────────────┐  │
                   │                        │  Debate     │──────────────►│   Challenge      │  │
                   │                        │  Agent      │◄── student    │   Validator      │  │
                   │                        │  (3 rounds) │    response   │   Agent          │  │
                   │                        └─────────────┘               └────────┬─────────┘  │
                   │                                                   pass/retry  │ challenge  │
                   │                                                               ▼  to student│
                   │                                              ┌────────────────────────┐    │
                   │                                              │ transcript (validated) │    │
                   │                                              └───────────┬────────────┘    │
                   │                                                          │                 │
                   │                                                 ┌────────▼──────┐          │
                   │                                                 │  Analytics    │          │
                   │                                                 │  Agent        │          │
                   │                                                 └────────┬──────┘          │
                   │                                                          │ scores+analysis │
                   │                                                 ┌────────▼──────┐          │
                   │                                                 │  Report       │──► Gmail │
                   │                                                 │  Agent        │  (HITL)  │
                   │                                                 └───────────────┘          │
                   └─────────────────────────────────────────────────────────────────────────────┘

MCP Servers
───────────
  [HTTP/SSE]  argument-scorer (Cloud Run)  ──►  Analytics Agent  (hybrid: 0 tokens EN · ~300 tokens non-EN, LRU-cached)
  [SSE]       Google Drive MCP  ──►  read essay from Google Docs
  [SSE]       Google Sheets MCP ──►  append debate log rows
  [SSE]       Google Gmail MCP  ──►  create_draft only (compose scope)

  Local dev fallback: argument-scorer can run as stdio subprocess or be imported directly.

Key multi-agent interaction:
  Debate Agent generates a challenge → Validator Agent independently reviews it
  (deterministic: answer-leak check + single-question rule + length check) → passes or
  triggers regeneration (max 1 retry, zero LLM tokens). Two agents collaborating, not just a chain.
```

---

## Agents

### 1. Summarizer Agent

| Property | Value                                                                          |
| -------- | ------------------------------------------------------------------------------ |
| File     | `agents/orchestrator.py` → `run_summarizer()`                                  |
| Model    | `gemini-2.5-flash-lite`                                          |
| Input    | Full essay text (up to 2000 words, sanitized)                                  |
| Output   | `{main_claim, supporting_points[], evidence[], conclusion}`                    |
| Purpose  | Compress essay to ~200 tokens before downstream agents; reduces token cost 75% |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [francisnguyenanh/CritqAI](https://github.com/francisnguyenanh/CritqAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
