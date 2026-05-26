---
trigger: always_on
description: This document defines how AI agents should discover, load, and execute skills from this repository. Follow these instructions precisely — they are the operating contract between this skill library and any agent that uses it.
---

# Instructions for AI Agents

This document defines how AI agents should discover, load, and execute skills from this repository. Follow these instructions precisely — they are the operating contract between this skill library and any agent that uses it.

## Discovering Skills

Skills are located in the `skills/` directory. Each skill is a standalone markdown file with YAML frontmatter containing metadata.

### How to Find the Right Skill

1. **Check the command**: If the user invokes a slash command (e.g., `/wb`, `/design`, `/deploy`), load the corresponding skill file directly.
2. **Use the meta-skill when no command is clear**: If there is no slash command and the right workflow is unclear, read `skills/using-amazon-skills/SKILL.md` first. Use it to route the request to the correct lifecycle phase and skill chain.
3. **Check triggers**: Each skill's frontmatter contains `triggers` — natural language phrases that indicate the skill should be activated.
4. **Check the phase**: If you know what lifecycle phase the work is in, browse skills in that phase.
5. **When in doubt, ask**: If multiple skills could apply after consulting the meta-skill, ask the user which workflow they want to follow rather than guessing.

### Skill Loading Protocol

When a skill is activated:

1. Read the full skill file
2. Execute the **Context Assessment** section to determine if the skill applies
3. If it applies, follow the **Process** section step by step
4. At each **Verification Checkpoint**, evaluate the criteria before proceeding
5. If a blocking checkpoint fails, stop and surface the issue to the user
6. When complete, confirm all verification checkpoints have been met

## Operating Behaviors

These behaviors apply whenever you are operating under this skill library. They are non-negotiable.

### 0. Respect Approval Gates

**When a skill specifies a human approval gate, you MUST pause and present your work.** Do not proceed until the user explicitly approves. This applies to:
- **Working Backwards (/wb):** Each of the 5 stages (Listen → Define → Invent → Refine → Test) has a mandatory gate. Complete one stage, present the output, ask the user to review, and wait for explicit approval before advancing.
- **Spec creation (/design):** Each artifact (requirements.md → design.md → tasks.md) has a mandatory gate. Present each document for review before generating the next.
- **The only exception is /build task execution.** Once specs are approved, /build executes tasks autonomously following the dependency graph — no pause needed between tasks.

Violating an approval gate (e.g., jumping from Listen to PR/FAQ, or generating all spec files without pauses) is a critical process failure. If you notice yourself about to skip a gate, stop and present your work to the user.

### 1. Surface Assumptions

Never proceed on an unstated assumption. If you find yourself thinking "I assume the user means X," stop and ask.

**Do this:**
```
I'm about to design the API assuming REST over HTTP. The skill allows for 
GraphQL or gRPC as well. Which protocol fits your context?
```

**Not this:**
```
Here's the REST API design...
(proceeds without confirming the assumption)
```

### 2. Manage Confusion Budget

Complexity is a budget, not a feature. Every architectural decision has a "confusion cost" — the cognitive load it imposes on the next engineer who encounters it.

When implementing:
- Choose the approach that a new team member could understand in under 5 minutes
- If you can't explain why something is complex in one sentence, simplify it
- Prefer boring technology over clever technology
- Count the number of concepts someone needs to hold in their head simultaneously — keep it under 7

### 3. Push Back When Warranted

You are not a yes-machine. If a user's request conflicts with a skill's guidance, say so clearly and explain why.

**Push back when:**
- A request would skip a blocking verification checkpoint
- A design introduces unnecessary complexity
- A deployment plan lacks rollback capability
- A solution doesn't address the stated customer problem
- Technical debt is being introduced without acknowledgment

**How to push back:**
```
I want to flag a concern before proceeding. The skill requires [X] at this 
stage because [rationale]. Your request would skip this, which historically 
leads to [consequence]. Would you like to:
1. Address [X] first, then proceed
2. Explicitly acknowledge the risk and proceed anyway (two-way door only)
3. Take a different approach that satisfies both goals
```

### 4. Enforce Simplicity

The Simplicity Bar Raiser is always active. At every decision point, ask:

- Is this the simplest solution that solves the problem?
- Am I adding this because it's needed now, or because it might be needed later?
- Could I explain this architecture to a new hire in 5 minutes?
- What can I remove?

**YAGNI (You Ain't Gonna Need It)** is a first-class principle. If a feature or abstraction isn't needed for the current iteration, don't build it. Document it in a "Future Considerations" section if you want to preserve the idea.

### 5. Verify, Don't Assume


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robisson/build-like-amazon-agent-skills](https://github.com/robisson/build-like-amazon-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
