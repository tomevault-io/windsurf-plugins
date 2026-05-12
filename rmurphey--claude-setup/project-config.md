---
trigger: always_on
description: **Commands** are for routine, predictable tasks you do the same way every time. **Agents** are for complex analysis and intelligence-requiring tasks.
---

# Claude Code Agents Guide

## What Are Agents and When Should You Use Them?

### Commands vs Agents: The Key Distinction

**Commands** are for routine, predictable tasks you do the same way every time. **Agents** are for complex analysis and intelligence-requiring tasks.

## Decision Framework

### Use a Command When:
- ✅ The task is routine and predictable
- ✅ You know exactly what steps to take
- ✅ You do it frequently the same way
- ✅ It's a quick, focused action
- ✅ The outcome is standardized

**Examples:**
- "Check my code quality" → `/hygiene` command
- "Commit my changes" → `/commit` command
- "Add a task to my todo list" → `/todo add` command
- "Push my code" → `/push` command

### Use an Agent When:
- 🧠 The task requires analysis across multiple files
- 🧠 You need intelligent decision-making
- 🧠 The outcome depends on what the analysis finds
- 🧠 It's a one-time or infrequent deep-dive
- 🧠 You need pattern recognition and insights

**Examples:**
- "Analyze my development patterns over 6 months" → `session-insights` agent
- "Find optimization opportunities in my commands" → `command-analyzer` agent
- "Audit my documentation for completeness" → `documentation-auditor` agent

## Quick Decision Tree

```
Is this something you do routinely?
├─ Yes → Use a Command
└─ No → Does it require analysis or intelligence?
   ├─ Yes → Use an Agent
   └─ No → Consider creating a new Command
```

## Real-World Scenarios

### Scenario 1: Daily Development
**Situation:** Starting your workday, checking project health
**Solution:** `/hygiene` command (routine, predictable, same every time)
**Why not agent:** No analysis needed, just standard checks

### Scenario 2: Quarterly Review
**Situation:** Understanding your development patterns over 3 months
**Solution:** `session-insights` agent (requires analysis, pattern recognition)
**Why not command:** Each quarter's data is different, requires intelligent analysis

### Scenario 3: Bug Fix
**Situation:** Need to commit a simple bug fix
**Solution:** `/commit fix` command (standard process, quick action)
**Why not agent:** Straightforward task with known steps

### Scenario 4: Repository Optimization
**Situation:** Want to optimize your entire command library for efficiency
**Solution:** `command-analyzer` agent (complex analysis across many files)
**Why not command:** Requires intelligence to identify patterns and opportunities

### Scenario 5: Real Example - The Push Command Problem
**Situation:** We had a 320-line "safe push workflow" command that did quality validation
**Problem:** Users just wanted `git push`, not a QA audit
**Solution:** Simplified `/push` to `git push`, moved git complexities to `/push-detailed`
**Lesson:** Commands should handle routine operations, CI/CD should handle validation

### Scenario 6: Major Command → Agent Conversions
**Problem:** Several commands were doing analysis work instead of routine tasks
**Examples:**
- `/next` (400+ lines) → `next-priorities` agent
- `/estimate` (330+ lines) → `usage-estimator` agent  
- `/retrospective` → Split into session capture (command) + analysis (agent)
**Result:** Massive token reduction while improving intelligence capabilities

## Cost-Benefit Analysis

### Commands
- **Cost:** Low tokens (30-100 per execution)
- **Speed:** Fast execution (seconds)
- **Benefit:** Consistent, reliable results
- **Best for:** Daily workflows, routine tasks

### Agents
- **Cost:** Higher tokens (200-800 per execution)
- **Speed:** Longer execution (minutes)
- **Benefit:** Deep insights, intelligent analysis
- **Best for:** Optimization, analysis, strategic decisions

## Available Agents in This Repository

### `command-analyzer`
**When to use:** You want to optimize your command library
**What it does:** Analyzes usage patterns, finds redundancies, optimizes for token efficiency
**Example situation:** "I have 20+ commands and want to streamline them" or "My commands feel slow and verbose"

### `session-insights`
**When to use:** You want to understand your development patterns
**What it does:** Processes session history to extract patterns, productivity insights, and recommendations
**Example situation:** "I want to see how my productivity has changed over time" or "What are my development patterns?"


### `documentation-auditor`
**When to use:** You want to ensure documentation quality
**What it does:** Audits all documentation for completeness, consistency, and tone
**Example situation:** "I want to make sure my project documentation is professional and welcoming"


### `next-priorities`
**When to use:** You want intelligent analysis of what to work on next
**What it does:** Analyzes project state, git status, tasks, and context to recommend actions
**Example situation:** "I'm not sure what to prioritize next in my development work"

### `usage-estimator`
**When to use:** You want accurate estimates for Claude usage on tasks
**What it does:** Analyzes your project and patterns to provide personalized estimates
**Example situation:** "How much will it cost to add authentication to my app?"


## Agent Structure

Each agent file contains frontmatter metadata followed by detailed instructions:

```yaml
---

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rmurphey/claude-setup](https://github.com/rmurphey/claude-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
