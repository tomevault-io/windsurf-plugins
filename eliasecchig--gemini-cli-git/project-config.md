---
trigger: always_on
description: You are an autonomous agent that handles requests across multiple skills.
---

# Autonomous Agent - Universal Entrypoint

You are an autonomous agent that handles requests across multiple skills.

## Your Job

1. **Understand the prompt provided** (demand file OR user request with context)
2. **Identify required skill(s)** from the prompt
3. **Load skill knowledge** from memory/skills/
4. **Check memory for relevant context** (learnings, conversations)
5. **Execute the tasks**
6. **Record new learnings or conversation context** (if applicable)

## Understanding Skills vs Demands

**Skills** and **Demands** have different responsibilities:

### Skills (Reusable Knowledge)
**Located in**: `memory/skills/{skill-name}/knowledge/GUIDELINES.md`

**Purpose**: Define **HOW** to perform a type of work (the expertise, methodology, quality standards)

**Contains**:
- Domain expertise and best practices
- Quality standards and style guidelines
- General methodology (applies across multiple use cases)
- Output format templates
- When/how to perform the skill

**Examples**:
- `weather_analysis/knowledge/GUIDELINES.md` - HOW to analyze weather data (data sources, analysis methods, report formats)
- `content_creation/knowledge/GUIDELINES.md` - HOW to create content (tone, structure, quality standards)
- `code_review/knowledge/GUIDELINES.md` - HOW to review code (security checks, quality criteria)

**Key principle**: Skills are **generic and reusable**. They don't specify WHAT to analyze or WHEN to run.

### Demands (Specific Tasks)
**Located in**: `memory/demands/{demand-name}.md`

**Purpose**: Define **WHAT** specific task to execute

**Contains**:
- Which skill(s) to use (via YAML frontmatter)
- Specific target/subject to analyze (e.g., "Local weather for beginners", "Project XYZ repository")
- Where to save outputs
- When to skip (conditions specific to this task)
- Task-specific parameters

**Note**: **WHEN** to execute is defined by the workflow schedule (`.github/workflows/agent-scheduler.yml`), NOT in the demand file.

**Examples**:
- `daily-weather-report.md` - WHAT: "Generate weather report for specific location for target audience" (uses weather_analysis skill)
- `weekly-content.md` - WHAT: "Create content about specific project" (uses content_creation skill)
- `pr-review-automation.md` - WHAT: "Review all open PRs in this repository" (uses code_review skill)

**Key principle**: Demands are **specific and targeted**. They specify the exact subject/target and use skill knowledge to execute.

### When Creating New Skills/Demands

**Create a SKILL when**:
- Defining a new type of expertise/capability
- Writing methodology that applies to multiple use cases
- Establishing quality standards for a domain

**Create a DEMAND when**:
- Defining a specific task to be executed (the WHAT)
- Targeting a specific subject/resource
- Combining existing skills for a particular workflow

**Note**: The schedule (WHEN) is defined separately in `.github/workflows/agent-scheduler.yml`

**Example**:
- ✅ SKILL: "How to analyze weather conditions for outdoor activities" (generic, reusable)
- ✅ DEMAND: "Generate daily weather report for specific location and audience" (specific, uses the skill)
- ❌ Don't mix: Skill that only works for one location, or Demand that contains weather analysis methodology

## How You Receive Context

**GEMINI.md (this file)** is auto-loaded by Gemini CLI and provides universal context for all executions.

**The prompt** you receive varies by workflow:

### For Scheduled Runs (`agent-scheduler.yml`)
The prompt IS the demand file (e.g., `memory/demands/example-scheduled.md`).

**What you'll see:**
- YAML frontmatter specifying skill(s):
  ```yaml
  ---
  skill: example_skill
  ---
  ```
  or for multiple skills:
  ```yaml
  ---
  skills:
    - skill_one
    - skill_two
  ---
  ```
- Full task instructions (WHAT to do, WHERE to output, conditions for WHEN to skip)
- Reference to skill's GUIDELINES.md for HOW to do it

**Note**: The schedule (WHEN to run) is defined in the workflow file, not in the demand.

### For @gemini Mentions (`agent.yml`)
The workflow concatenates all context into the prompt file (`tmp/prompt.md`).

**What you'll see in the prompt:**

**For Issue Requests:**
- `# User Request` - what they asked for
- `# Context Type` - `issue`
- `# Original Issue Body` - (if issue comment)
- `# Issue Comment History` - (if issue comment)

**For PR Comments:**
- `# User Request` - the @gemini comment
- `# Context Type` - `pr_comment`
- `# PR Changed Files` - list of files in the PR
- `# PR Conversation History` - all previous comments
- `# PR Diff` - the actual code changes
- `# Inline Comment Details` - (if inline comment)

**All context is already in your prompt** - no need to read tmp files or run cat commands on context files.

## Step 1: Identify Required Skills

Based on the prompt you received:

**For Scheduled Runs:**
- Extract skill(s) from YAML frontmatter in the demand file

**For Issue/PR Requests:**
- Parse the user request to determine which skill(s) are needed:
  - "Create content about X" → content_creation skill
  - "Review this code" → code_review skill
  - "Create content AND review code" → content_creation + code_review skills (multiple!)
- For PRs: Determine which skill owns the files being discussed (usually one skill per PR)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliasecchig/gemini-cli-git](https://github.com/eliasecchig/gemini-cli-git) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
