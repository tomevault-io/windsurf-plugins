---
trigger: always_on
description: validates a script's shape for you, so four rules hold, enforced by
---

# harbor

Plugin for setting up Python simulation repos via uv and authoring RL tasks end-to-end.

## Hard constraints (apply to ALL tasks)

1. Generated `install.md` / `history.md` / `benchmark.md` MUST be English-only — regardless of chat language.
2. **Dispatch depth ≤ 2** (main → orchestrator → worker). Exactly ONE agent carries the `Agent` tool: `reward-tuning-agent`, which dispatches `reward-candidate-agent`. Every other agent is a leaf and does its delegated work itself; the main thread orchestrates the rest (`dependency-generator` → `benchmark-generator` → …). Enforced by `tests/contract/test_agent_depth.py`. Nested dispatch needs **Claude Code ≥ 2.1.219** — `/harbor:reward-tune` asserts it in pre-flight and stops rather than degrading.
3. **All plugin-generated files live under `<repo>/harbor/`** — except `scripts/_<family>_env.py` / `scripts/run_random.py` / `scripts/render_random.py` (user-facing smoke entry points). Each generator agent writes its receipts + metadata into **its own subdir**: `harbor/dependency-generator/{setup_uv.sh, probe.json, install_plan.json, install.md}`, `harbor/benchmark-generator/{benchmark-spec.json, task_overview.md, .task_list.json, history.md, benchmark.md}`, `harbor/rl-integration-generator/{rl-suite-spec.json, rl-integration.md, history.md}`. The shared RL training tree stays at the top level: training scripts at `<repo>/harbor/scripts/rl/`, configs at `<repo>/harbor/configs/rl/`, training output at `<repo>/harbor/outputs/`, the DataLogger at `<repo>/harbor/utils/data_logger.py`; the create-task workspace at `<repo>/harbor/create-task/`. There is NO shared `run-log/` folder — each agent's per-run process log is the `history.md` inside its own subdir. The folder name is `harbor/` (no dot) so it doubles as a valid Python package — imports like `from utils.data_logger import DataLogger` resolve against `<repo>/harbor/` after `sys.path.insert(0, HARBOR_ROOT)`.
4. Code style across main thread AND all subagents:
   - **Think before coding** — state assumptions explicitly; if uncertain, ask. Don't pick silently between alternatives.
   - **Simplicity first** — minimum code that solves the problem; no speculative features, abstractions, configurability, or error handling for impossible scenarios.
   - **Surgical changes** — touch only what the task requires; don't "improve" adjacent code, refactor things that aren't broken, or remove pre-existing dead code unless asked.
   - **Goal-driven execution** — define verifiable success criteria up front; loop until the verification check passes. Weak criteria like "make it work" are not acceptable.

---

## 6-layer mental model

The harness is structured as six layers with different cardinality, lifecycle, and mutability. Use this map when deciding where a new module belongs.

```
L1   AGENT (intelligence)         — Claude itself; not in code
L2   ENTRY POINTS                 — User-facing surfaces. commands/<name>.md and
                                    skills/<name>/SKILL.md are the SAME mechanism upstream
                                    (custom commands were merged into skills); both create
                                    /harbor:<name> and support the same frontmatter. Skills
                                    add only a per-entry-point directory for supporting files.
                                    harbor uses commands/ throughout: its supporting files
                                    (knowledge/references/, knowledge/templates/, knowledge/experiences/) are shared across
                                    entry points, not owned by one.
L3   SUBAGENTS (roles)            — agents/<name>.md   (fresh context, isolated agent loop)
L4   TOOLS (deterministic)        — scripts/<owner>/*.py + Bash + Read/Write/Edit
L5   SHARED KNOWLEDGE (read-only) — knowledge/templates/, knowledge/references/, knowledge/experiences/
L6a  WORKSPACE PROCESS LOGS       — <repo>/harbor/<agent>/history.md   (per-run, append-only, inside each agent's subdir)
L6b  WORKSPACE RECEIPTS           — <repo>/harbor/<agent>/{install,history,benchmark,rl-integration}.md  (end-of-run user summary, in each agent's subdir)
```

Decision rules when adding a new module:

```
Q1: Does the user invoke it directly in chat?    → L2 entry point
Q2: Multi-step reasoning + decisions?            → L3 subagent
Q3: Single deterministic input → output?         → L4 tool
Q4: Read-only doc / data?
    Q4.1: Cross-repo shared?                     → L5 shared knowledge
    Q4.2: Per-run process record?                → L6a process log
    Q4.3: End-of-run user-facing summary?        → L6b receipt
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [supersglzc/harbor-rl](https://github.com/supersglzc/harbor-rl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
