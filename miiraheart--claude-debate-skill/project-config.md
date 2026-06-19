---
trigger: always_on
description: Multi-agent adversarial debate for product research. 5 agents research independently, debate with elimination rounds, produce structured recommendations.
---


# Multi-Agent Adversarial Debate System

You are orchestrating a 6-phase adversarial debate between 5 specialized research agents. Each agent has a unique persona, full web access, and the ability to search, analyze, and argue for product recommendations.

**Query**: $ARGUMENTS

## Architecture

⚠️ **MANDATORY RULE — SEPARATE TASKS PER AGENT**:
Every time this skill says "spawn agents", you MUST create **one Task() call per agent**.
NEVER write responses for multiple agents inside a single Task. Each agent is an independent
LLM instance with its own persona. One LLM roleplaying 5 agents defeats the entire system.
This applies to ALL phases: research, debate rounds, elimination votes, finals, and jury.

- **Communication**: Pure prompt relay via files — no direct agent-to-agent talk (from Mysti)
- **Topology**: Dense — all agents see all responses each round (from debate-or-vote)
- **State management**: Python scripts in `${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/` handle convergence detection, vote tallying, domain detection, and session state
- **Context passing**: `/tmp/debate-session/` symlink (always points to latest session). Each session gets a timestamped directory under `/tmp/debate-sessions/` (e.g. `debate-20260304-143022/`). Previous sessions are preserved.

### Mandatory Steps Checklist (DO NOT SKIP ANY)

These steps are marked with ⛔ throughout the document. If you skip any, the debate is invalid:

| Step | When | Script/Action |
|------|------|---------------|
| `init` | Before anything | `debate_orchestrator.py init "$ARGUMENTS"` — creates dated session folder |
| URL verification | After Phase 1 | WebFetch each product URL — exclude unverified products |
| `check-duplicates` | After Phase 2 | `debate_orchestrator.py check-duplicates` — verify distinct picks |
| `assess-convergence` | After each debate round | `debate_orchestrator.py assess-convergence {R}` — follow exit code |
| `vote_tallier.py` | After each elimination round | `vote_tallier.py /tmp/debate-session/phase4/` — official tally |

## Scripts Reference

| Script | Purpose | Key Functions |
|--------|---------|---------------|
| `scripts/debate_orchestrator.py` | Session state, domain detection, persona selection, context formatting | `init`, `detect-domain`, `select-personas`, `format-debate-context`, `format-judge-input`, `compile-synthesis` |
| `scripts/convergence_detector.py` | Convergence assessment (from Mysti _assessConvergence) | `assess_convergence()` — keyword agreement + Jaccard stability + Delphi facilitator override |
| `scripts/vote_tallier.py` | Vote collection + 4-step tie-break elimination (from elimination_game) | `run_elimination()` — plurality → confidence → cumulative → random |

## Setup

⛔ **BLOCKING — DO NOT SKIP**: You MUST run init before ANY other action. Do NOT `mkdir` manually.

```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py init "$ARGUMENTS"
```

This creates a timestamped session directory (e.g. `/tmp/debate-sessions/debate-20260304-143022/`) with phase subdirectories and `state.json`. A symlink at `/tmp/debate-session` always points to the latest session. Previous sessions are preserved — never overwritten.

**If you skip this step, sessions will overwrite each other and all previous debate data is lost.**

Then read prompt templates:
```bash
cat ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/prompts/researcher.md
cat ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/prompts/debater.md
cat ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/prompts/judge.md
cat ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/prompts/synthesizer.md
```

---

## Phase 1 — Research Sprint (Parallel)

**Goal**: 5 agents research independently, each producing 3-5 product picks with evidence.

### Step 1: Detect domain and select personas
```bash
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py detect-domain "$ARGUMENTS"
python3 ${DEBATE_SKILL_DIR:-~/.claude/skills/debate}/scripts/debate_orchestrator.py select-personas <detected_domain>
```

### Step 2: Spawn 5 research agents in parallel

⚠️ **CRITICAL**: You MUST make **5 separate Task() calls** in a single message — one per agent.
Each Task is a separate LLM instance. NEVER combine multiple agents into one Task.

```
Task(description="Agent 1 research", subagent_type="general-purpose", model="opus",
     prompt="You are {persona_1}... Write to /tmp/debate-session/phase1/agent-1.md ...")
Task(description="Agent 2 research", subagent_type="general-purpose", model="opus",
     prompt="You are {persona_2}... Write to /tmp/debate-session/phase1/agent-2.md ...")
Task(description="Agent 3 research", ...)
Task(description="Agent 4 research", ...)
Task(description="Agent 5 research", ...)
```

Each agent gets:
- Their unique persona from personas.md injected into the researcher.md template
- Full tool access: WebSearch, WebFetch, perplexity_search_web, scrapling-fetch MCP (s_fetch_page, s_fetch_pattern)
- Instruction: "Find at least 2 independent sources per product. Use scrapling-fetch for pages that block basic fetching."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miiraheart/claude-debate-skill](https://github.com/miiraheart/claude-debate-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
