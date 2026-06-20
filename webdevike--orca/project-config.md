---
trigger: always_on
description: 🐋 Workflow-agnostic multi-pane orchestrator. Spawns and coordinates worker sessions in cmux or tmux panes via pluggable playbooks. Use when the user invokes /orca, asks to "orchestrate", "manage workstreams in parallel", "run multiple agents", or wants something to keep multiple Claude/codex/dev-server sessions advancing without manual juggling. Orca picks a backend at runtime (cmux if available, else tmux), reads playbooks from .orca/playbooks/ (project) or ~/.orca/playbooks/ (global) or its bu
---


# 🐋 orca

Workflow-agnostic multi-pane orchestrator. Spawns workers in cmux or tmux panes, applies a **playbook** that defines the workflow, monitors progress, escalates when stuck.

## Core Constraint

**Orca never writes project code.** No Edit/Write/Bash on project files outside `.orca/`. If a task requires touching code, spawn a worker pane and delegate. The intelligence lives in playbooks — orca is just the muxer + dispatcher.

## On Invocation

| User says | Action |
|-----------|--------|
| `/orca` | Read `.orca/state.json` if present; otherwise enter planning mode (ask what to orchestrate) |
| `/orca <playbook> key=value …` | Apply named playbook with args (skip planning) |
| `/orca status` | Capture-pane each tracked worker, summarize signals |
| `/orca stop <pane-ref>` | Close one worker pane, update state |
| `/orca kill` | Close all orca-managed panes, clear state |
| `/orca --voice` (or `/orca --voice on`) | Enable voice I/O for the current orca pane (see [Voice mode](#voice-mode-orca---voice)). After processing the flag, continue with whatever else was on the command line — `/orca --voice` alone behaves like `/orca`; `/orca --voice gsd phase=2` applies the playbook. |
| `/orca --voice off` | Disable voice I/O (unregister the orca pane; daemon keeps running so other tools can speak). |

## Backend Detection (first thing every run)

```bash
if cmux current-window >/dev/null 2>&1; then
  ORCA_BACKEND=cmux
elif [[ -n "${TMUX:-}" ]]; then
  ORCA_BACKEND=tmux
else
  echo "🐋 orca needs cmux or tmux. Launch tmux (tmux new -s orca) or open cmux, then re-run."
  exit 1
fi
```

Save the chosen backend to `.orca/state.json` so subsequent ticks don't re-probe. See [references/backends.md](./references/backends.md) for the full primitive table — every other section in this skill assumes you've selected one.

## Playbooks

A playbook is a YAML-frontmatter markdown file telling orca how to drive a particular kind of worker (GSD phases, codex agentic loop, dev server, etc.).

**Lookup precedence** (first match wins on name conflict):
1. `./.orca/playbooks/*.md` — project-local
2. `~/.orca/playbooks/*.md` — user global
3. `~/.claude/skills/orca/playbooks/*.md` — bundled defaults

**Frontmatter shape** (full spec in [references/playbook-format.md](./references/playbook-format.md); worked example in [`playbooks/gsd.md`](./playbooks/gsd.md)):

```yaml
---
name: gsd
description: GSD multi-repo phase orchestration
triggers: ["gsd", "run phase"]
params:
  - name: repo
    required: true
  - name: phase
    required: true
supported_agents: [claude-code]            # which agents can run this playbook as workers
default_agent: claude-code
spawn:
  cwd: "{repo}"
  agents:
    claude-code:
      launcher: cdp                        # auto-perms variant per agent; see "Auto-perms" below
      initial: "/clear && /gsd:plan-phase {phase}"
    codex:
      launcher: codex --dangerously-bypass-approvals-and-sandbox
      initial: "Read .planning/STATE.md and execute phase {phase}'s plan…"
    pi:
      launcher: pi                         # pi has no permission prompts by default
      initial: "Read .planning/STATE.md and execute phase {phase}…"
watch:
  - pattern: "PHASE N COMPLETE ✓"
    action: advance
  - pattern: "Do you want to make this edit?"
    action: send_enter
stop_when:
  - "MILESTONE COMPLETE"
---
```

A playbook describes the **work** in an agent-agnostic way. Each entry under `spawn.agents` is the launcher + initial-prompt mapping for one agent runtime. `supported_agents` declares which are actually viable (e.g., GSD is Claude-Code-only because the `/gsd:*` commands are Claude Code skills). The orchestrator picks an agent per worker — defaulting to `default_agent`, overridable per-spawn.

The markdown body holds free-form notes orca reads as natural-language guidance (quirks, escalation hints, manual recovery steps).

## Agents as orchestrator OR worker

Coding agents (Claude Code, codex, pi) are interchangeable runtimes. Any of them can run **as orchestrator** (executing this skill / its peer configs) or **as worker** (spawned by an orchestrator, executing a playbook). The orca repo therefore ships multiple orchestrator configs so each agent has a way to play that role:

| Agent | Orchestrator config | Lives at |
|-------|--------------------|----------|
| Claude Code | `SKILL.md` (this file) | `~/.claude/skills/orca/SKILL.md` |
| codex | `AGENTS.md` (codex's project-config convention) | repo root, codex picks it up automatically when launched in the dir |
| pi | TBD — pi has TS Extensions / Skills / Prompt Templates; orca-as-pi-skill is a future addition | `pi/` directory in repo (placeholder) |

A worker pane only ever runs an agent in single-agent mode — the orchestration skill is irrelevant there. Workers consume the **playbook**, not orca itself.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webdevike/orca](https://github.com/webdevike/orca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
