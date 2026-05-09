---
trigger: always_on
description: > **Non-negotiable.** Auto-injected every message. Budget: <4KB. Detail in `claude/*.md` — read via `read_file`.
---

# Claude Code — {{PROJECT_NAME}} Instructions

> **Non-negotiable.** Auto-injected every message. Budget: <4KB. Detail in `claude/*.md` — read via `read_file`.

@claude/architecture.md
@claude/rules.md

## ⚠️ Mandatory Reads — You MUST consult before acting

**Session start →** read **`claude/tasks/todo.md`** (current task state) + **`claude/tasks/lessons.md`** (accumulated wisdom from past sessions).

| If task involves…                                                                                                             | YOU MUST read FIRST                                                                                                                 |
| ----------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| _anything_ (first action)                                                                                                     | `claude/tasks/todo.md` + `claude/tasks/lessons.md` + `claude/tasks/CLAUDE_ERRORS.md` + `claude/architecture.md` + `claude/rules.md` |
| build, test, CI, lint, format, migration, local dev                                                                           | `claude/build.md`                                                                                                                   |
| MR, PR, ticket, context management                                                                                            | `claude/templates.md`                                                                                                               |
| terminal, command, shell, subprocess, pager, interactive                                                                      | `claude/terminal-safety.md`                                                                                                         |
| CVE, dependency upgrade, security scan                                                                                        | `claude/cve-policy.md`                                                                                                              |
| plugin, claude-mem, hook coexistence, API quota, rtk, graphify                                                                | `claude/plugins.md`                                                                                                                 |
| structural query, call trace, codebase-memory, semantic search, cocoindex, code-review-graph, change risk, playwright, serena | `claude/plugins.md`                                                                                                                 |

<!-- {{DOMAIN_LOOKUP_TABLE}} -->

## Operating Protocol (always active)

1. **Plan first** — write plan to `claude/tasks/todo.md` before non-trivial tasks (3+ steps or architectural decisions).
2. **Use subagents** — offload research and exploration to subagents (saves main context window).
3. **Prove completion** — run tests, check logs, demonstrate correctness.
4. **No hacky solutions** — find the elegant way. Ask "is there a more elegant way?" before committing.
5. **Fix bugs autonomously** — don't ask, just fix. Zero context switching from the user.
6. **Mark progress** — check items in `claude/tasks/todo.md` as you go.
7. **Evidence-based** — verify before stating. Distinguish pre-existing issues from introduced ones.
8. **Maintain knowledge autonomously** — update `claude/*.md` when you discover stale info. Stale docs are bugs.

## Token Cost Strategy

- **Subagents** for exploration: `research` + `reviewer` + `plan-challenger` + `session-reviewer` + `security-auditor` run in isolated context — main stays clean. Quality-critical agents (reviewer, plan-challenger, security-auditor) declare their optimal model; lightweight agents (research, session-reviewer) inherit the session model for efficiency.
- **Effort levels**: quick commands (`/build`, `/lint`, `/test`) use `effort: low`; research (`/plan`, `/review`) use `effort: high`.
- **Compact-safe hooks**: `SessionStart` hooks inject project context on cold starts and after compaction.
- **Read domain docs on-demand** from the lookup table, not preemptively.
- **Discard after use**: tool outputs, file reads (re-readable), intermediate research. Keep only: decisions, corrections, test results.
- **`ultrathink`** on high-reasoning commands (`/plan`, `/review`, `/mr`, `/debug`) — extended thinking where it matters.
- **`!command` pre-fetching** on 6 commands — injects git/task data before Claude starts, eliminating setup tool calls.
- **`disable-model-invocation: true`** on side-effect commands — prevents accidental skill loading into context budget.
- **Short descriptions** — all command descriptions ≤127 chars. Front-loaded key use cases.

## Model Awareness

Agents declare their **optimal model** for maximum quality — and fall back to the session model when unavailable. This guarantees the best choice when multiple models are accessible (Anthropic API, Bedrock, Vertex), and full compatibility when only one model exists (Ollama, LM Studio, any local LLM).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brain-bootstrap/claude-code-brain-bootstrap](https://github.com/brain-bootstrap/claude-code-brain-bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
