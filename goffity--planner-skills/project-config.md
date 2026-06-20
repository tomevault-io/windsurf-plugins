---
trigger: always_on
description: Plans and breaks down work before implementation. Use this skill whenever the user wants to plan a feature, break down a task into subtasks, create a project plan, estimate effort, or organize work into trackable issues — then optionally create GitHub or Jira issues from the plan. MUST trigger when the user describes a feature or task and asks to "break it down", "plan", "figure out tasks", "estimate effort", "where to start", "what steps", "create issues for this work", or wants to think throug
---


# Planner

This skill helps you think through work before jumping into code. It analyzes your codebase, breaks a task into well-scoped subtasks with dependencies, writes a plan file you can track, and optionally creates GitHub or Jira issues from it. The goal is to turn a vague idea like "add authentication" into a concrete, ordered list of things to build.

## Usage

```
/planner [task/feature description]
```

## Instructions

### Step 1: Get Task Description

If no argument provided, ask:
```
What task or feature do you want to plan?
```

If argument provided, use it as the task description.

### Step 1.5: Clarifying Questions

Before jumping into analysis and planning, ask **5-10 clarifying questions** to fully understand the task scope and constraints. This step prevents wasted effort from misunderstanding requirements.

Generate questions based on the task description. Focus on areas that are ambiguous, have multiple valid approaches, or could significantly affect the plan structure. Categories to consider:

**Scope & Boundaries:**
- What's included vs explicitly out of scope?
- Are there specific user roles or personas to consider?
- Should this work with existing data/users or only new ones?

**Technical Constraints:**
- Are there specific technologies, libraries, or patterns to use (or avoid)?
- Are there performance requirements (latency, throughput, data volume)?
- Does this need to be backward-compatible?

**Dependencies & Integration:**
- Does this depend on other teams, services, or external APIs?
- Are there existing systems this needs to integrate with?
- Is there a deadline or release train to target?

**Prioritization:**
- What's the MVP vs nice-to-have?
- Are there phases (launch with X, add Y later)?

Present questions as a numbered list:

```
Before I plan this out, a few questions to make sure I scope it right:

1. [Question about scope/boundaries]
2. [Question about technical approach]
3. [Question about constraints]
4. [Question about integration]
5. [Question about priority/MVP]
...

Answer what you can — skip any that aren't relevant. I'll make reasonable assumptions for anything unanswered.
```

After receiving answers (or if the user skips), incorporate the context into the planning process. Unanswered questions should be noted as assumptions in the plan's Context section.

### Step 2: Choose Language

Ask the user which language to use for the plan output:

```
What language should the plan be written in? (default: English)
```

Use the selected language for all generated content (plan file, issue bodies, summaries). Default to English if no preference given.

### Step 3: Analyze Codebase

Before doing a full analysis, check if a previous plan already analyzed this codebase. This avoids redundant work when the codebase hasn't changed significantly.

```bash
# Check for existing plans and current git state
CURRENT_COMMIT=$(git rev-parse --short HEAD 2>/dev/null)
echo "Current commit: $CURRENT_COMMIT"
ls docs/plans/*.md 2>/dev/null
```

If a previous plan exists, read its "Codebase Analysis" section and compare its commit hash with the current one:

```bash
# Check what changed since the plan was written
PLAN_COMMIT=$(grep -o 'Commit: [^ ]*' docs/plans/*.md 2>/dev/null | tail -1 | sed 's/.*Commit: //')
if [ -n "$PLAN_COMMIT" ]; then
    git diff --stat "$PLAN_COMMIT"..HEAD 2>/dev/null
fi
```

- **If no changes** (or only unrelated files changed): reuse the previous analysis and tell the user. Skip to Step 4.
- **If there are changes**: do a focused analysis on what changed, and merge with the previous analysis.
- **If no previous plan exists**: do a full analysis from scratch.

For a full or focused analysis, use an Explore agent. Focus on:

- Project structure and tech stack
- Architecture patterns and conventions already in use
- Files and modules that will need changes
- Existing test setup and coverage
- Related code that might be affected

Present a brief summary to the user:

```
## Codebase Analysis

**Commit:** [short hash]
**Tech Stack:** [languages, frameworks]
**Architecture:** [patterns found]
**Affected Areas:**
- [file/module] - [why it's relevant]
**Existing Tests:** [test framework, coverage notes]
```

This analysis informs how tasks get scoped — for instance, if the project has no tests yet, a "write tests" task carries more effort than adding tests to an existing suite.

### Step 3.5: Detect Project Structure

Determine whether the project is a **monorepo / multi-repo** or a **single repo**:

```bash
# Detect project structure
# Check for monorepo indicators: multiple go.mod, package.json in subdirs, workspace config, etc.
MONOREPO=false
if ls */go.mod 2>/dev/null | head -1 >/dev/null 2>&1 || \
   grep -q '"workspaces"' package.json 2>/dev/null || \
   [ -f "pnpm-workspace.yaml" ] || \
   [ -f "lerna.json" ] || \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goffity/planner-skills](https://github.com/goffity/planner-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
