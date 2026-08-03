---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`swe-interview-coach` is a **Claude Code plugin** (distributed via the `kirilxd-plugins` marketplace) that turns Claude Code into an interview-prep coach. There is **no application runtime and no build step** — the plugin is markdown (commands, agents, skills, a reference library) plus two small scripts that power a local diagramming canvas. "The code" is mostly prompt-as-program: command files are procedural instructions the main Claude session executes.

Three interview domains are implemented and structured identically:
- **Behavioral** (v0.1.0): STAR story bank → company mapping → mock → drill → debrief.
- **System design** (v0.2.0): canonical walkthrough → practice → graded mock → debrief, on a shared local Excalidraw canvas.
- **Coding** (v0.3.0): DSA pattern walkthrough → practice → graded mock → drill → debrief, with a local Python-stdlib test harness that runs the candidate's code; `/coding-import` pulls public LeetCode problems into the library.

`take-home/` is still planned; the data layout already reserves room for it so adding a domain needs no migration.

## Commands

There is no compiler, linter, or package manager — nothing to `npm install`. The only executable code is the canvas scripts.

```bash
# Develop the plugin locally (loads this dir as a plugin)
claude --plugin-dir ~/Documents/swe-interview-coach
# After editing ANY command/agent/skill .md, reload inside the session:
/reload-plugins

# Run the canvas server test suite (uses an isolated port + temp config dir)
bash tests/canvas-server.test.sh        # prints "pass=N fail=M"; exits non-zero on any failure

# Syntax-check the scripts before committing
node --check scripts/canvas-server.js
bash -n scripts/ensure-canvas.sh tests/canvas-server.test.sh

# Manually boot the canvas (rarely needed; commands do this themselves)
bash scripts/ensure-canvas.sh            # opens browser; last line is READY or ERROR: <details>
bash scripts/ensure-canvas.sh --no-browser
```

`tests/canvas-server.test.sh` is the whole test suite (8 checks against `canvas-server.js`). To run a single check, comment out the others — they share one server process started at line 15. The test isolates state via `SWE_INTERVIEW_COACH_PORT` and `SWE_INTERVIEW_COACH_CONFIG_DIR`, so it never touches real canvas state.

## Two path roots — never confuse them

Every command body distinguishes two locations, and mixing them up is the most common way to break things:

- **`${CLAUDE_PLUGIN_ROOT}`** — the installed plugin dir. Read-only bundled assets live here: `library/`, `scripts/`, `agents/`, `skills/`. Commands read from it; nothing user-specific is ever written here.
- **`$CLAUDE_PROJECT_DIR`** — the user's working directory (typically `~/Documents/interview/`). **All user state is written here** as plain markdown: STAR stories, mapped variants, per-company values/JDs, and timestamped session folders. No database, no JSON config, no hidden home dir.

The one exception to "state lives in the project dir": the live canvas scene at `~/.config/swe-interview-coach/canvas.json` (transient working state; final diagrams are exported into the project dir's session folders).

## How a command runs (the orchestration model)

Files in `commands/*.md` are **not passive prompts** — they are numbered `## Step N` procedures the main session executes top to bottom. A typical command: resolves `$ARGUMENTS` → reads bundled assets from `${CLAUDE_PLUGIN_ROOT}` → runs the interview (via one of the two agent patterns below) → post-processes (scores, writes a session file to `$CLAUDE_PROJECT_DIR`). Read `commands/mock-sysdesign.md` for the most complete example (8 steps: boot canvas → resolve topic → guard → reset → interview → rubric → annotate → write).

Command frontmatter is just `description` + `argument-hint`.

### Two agent invocation patterns (this trips people up)

`agents/*.md` files are used in **two structurally different ways** — check the file's header before assuming:

1. **Spawned subagents** — `behavioral-interviewer`, `behavioral-story-extractor`. They have full frontmatter (`name`, `description`, `tools: Read`, `model: sonnet`) and are launched with the Task tool. They are **Read-only by design**: they conduct the interview and *return a draft/transcript as text*; the main session does every file write. This keeps writes in one place and isolates the persona's context.

2. **Embodied personas** — `agents/sysdesign-interviewer.md`. It has **no frontmatter** and states "Not a spawnable subagent." The command `Read`s the file and the **main session role-plays it inline**. This is necessary because the sysdesign interviewer needs Bash (`date +%s` for live timing) and canvas read/write in the main context — capabilities a Read-only subagent lacks.

### The yield-marker handoff

In-character interview portions end with an **exact marker line** that signals the command's post-processing to take over:
- `[end of session — yielding to debrief]` (sysdesign-interviewer)
- `[end of mock — yielding to main session for debrief]` (behavioral-interviewer)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirilxd/swe-interview-coach](https://github.com/kirilxd/swe-interview-coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
