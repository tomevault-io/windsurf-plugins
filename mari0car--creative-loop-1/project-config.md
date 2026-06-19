---
trigger: always_on
description: Run a structured creative thinking loop on any problem. Generates diverse ideas via parallel personas, evaluates them, refines top candidates, and runs experiments to validate. Use when facing open-ended design decisions, architecture choices, or any problem that would benefit from genuine divergent thinking before converging on a solution.
---


When the user invokes `/creative`, you enter a structured creative thinking mode. This skill transforms any problem into a multi-perspective exploration. Follow these instructions precisely.

---

## COMMANDS

### `/creative init`
Initialize creative loop in the current project. Create `.creative-loop/` directory structure with all required files. Copy default personas and prompts. Create `config.json`. Tell the user it's ready.

### `/creative <problem>`
Run the full creative loop on the problem. Follow the FULL LOOP PROTOCOL below.

### `/creative quick <problem>`
Run a fast version: 3 personas, no experiments, 1 iteration. Follow QUICK MODE PROTOCOL.

### `/creative evaluate <idea>`
Skip generation. Evaluate a user-provided idea using the EVALUATION PHASE only.

### `/creative experiment <idea>`
Skip generation and evaluation. Run the EXPERIMENT PHASE on the idea.

### `/creative optimize`
Run SELF-OPTIMIZATION PROTOCOL to improve the loop's own prompts.

### `/creative sync`
Sync self-optimized improvements back to the canonical repo so they can be shared with others. Handles all three cases: local changes on this machine, improvements from another project on the same machine, and improvements from a different machine entirely. Follow the SYNC PROTOCOL.

### `/creative status`
Show the most recent session summary from `.creative-loop/sessions/`.

### `/creative help`
List these commands with brief descriptions.

---

## FULL LOOP PROTOCOL

Work through these phases in order. Be transparent with the user about what phase you're in.

---

### PHASE 0: CHECK SETUP

1. Check if `.creative-loop/` exists in the current directory.
2. If not: tell the user and offer to run `/creative init` first. Stop here.
3. Read `.creative-loop/config.json`. Use its settings throughout.
4. Create a session directory: `.creative-loop/sessions/{YYYYMMDD_HHMMSS}/`

---

### PHASE 1: CREATIVE BRIEF

Generate a structured creative brief from the user's problem statement.

**Read the problem** alongside:
- CLAUDE.md (project context, constraints, conventions)
- Recent git log summary if available: `git log --oneline -10`
- Any relevant file the user mentioned

**Write** `.creative-loop/sessions/{timestamp}/brief.json`:
```json
{
  "problem_statement": "Clear one-sentence description",
  "constraints": ["constraint1", "constraint2"],
  "success_criteria": ["criterion1", "criterion2"],
  "domain": "e.g. backend-performance, api-design, ux, architecture",
  "tags": ["relevant", "tags"],
  "exploration_budget": "low|medium|high",
  "context_summary": "2-3 sentences about relevant project context",
  "known_attempts": ["anything the user already tried"]
}
```

**Present the brief to the user** and ask: "Does this capture it, or should I adjust anything before we explore?"

Wait for confirmation. Update brief.json if needed.

---

### PHASE 2: DIVERGE — GENERATE IDEAS

Select personas and launch generators in parallel.

**Load persona files** from `.creative-loop/personas/` — first check `custom/` then `builtin/`.

**Select 5 personas** using this logic:
1. Pick personas matching the domain (check persona `domains` field)
2. Always include at least 1 "wild card" persona (Provocateur or Constraint Inverter)
3. Check `.creative-loop/patterns/persona_effectiveness.json` — deprioritize personas with hit_rate < 0.2 for this domain
4. Fill remaining slots with highest-effectiveness personas

**Load creative memory** from `.creative-loop/patterns/successful.json` and `failed.json`. Select entries matching the brief's domain and tags.

**Tell the user**: "Launching 5 generator agents in parallel. Each explores the problem from a different angle..."

**Launch 5 sub-agents in parallel** (one per persona). Each sub-agent receives this prompt — fill in the placeholders:

```
## Your Creative Persona: {PERSONA_NAME}

### Your Thinking Frame
{PERSONA_THINKING_FRAME}

### The Problem to Explore
{PROBLEM_STATEMENT}

### Constraints
{CONSTRAINTS}

### Project Context
{CONTEXT_SUMMARY}

### Successful Patterns (from past sessions — consider these as starting points)
{RELEVANT_SUCCESSFUL_PATTERNS or "None yet"}

### Approaches to Avoid (known failures)
{RELEVANT_FAILED_APPROACHES or "None yet"}

### Your Task
Generate 3 candidate ideas using your specific thinking frame. Do NOT produce safe, obvious ideas — your persona demands a specific unconventional angle.

For each idea, produce:
- **title**: Short descriptive name
- **core_idea**: What this approach is (2-3 sentences)
- **mechanism**: How it would work technically (3-5 sentences)
- **novelty_claim**: What assumption this challenges or what makes it non-obvious
- **risks**: 1-3 specific risks
- **confidence**: 0.0-1.0 how strongly you believe this is worth exploring
- **thinking_process**: 1-2 sentences on how your persona shaped this idea

Think boldly. The evaluation phase will filter — your job is to expand the possibility space.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mari0car/creative-loop-1](https://github.com/mari0car/creative-loop-1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
