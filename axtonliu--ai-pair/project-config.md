---
trigger: always_on
description: |
---


# AI Pair Collaboration

Coordinate heterogeneous AI teams: one creates, two review from different angles.
Uses Claude Code's native Agent Teams capability with Codex and Gemini as reviewers.

## Why Multiple AI Reviewers?

Different AI models have fundamentally different review tendencies. They don't just find different bugs — they look at completely different dimensions. Using reviewers from different model families maximizes coverage.

## Commands

```bash
/ai-pair dev-team [project]       # Start dev team (developer + codex-reviewer + gemini-reviewer)
/ai-pair content-team [topic]     # Start content team (author + codex-reviewer + gemini-reviewer)
/ai-pair team-stop                # Shut down the team, clean up resources
```

Examples:
```bash
/ai-pair dev-team HighlightCut        # Dev team for HighlightCut project
/ai-pair content-team AI-Newsletter   # Content team for writing AI newsletter
/ai-pair team-stop                     # Shut down team
```

## Prerequisites

- **Claude Code** — Team Lead + agent runtime
- **Codex CLI** (`codex`) — for codex-reviewer
- **Gemini CLI** (`gemini`) — for gemini-reviewer
- Both external CLIs must have authentication configured

## Team Architecture

### Dev Team (`/ai-pair dev-team [project]`)

```
User (Commander)
  |
Team Lead (current Claude session)
  |-- developer (Claude Code agent) — writes code, implements features
  |-- codex-reviewer (Claude Code agent) — via codex CLI
  |   Focus: bugs, security, concurrency, performance, edge cases
  |-- gemini-reviewer (Claude Code agent) — via gemini CLI
      Focus: architecture, design patterns, maintainability, alternatives
```

### Content Team (`/ai-pair content-team [topic]`)

```
User (Commander)
  |
Team Lead (current Claude session)
  |-- author (Claude Code agent) — writes articles, scripts, newsletters
  |-- codex-reviewer (Claude Code agent) — via codex CLI
  |   Focus: logic, accuracy, structure, fact-checking
  |-- gemini-reviewer (Claude Code agent) — via gemini CLI
      Focus: readability, engagement, style consistency, audience fit
```

## Workflow (Semi-Automatic)

Team Lead coordinates the following loop:

1. **User assigns task** → Team Lead sends to developer/author
2. **Developer/author completes** → Team Lead shows result to user
3. **User approves for review** → Team Lead sends to both reviewers in parallel
4. **Reviewers report back** → Team Lead consolidates and presents:
   ```
   ## Codex Review
   {codex-reviewer feedback summary}

   ## Gemini Review
   {gemini-reviewer feedback summary}
   ```
5. **User decides** → "Revise" (loop back to step 1) or "Pass" (next task or end)

The user stays in control at every step. No autonomous loops.

## Project Detection

The project/topic is determined by:

1. **Explicitly specified** → use as-is
2. **Current directory is inside a project** → extract project name from path
3. **Ambiguous** → ask user to choose

## Team Lead Execution Steps

### Step 1: Create Team

```
TeamCreate: team_name = "{project}-dev" or "{topic}-content"
```

### Step 2: Create Tasks

Use TaskCreate to set up initial task structure:
1. "Awaiting task assignment" — for developer/author, status: pending
2. "Awaiting review" — for codex-reviewer, status: pending, blockedBy task 1
3. "Awaiting review" — for gemini-reviewer, status: pending, blockedBy task 1

### Step 3: Pre-flight CLI Check

Before launching agents, verify external CLIs are available:

```bash
command -v codex && codex --version || echo "CODEX_MISSING"
command -v gemini && gemini --version || echo "GEMINI_MISSING"
```

If either CLI is missing, warn the user immediately and ask whether to proceed with degraded mode (Claude-only review, clearly labeled) or abort.

### Step 4: Launch Agents

Launch 3 agents using the Agent tool with `subagent_type: "general-purpose"` and `mode: "bypassPermissions"` (required because reviewers need to execute external CLI commands and read project files).

See Agent Prompt Templates below for each agent's startup prompt.

### Step 5: Confirm to User

```
Team ready.

Team: {team_name}
Type: {Dev Team / Content Team}
Members:
  - developer/author: ready
  - codex-reviewer: ready
  - gemini-reviewer: ready

Awaiting your first task.
```

## CLI Invocation Protocol (Shared)

All reviewer agents follow this protocol. Team Lead includes it in each reviewer's prompt.

```
CLI Invocation Protocol:

[Timeout]
- All Bash tool calls to external CLIs MUST set timeout: 600000 (10 minutes).
- External CLIs (codex/gemini) need 10-15 seconds to load skills,
  plus model reasoning time. The default 2-minute timeout is far too short.

[Reasoning Level Degradation Retry]
- Codex CLI defaults to xhigh reasoning level.
- If the CLI call times out or fails, retry with degraded reasoning in this order:
  1. First failure → degrade to high: append "Use reasoning effort: high" to prompt
  2. Second failure → degrade to medium: append "Use reasoning effort: medium"
  3. Third failure → degrade to low: append "Use reasoning effort: low"
  4. Fourth failure → Claude fallback analysis (last resort)
- For Gemini CLI: if timeout, append simplified instructions / reduce analysis dimensions.
- Report the current degradation level to team-lead on each retry.

[File-based Content Passing (no pipes)]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axtonliu/ai-pair](https://github.com/axtonliu/ai-pair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
