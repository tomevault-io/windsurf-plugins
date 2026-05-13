---
trigger: always_on
description: ⚠️ **SETUP REQUIRED** - Run `/setup-orchestrator` to configure this for your organization.
---

# Orchestrator - Claude Code Context

⚠️ **SETUP REQUIRED** - Run `/setup-orchestrator` to configure this for your organization.

**Last Updated: [DATE - will be updated by setup wizard]**

---

## Repository Overview

**What is this repo?** The Orchestrator is a **central infrastructure repository** that provides shared Claude Code resources (agents, skills, hooks, commands) to all projects via symlinks.

**Purpose:** Single source of truth for:
- Generic AI agents (code review, planning, documentation, refactoring)
- Auto-triggering skills (organization-specific development patterns)
- Hooks (skill activation, file tracking)
- Slash commands (dev docs creation)
- Shared guidelines (architecture, error handling, testing)

**Key Difference from Application Repos:**
- **orchestrator** (this repo): Shared infrastructure provider
- **your repositories**: **[Will be listed here after setup - e.g., app repo, core repo, etc.]**

**Critical:** Changes here affect ALL repos via symlinks. Test in application repos before committing.

---

## Resource Discovery Map

This section lists **everything** available in the orchestrator. Use this as your reference for what exists and when to use it.

### Skills (Auto-Trigger)

Skills activate automatically when you edit files matching specific patterns or keywords.

**[REPOSITORY-SPECIFIC SKILLS WILL BE GENERATED DURING SETUP]**

After running `/setup-orchestrator`, you will have one skill per repository, like:

#### **[repo-name]**-guidelines (Example)
**When it triggers:** Editing files in your **[repo-name]** repo
- File patterns: **[e.g., \*\*/repo-name/\*\*/\*.tsx, \*\*/repo-name/\*\*/\*.py]**
- Keywords: **[e.g., React, FastAPI, etc. - detected from your tech stack]**

**What it provides:**
- **[Tech stack-specific development patterns]**
- **[Error handling guidance for your frameworks]**
- **[Testing patterns for your tools]**

**References:** guidelines/error-handling.md, guidelines/testing-standards.md

**Status:** ⏳ Will be created during setup

---

#### skill-developer
**When it triggers:** Working on skill system
- File patterns: `**/.claude/skills/**`
- Keywords: "skill", "skill-rules.json"

**What it provides:**
- Meta-skill for creating new skills
- Skill structure and patterns
- skill-rules.json configuration
- Testing skill triggers

**Status:** ✅ Active

**Location:** `shared/skills/skill-developer/`

### Agents (Invoke via Task Tool)

Agents must be invoked explicitly using the Task tool. They don't auto-trigger.

**How to invoke an agent:**
```
Task tool with:
- subagent_type: "agent-name"
- prompt: "What you want the agent to do"
```

#### code-architecture-reviewer
**When to use:** Need code review for architectural issues, best practices, consistency

**How to invoke:**
- Task tool with subagent_type: "code-architecture-reviewer"
- Prompt: "Review the authentication module for architectural issues"

**What it does:**
- Reviews code for best practices
- Checks architectural consistency with **[your organization's]** patterns
- Identifies potential technical debt
- References: guidelines/architectural-principles.md

**Tech stack:** **[Will be customized based on your detected tech stacks]**

**Location:** `shared/agents/global/code-architecture-reviewer.md`

#### refactor-planner
**When to use:** Planning a refactoring project

**How to invoke:**
- Task tool with subagent_type: "refactor-planner"
- Prompt: "Plan refactoring of the authentication system"

**What it does:**
- Creates comprehensive refactoring plan
- Identifies files affected
- Suggests implementation phases
- Assesses risks

**Location:** `shared/agents/global/refactor-planner.md`

#### code-refactor-master
**When to use:** Executing a refactoring (after planning)

**How to invoke:**
- Task tool with subagent_type: "code-refactor-master"
- Prompt: "Execute refactoring of authentication module per plan"

**What it does:**
- Executes refactoring systematically
- Tracks file changes
- Ensures no broken imports
- Maintains backward compatibility

**Location:** `shared/agents/global/code-refactor-master.md`

#### plan-reviewer
**When to use:** Review an implementation plan before execution

**How to invoke:**
- Task tool with subagent_type: "plan-reviewer"
- Prompt: "Review the plan in dev/active/feature-name/"

**What it does:**
- Reviews plans for completeness
- Identifies missing considerations
- Suggests improvements
- Validates timelines

**Location:** `shared/agents/global/plan-reviewer.md`

#### documentation-architect
**When to use:** Create or update documentation

**How to invoke:**
- Task tool with subagent_type: "documentation-architect"
- Prompt: "Create documentation for **[your feature/module]**"

**What it does:**
- Creates concise, actionable documentation
- Follows documentation standards
- References: guidelines/documentation-standards.md
- Maintains consistent structure

**Location:** `shared/agents/global/documentation-architect.md`

#### auto-error-resolver
**When to use:** Fix errors or resolve build issues

**How to invoke:**
- Task tool with subagent_type: "auto-error-resolver"
- Prompt: "Fix errors in the codebase"

**What it does:**
- Analyzes errors
- Suggests and applies fixes
- Checks cached error logs
- Validates fixes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BoardKit/orchestrator](https://github.com/BoardKit/orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
