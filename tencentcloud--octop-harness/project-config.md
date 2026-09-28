---
trigger: always_on
description: Working handbook for AI coding agents in the `octop-harness` repository.
---

# AGENTS.md

Working handbook for AI coding agents in the `octop-harness` repository.

> This file is **how to change this repo**. `src/octop_harness/builtin/md_files/AGENTS.md` (shipped in the library and copied into end-user workspaces) is **how to use the agent this library creates**. They are not the same document — do not mix them.

## 1. Collaboration principles

> Favor caution over speed; trivial tasks may relax these rules. These principles complement [§12 Change workflow](#12-change-workflow) and [§13 Communication](#13-communication).

### Think before writing

- Read first: the docstring of the file you will change, neighboring implementations, and related specs.
- State assumptions up front; ask when unsure — do not guess. When several interpretations exist, list them and let the user choose — do not pick one silently.
- Suggest simpler approaches when they exist; push back when appropriate.
- Stop when blocked; name exactly what is unclear.

### Simplicity first

- Write the minimum code that solves the problem; no unrequested features, abstractions, or config knobs.
- Do not add defensive error handling for scenarios that cannot realistically happen.
- Trim the diff when it grows unnecessarily large.

### Surgical edits

- Touch only lines directly related to the task; do not opportunistically "clean up" nearby code, comments, or formatting.
- Do not refactor working code or unify style just because it differs from yours.
- Unrelated dead code: mention it, do not delete it proactively.
- Remove orphan imports, variables, and functions **you** introduced.

### Verifiable outcomes

| Task | Plan | Verify |
|------|------|--------|
| Bug fix | Write a failing test that reproduces → fix → run the suite | New test fails before the fix; `make test` passes after |
| New public API | Export from `__all__` in `__init__.py` → implement + docstring + tests + mention in README | `make all` green |
| Behavior change | Add/change tests first → then change the implementation | Targeted `pytest`, then `make all` |
| Refactor | `make test` before → refactor → `make test` after | Zero behavior change unless requested |

**Ship bar: `make all` green** (format + lint + typecheck + test — exactly what the pre-commit hook runs). For multi-step work, sketch a short plan:

```
1. Read resolve_path in backends/workspace.py → verify: understand root_dir vs workspace_dir
2. Implement xxx → verify: pytest tests/test_backends_utils.py -k xxx
3. Run the full gate → verify: make all
```

## 2. What this is

`octop-harness` is a published Python library (`pip install octop-harness`). It is a moderate wrapper around [LangChain Deep Agents](https://docs.langchain.com/oss/python/deepagents/) so users can create production-grade agents with very little code.

**It is a library, not an application.** Anything that looks like application-layer functionality (cron, MBTI onboarding, vector-memory persistence, HTTP server, user login, UI) does not belong here — those belong in hosts such as `Octop`. `octop-harness` provides the agent runtime; `octop-gateway` (the IM bridge) and Octop (the self-hosted platform) assemble applications on top of it.

## 3. Tech stack

| Layer | Technology |
|-------|------------|
| Language | Python 3.12+; every source file has `from __future__ import annotations` |
| Agent runtime | `deepagents>=0.7,<0.8` + `langchain` / `langchain-core` / `langgraph` |
| Config objects | `dataclasses` (`HarnessAgentConfig` / `ProviderConfig` / `ModelConfig`) with `to_dict` / `from_dict` |
| Optional extras | `cli` (click + rich + prompt-toolkit), `bedrock`, `remote-backends`, `docker`, `opensandbox`, `observability` (langfuse), `acp`, `desktop` (mss + pynput + pillow), `web-search-all`, `object-storage`, `all` |
| Ecosystem | `octop-memory`, `octop-browser`, `langchain-mcp-adapters` + `mcp` |
| Packaging | hatchling; local development with `uv` (`uv sync --group dev`) |
| Quality gates | ruff (lint + format, line length 120), `mypy --strict`, pytest + pytest-asyncio |

## 4. Package layout

```
src/octop_harness/
├── __init__.py             # Public exports + type stubs + lazy load of HarnessAgent / Workspace
├── _version.py             # Resolve [project].version from pyproject.toml via importlib.metadata
├── agent.py                # HarnessAgent (construct + invoke/stream + init())
├── manager.py              # HarnessAgentManager + multi-agent orchestration + team subsystem
├── registry.py             # In-memory agent registry (AgentEntry + storage CRUD)
├── request.py              # ChatRequest (messages/thread_id/user/source/...)
├── config/                 # HarnessAgentConfig + ProviderConfig + ModelConfig + env parsing
├── init.py                 # init_workspace + InitResult (workspace seeding)
├── backends/               # Backend string / dict → BackendProtocol instance
│   ├── workspace.py        # BackendWorkspace: L1 agent-storage facade
│   └── utils.py            # Path rules + backend I/O helpers + materialize_storage_path
├── llm/                    # ChatModelFactory (ProviderConfig → BaseChatModel) + model routing
├── middleware/             # SessionLogger / PII / memory / media offload, etc.
├── memory/                 # MemoryRuntime (octop-memory adapter)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TencentCloud/octop-harness](https://github.com/TencentCloud/octop-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
