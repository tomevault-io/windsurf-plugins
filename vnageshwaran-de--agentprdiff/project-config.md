---
trigger: always_on
description: You are an AI coding agent (Claude Code, Cursor, Aider, Copilot, or similar) and the user has asked you to add `agentprdiff` to a repository, write a regression suite for an existing LLM agent, or fix an existing suite. This document is your brief.
---

# Adopting agentprdiff — instructions for AI coding agents

You are an AI coding agent (Claude Code, Cursor, Aider, Copilot, or similar) and the user has asked you to add `agentprdiff` to a repository, write a regression suite for an existing LLM agent, or fix an existing suite. This document is your brief.

It is dense by design. Each section is self-contained and grep-friendly. Read it linearly the first time; on subsequent runs jump to the section you need.

For the canonical file layout (what files to create, what's mandatory vs optional, what each contains), see [`docs/suite-layout.md`](./docs/suite-layout.md). Use it as the spec for your final diff before you tell the user you're done.

If you are a human reader, the same content is here but `docs/writing-suites.md` (forthcoming) is friendlier prose. This file is optimized for an agent that wants to copy templates and substitute values.

---

## What agentprdiff is, in one paragraph

`agentprdiff` is a regression-testing tool for non-deterministic LLM agents. The user writes a small Python file (a "suite") that lists `case(input, expect=[...])` entries. The first time `agentprdiff record` is run, it captures a JSON "baseline" trace of every model and tool call the agent made for each case. Baselines are committed to git. On every subsequent run (`agentprdiff check`, typically in CI), the new traces are diffed against the committed baselines; the build fails on regression. Think `git diff` for agent behavior, or Jest snapshot tests for `(model, prompt, tool_dispatch)` triples.

---

## Decision tree: should this project adopt agentprdiff?

Run through these in order. Stop at the first **NO** — agentprdiff is the wrong tool. Reach the end with all **YES** answers and proceed.

1. Does the project contain code that calls an LLM API (`openai`, `anthropic`, `langchain`, `google.genai`, a raw `requests.post` to an inference endpoint, …)? **YES** → continue. **NO** → stop. agentprdiff is for projects that ship LLM agents.
2. Does the agent's behavior depend on which model, prompt, or tool definitions are used? **YES** → continue. **NO** → stop. If the LLM call is incidental (one-off summarization, no tools, no system prompt the user controls), skip agentprdiff.
3. Is there at least one *contract* the agent must honor — "must call tool X first," "must not refund without verification," "must ask for missing inputs," "must stay under cost budget Y"? **YES** → continue. **NO** → stop. If you can't articulate a contract, you don't have a suite to write.
4. Is the agent already in production or about to be? Are model/prompt/SDK upgrades realistic in the project's lifetime? **YES** → continue. **NO** → consider deferring; agentprdiff's value compounds across changes.

If all four are YES, proceed.

---

## Canonical layout — what files you'll produce

Memorize this tree. Every adoption produces exactly this shape; deviating from it is a smell. Full per-file spec at [`docs/suite-layout.md`](./docs/suite-layout.md).

```
<project_root>/
├── suites/
│   ├── __init__.py                 ← optional (package marker)
│   ├── _eval_agent.py              ← MANDATORY (Step 3 produces this)
│   ├── _stubs.py                   ← MANDATORY iff side-effecting tools (Step 4)
│   ├── <project>.py                ← MANDATORY (Step 5 produces this)
│   ├── <project>_cases.md          ← MANDATORY (Step 5 produces this — case dossier)
│   └── README.md                   ← recommended
│
├── .agentprdiff/
│   ├── .gitignore                  ← auto-created by `agentprdiff init` (Step 6)
│   ├── baselines/<suite>/<case>.json   ← auto-created by `agentprdiff record` (Step 6)
│   └── runs/                       ← auto-created, NEVER committed
│
└── .github/workflows/
    └── agentprdiff.yml             ← strongly recommended (Step 7)
```

| File | Status | Created by |
|---|---|---|
| `suites/<project>.py` | MANDATORY | you |
| `suites/<project>_cases.md` | MANDATORY (the case dossier — see Step 5) | you |
| `suites/_eval_agent.py` | MANDATORY | you |
| `suites/_stubs.py` | MANDATORY iff side-effecting tools exist | you |
| `suites/__init__.py` | optional | you |
| `suites/README.md` | recommended | you |
| `.agentprdiff/.gitignore` | MANDATORY | `agentprdiff init` |
| `.agentprdiff/baselines/<suite>/<case>.json` | MANDATORY (one per case) | `agentprdiff record` |
| `.agentprdiff/runs/` | auto-created | `agentprdiff check` (NEVER commit) |
| `.github/workflows/agentprdiff.yml` | strongly recommended | you |

The Steps below produce these files in order. When you finish, your final `git status` should show *only* these paths added — no production-code modifications.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vnageshwaran-de/agentprdiff](https://github.com/vnageshwaran-de/agentprdiff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
