---
trigger: always_on
description: Self-improving browser automation via the auto-research loop. Iteratively runs a browsing task, reads the trace, and improves the navigation skill (strategy.md) until it reliably passes. Supports parallel runs across multiple tasks using sub-agents. Use when you want to build or improve browser automation skills for specific website tasks.
---


# AutoBrowse — Self-Improving Browser Skill

Build reliable browser automation skills through iterative experimentation. An inner Codex agent browses the site (`evaluate.mjs`). You — the outer agent — read what happened and improve the instructions (`strategy.md`). Repeat until it passes consistently.

## Entry Points

Invocation is flexible — both explicit flags and free-form natural language work:

```
/autobrowse --task google-flights
/autobrowse --task google-flights --iterations 10 --env remote
/autobrowse --task google-flights --browser-trace
/autobrowse --tasks google-flights,amazon-add-to-cart
/autobrowse --all

# Also fine — parse freely:
/autobrowse https://flights.google.com/
/autobrowse book a flight on delta.com
/autobrowse fix the existing google-flights skill
```

`--browser-trace` (default off, remote-only): pairs each iteration with the sibling `browser-trace` skill — wraps the inner agent in a CDP capture for per-page network/console/page-lifecycle evidence. Implies `--env remote`; errors if combined with `--env local`. Requires the sibling `browser-trace` skill present at `${CODEX_SKILL_DIR}/../browser-trace/`, and the `BROWSERBASE_API_KEY` env var.

When the user drops a URL or free-form instruction instead of `--task <name>`:
- If an existing task in `${WORKSPACE}/tasks/` clearly matches the site/intent, use it.
- Otherwise, pick a short kebab-case name, create `${WORKSPACE}/tasks/<name>/task.md` from `${CODEX_SKILL_DIR}/references/example-task.md`, fill in the URL/goal based on what the user said, and proceed. Tell the user the chosen name in one line.

---

## How to run

### Step 1 — Parse arguments and orient

Check what was passed:
- `--task <name>` → single task mode
- `--tasks a,b,c` or `--all` → multi-task mode (spawn sub-agents)
- `--iterations N` → how many evaluate → improve cycles (default: 5)
- `--env local|remote` → browser environment (default: local; use remote for bot-protected sites)
- `--browser-trace` → opt in to the browser-trace integration (default off). Implies `--env remote`. If `--env local --browser-trace` are both passed explicitly, error with: `browser-trace requires Browserbase; drop --env local or drop --browser-trace.`

If the user passed free-form text instead, map it to one of the above before continuing.

### Step 2 — Set up the workspace

All training artifacts (task definitions, strategy iterations, traces, reports) live in a workspace directory in the **current working directory** — NOT inside `~/.codex/skills/`. This keeps the inner agent's file writes out of Codex's home dir and away from permission friction.

Default workspace: `${CWD}/autobrowse/`

```bash
mkdir -p ./autobrowse/tasks ./autobrowse/traces ./autobrowse/reports
```

If the task directory (`./autobrowse/tasks/<task>/task.md`) doesn't exist yet, scaffold it:

```bash
mkdir -p ./autobrowse/tasks/<task>
cp ${CODEX_SKILL_DIR}/references/example-task.md ./autobrowse/tasks/<task>/task.md
# Then edit task.md to describe the URL, inputs, steps, and expected JSON output
```

The skill source at `${CODEX_SKILL_DIR}` stays read-only — only `./autobrowse/` in CWD gets written to during training. Graduation (final step) writes a single file to `~/.codex/skills/<task>/SKILL.md`.

List available tasks:
```bash
ls ./autobrowse/tasks/
```

### Step 3 — Multi-task: spawn parallel sub-agents

If running multiple tasks, use the Agent tool to spawn one sub-agent per task simultaneously. Each sub-agent receives a self-contained prompt to run the full autobrowse loop for its task:

> "You are running the autobrowse skill for task `<name>`. Workspace: `<absolute-path-to-workspace>` (e.g. `/path/to/project/autobrowse`). Run `<N>` iterations of: evaluate → read trace → improve strategy.md → repeat. Use `--env <env>`. Pass `--workspace <workspace>` to every evaluate.mjs invocation. If the parent invocation used `--browser-trace`, you MUST use the traced-path block of the SKILL.md loop for every iteration (pre-create session, attach bb-capture, pass `--connect-url` to evaluate.mjs, stop+bisect, release) — do not fall back to the default single-command path. Follow the autobrowse loop instructions exactly.
>
> When graduating, install the skill to `~/.codex/skills/<task-name>/SKILL.md` with proper agentskills frontmatter (name + description). Do not just copy strategy.md — write a self-contained skill.
>
> At the end, output a structured summary with: task name, pass/fail on final run, total cumulative cost, iterations completed, per-iteration table (iter number, turns, cost, status, hypothesis tested), and 2-3 bullet key learnings."

Spawn all sub-agents in parallel, wait for all to complete, then collect their summaries and write the session report.

**For single task**, skip this step and run the loop directly below.

---

## The Loop (run this for each task)

### Iteration start


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anantgar/autobrowse_codex](https://github.com/anantgar/autobrowse_codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
