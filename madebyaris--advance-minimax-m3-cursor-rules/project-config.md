---
trigger: always_on
description: Cursor 3.7 orchestration guide: when to plan, when to delegate, nested subagents, multi-environment handoffs, /best-of-n, and Await for long-running branches.
---


# Cursor 3.7 Agent Orchestration

Current through **Cursor 3.7** (Jun 2026). Use this rule for deep or exhaustive tasks where coordination matters more than raw implementation speed.

## Core Idea

The main agent should own synthesis. Plans, subagents, and task tracking exist to reduce context load and make large work safer. On M3, the main agent also owns the 1M-context loader — see `minimax-m3-long-context`.

## Local, worktree, cloud, and SSH sessions

Cursor 3.7 emphasizes moving work between execution environments inside the Agents Window. Treat this as workflow guidance, not a requirement:

- Prefer **local** when you need fast edit–run–debug loops, full filesystem access, or machine-specific verification.
- Prefer **/worktree** (isolated git worktree) when the task must not collide with the main tree, when you are running parallel branches, or when you want a disposable environment.
- Consider **cloud** when a task should keep running while you are away, or when you want continuation without tying up the local machine — then **hand back to local** to validate on the real environment when claims are environment-specific.
- Consider **SSH** when the runtime is on a remote host (e.g. an edge box, a build farm, a server) and the work needs direct access.

Handoffs should still use explicit goals, owned paths, environment, model, and what was verified (see `agent-teams.mdc`).

## Parallel sessions and the sidebar

Cursor 3.7 can show **many agent sessions** at once (including from other surfaces — mobile, web, Slack, GitHub, Linear). That mirrors the idea of concurrent **`Task`** / subagent delegation: only parallelize **independent** slices, merge results in one synthesis step, and avoid two sessions racing the same files. The UI makes parallelism easier; it does not remove the need for clear ownership, non-overlapping paths, and one synthesis step.

## When to Plan

Plan when:
- the task is ambiguous
- multiple valid approaches exist
- many files or systems will change
- the user needs approval before implementation

Do not over-plan one-file fixes or straightforward edits.
Use the environment's active planning workflow or mode when it exists instead of assuming a specific planning tool name.

## When to Use Task (subagents)

In Composer-style Cursor agents the tool is usually **`Task`** (subagent-style delegation; types such as explore, debugger, verifier — see the live schema). Use it when:
- repo exploration is broad and would pollute main context
- independent investigations can run in parallel
- a verifier or debugger pass would help
- the task has a long-running research branch

Avoid delegation overhead for instant or light work.

When several independent branches exist, do not launch them serially by habit. Launch multiple `Task` calls together with separate scopes and merge their results in the main thread.

**Nested subagents (3.7):** a subagent can spawn its own `Task` children. Treat nesting as a depth budget, not a default — prefer one delegation layer; use two only when the child slice is still too broad. The main thread still owns synthesis, edits, and final verification; do not let nested branches drift into parallel implementation on the same files.

**Multitask Mode:** when the session is in Multitask Mode, background subagent runs are expected — set `run_in_background: true` on `Task` and `Await` the result instead of blocking the main thread.

## /best-of-n as an Orchestration Primitive

For high-stakes decisions (architecture, refactor approach, design parity), the `/best-of-n` command runs the same prompt across 2–4 models in parallel worktrees and then compares outcomes.

A small workflow for using it well:

```text
1. Pick 2-4 models (mix is fine: `composer-2.5` + MiniMax-M3 + a third)
2. Write one bounded prompt that names the question, the constraint, and the deliverable shape
3. Run /best-of-n in isolated worktrees
4. Read each result centrally; tag them with the model
5. Pick the strongest, merge complementary ideas, or escalate to the user with the comparison
```

Keep prompts bounded. `/best-of-n` is wasted when the prompt is vague.

## Await for Long-Running Branches

For dev servers, watchers, build jobs, parallel subagents, or CLI tools that take seconds-to-minutes, do not poll. Use the `Await` tool to wait for a background shell/subagent or a specific output token (`Ready`, `Compiled`, `Error`).

```text
1. Start the long-running work in the background
2. Await the result or a specific output string
3. If it fails, Await the error message and form the next hypothesis
```

This prevents the "waste 30s polling" failure mode and keeps the main thread responsive on M3.

## Delegation Pattern

```text
1. Define the slices
2. Delegate only independent work
3. Launch independent subagents concurrently when possible
4. Await long-running branches
5. Read subagent summaries
6. Synthesize decisions in the main thread
7. Implement and verify centrally
```

## Parallel Delegation Rules

- Give each subagent a bounded scope, not a vague restatement of the whole task.
- State exactly what the subagent should return: findings, file paths, risks, or a decision recommendation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m3-cursor-rules](https://github.com/madebyaris/advance-minimax-m3-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
