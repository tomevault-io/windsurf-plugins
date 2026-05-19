---
trigger: always_on
description: PR, pull request, commit, push, git push, gh pr create, open PR, create PR, reviewer, Asana task, merge, GitHub
---


# Pull Request Guidelines & Workflow

## 🚨 CRITICAL: Required Information and Approval Before Creating PR

**MANDATORY**: Before creating any PR, you MUST:

### Step 1: Gather Required Information

### 1. Task/Issue URL
**Ask**: "What is the Asana task URL for this PR?"
- **NEVER proceed** with placeholder text like `[TASK_ID]` or `[INSERT_URL]`
- **NEVER assume** you can skip this step
- **ONLY proceed** if user explicitly says to omit it or provides the URL

### 2. PR Reviewer Assignment (CRITICAL for Asana Integration)
**Ask**: "Who should review this PR?"
- Ask if they want to:
  - Assign a specific reviewer (get their GitHub username for `--reviewer` flag)
  - Use auto-assignment (`--reviewer Apple-dev` team)
  - Handle it themselves after PR creation
- **NEVER proceed** without understanding the reviewer assignment strategy
- **ONLY proceed** if user explicitly provides the information or strategy

**WHY THIS MATTERS**: 
- GitHub Action only creates Asana subtask when reviewer is **assigned via GitHub's reviewer mechanism**
- Using `--reviewer` flag triggers the `review_requested` event that runs the Asana integration
- Without reviewer assignment, no Asana subtask is created automatically

### 3. Tech Design URL (For Significant Changes)
- **Default to "N/A"** for minor changes and bug fixes
- **ASK for significant changes** (new features, architectural changes)
- **Can be omitted** if user doesn't explicitly provide one - use "N/A"
- Unlike Task/Issue URL, this is **optional** and can default to "N/A"

### 4. Exception: User Explicitly Opts Out
The **ONLY** acceptable reason to skip asking for Task URL and Reviewer is if the user explicitly states:
- "Skip Asana task" or "No Asana task"
- "I'll assign reviewer myself" or "Use auto-assignment"

**Failure to ask for Task URL and Reviewer = violation of PR workflow.**

---

### Step 2: Get User Approval Before Creating PR

**MANDATORY**: After gathering all information, you MUST:

1. **Present the complete PR body text** to the user for review and approval
2. **Include the reviewer name** that will be assigned
3. **Show the exact text** that will be used in the PR body (not the command)
4. **Wait for explicit approval** before proceeding
5. **ONLY after approval**: Execute the `gh pr create` command

**Do NOT create the PR without showing the user the exact PR body text first.**

**Format for approval request:**
```
Here's the PR I'm about to create:
**Title:** [PR title]

**Reviewer:** @username

**PR Body:**
[Show complete PR body text here]

Proceed with creating the PR?
```

After user approves, then execute the `gh pr create` command.

## 🚨 CRITICAL: Always Open PR URL After Creation

**MANDATORY**: After creating or updating a PR, **IMMEDIATELY** run:
```bash
open <PR_URL>
```

This ensures the PR is accessible and properly formatted in the browser.

## Objective

- **Maintain a clear and maintainable list** of open PRs in the Apple repositories
- **Improve PR review turnaround time** through proper assignment and notification processes
- **Establish clear rules** for internal (Apple team) and external (FrontEnd, etc.) contributions
- **Remove PR assignment** as part of the Apple Weekly process

## PR Types and Assignment Strategy

We have **two different types** of code contributions:

### **Projects**
Large features or significant changes with designated technical reviewers.

### **Tasks** 
Small improvements or bug fixes that require flexible reviewer assignment.

**Key Principle**: A PR **assignee** is the PR author, a PR **reviewer** is whoever will review it.

## Assignment Workflows

### Projects Workflow

For significant features and planned work:

1. **Use Technical Reviewer**: The technical reviewer should be the default person to assign the PR review
2. **No MM Posting**: There's no need to post the PR link on MM (Mattermost)
3. **Review Assignment Process**:
   - Create PR with: `gh pr create --reviewer TECHNICAL_REVIEWER_USERNAME`
   - This automatically creates Asana subtask and assigns it to the reviewer
   - No need to manually ping on Asana (automation handles it)
4. **Shared Responsibility**: Both the technical reviewer and developer are responsible for staying in sync
5. **Fallback**: If the technical reviewer can't review the PR, request different reviewer in GitHub UI (triggers new Asana assignment)

### Tasks Workflow

For bug fixes and small improvements:

1. **Pre-Agreement**: Think about who's the best person to review this task and **agree with them to be the reviewer even before posting the PR** (similar to choosing technical reviewer for projects)

2. **When Uncertain**: If you don't know who would be the best person, or the problem is generic and doesn't require domain knowledge, use **GitHub auto assignment** with `--reviewer Apple-dev`

3. **Assignment Process**:
   - Create PR with: `gh pr create --reviewer USERNAME` (or `--reviewer Apple-dev` for auto)
   - Asana subtask is automatically created and assigned
   - No need to manually ping on Asana (automation notifies them)
   - If reviewer is AFK, request different reviewer in GitHub UI (triggers new assignment)

4. **Availability Management**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
