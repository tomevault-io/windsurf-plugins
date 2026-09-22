---
trigger: always_on
description: Rules and architectural commandments for AI coding agents working on this codebase. Direct, data-driven, zero fluff.
---

# AGENTS.md

Rules and architectural commandments for AI coding agents working on this codebase. Direct, data-driven, zero fluff.

---

## 1. Core Philosophy & Architectural Commandments

- **Simplify First**: Lean towards simplifying, not complicating. If a feature, branch, or approach is ad-hoc, hardcoded, or not essential, **remove it**.
- **No Brittle Reinventions / Always Use Standard Libraries**: Reject complex manual implementations or from-scratch logic when reliable, standard libraries can replace them.
  - **PydanticAI** owns the agentic loop, tool dispatch, and message history.
  - **PyGObject's `gi.events`** (or `gbulb` on older PyGObject) owns the asyncio + GLib event loop unification.
  - **GNU Radio Companion's Python API** owns flowgraph parsing, block parameter evaluation, and graph validation.
  - Never reimplement or wrap these in custom hand-rolled loops or shadow abstractions.
- **Zero Ad-Hoc Heuristics & Zero Folklore**: Never use magic string branches, per-scenario regex routing, hardcoded command lists, or prompt folklore. If logic is required, it must be one uniform mathematical or algorithmic rule applied identically to all cases (e.g. `keep_param` in `adapter/graph.py`).
- **No Backward Compatibility Shims**: Delete dead code completely. Do not write shims, dual-format persistence layers, or legacy bridges. Keep changes clean and direct.
- **Fix at the Source**: Correctness lives in the tool or handler that produces data, not in a downstream post-processing filter.
- **No Assumed Reasoning Failures**: Do not assume task failures are solely due to LLM reasoning. Audit the execution harness for context flooding, poor prompt construction, hidden ad-hoc logic, or silent error message clipping.

---

## 2. Scientific Experimentation & Verification Persona

- **Assume Nothing, Verify Everything**: Base every decision on grounded, empirical observations (code inspection, live data flow, runtime execution), never on assumptions, intent, or memory.
- **Evidence Before Assertions**: Every claim must cite verified runtime observations. A green unit test or successful exit code is necessary but not sufficient — verify the underlying data flow, payload shapes, and state transitions.
- **Never Trust Memory for APIs — Always Use Context7 & Skills**:
  - When working with external libraries, frameworks, SDKs, or tools (even well-known ones like Pydantic, PyGObject, GNU Radio, FastAPI, AnyIO), unconditionally query current documentation via `context7` MCP or specialized agent skills.
  - Always check and use the specialized project skills in `/home/mahmoud/Desktop/AI_Projects/qoherent/GRC_Agent/.agents` (`building-pydantic-ai-agents`, `pydantic-ai-harness`, `hermes-subagent`).
  - Never rely on internal training memory, which may be outdated or hallucinated.
- **Empirical & Collaborative Decisions**: Stop and ask for clarification when requirements are ambiguous or when a major decision (e.g. library selection, backend architecture, destructive migration) needs to be made.

---

## 3. General Best Coding Practices

- **Git Workflow: Single-Branch Main Only**: Always work directly on `main` — never create feature branches, worktrees, or any alternative branch topology. Assume every change in the working tree (committed or uncommitted) is the user's own work: never stash, revert, exclude, or path-limit around it, and always include all current changes in any push. Commits stay conventionally scoped in message only, never in file ownership.
- **No Silent Transformations or Hidden Truncation**: Any filtering, truncation, or omission in model-facing output must be explicit, truthful, and honest (`output_truncated`, explicit counts). Never silently drop data.
- **Maximizing Context Honestly**: Do not enforce arbitrary context limits beyond what the backend actually supports. Never clip inputs or outputs using raw string slicing that breaks structured context (JSON/AST).
- **Atomic Operations & Concurrency Safety**:
  - All file writes must be atomic (temp file → fsync → `os.replace`).
  - Thread safety must be explicit: use `threading.Lock` for cross-worker process spawning (e.g. `ensure_server()`) and `asyncio.Lock` for async token refreshes.
  - Flowgraph disk access is locked via `fcntl.flock` on `.grc_agent/<name>.lock`.
- **Uniform Error Reporting — `ModelRetry` vs `ToolFailed`**: Every model-facing domain tool reports failure through one of two framework exceptions, chosen by what the model should do next.
  - `ModelRetry` — the model can fix this by trying again with different arguments or a different approach (bad param, invalid connection, validation failure). Carries actionable compiler/runtime feedback and consumes the tool's retry budget.
  - `ToolFailed` — terminal: no retry can fix it (an unwired run monitor, missing execution or save capability). The model sees a failed result and adapts. It consumes no retry budget, so repeated terminal failures are bounded at the run level instead: `StopGracefully.max_repeated_failures` ends the run after a tool fails terminally 3 times in a row.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qoherent/GRC-Agent](https://github.com/qoherent/GRC-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
