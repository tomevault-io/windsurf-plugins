---
trigger: always_on
description: This repository provides AgentSkills for following a Problem-Based Software Requirements Specification (SRS) methodology. The focus is on enabling AI-assisted requirements engineering through structured, problem-first approaches.
---

# GitHub Copilot Instructions for Problem-Based SRS

## Project Overview
This repository provides AgentSkills for following a Problem-Based Software Requirements Specification (SRS) methodology. The focus is on enabling AI-assisted requirements engineering through structured, problem-first approaches.

The repository follows the **[AgentSkills](https://agentskills.io)** standard and the **Claude Code Plugins** layout. Compatibility priority is **GitHub Copilot first**, then **Claude Code/Claude.ai**.

### Plugin Standards

**This repository follows the Claude Code plugins specification:**
- **Plugins Guide**: https://code.claude.com/docs/en/plugins.md
- **Plugins Reference**: https://code.claude.com/docs/en/plugins-reference.md

When modifying this repository structure, ensure compliance with these plugin standards.

### Compatibility Priority (GHCP → Claude)

1. **GitHub Copilot first**: Keep skills and instructions directly usable in Copilot workflows.
2. **Claude second**: Keep `.claude-plugin/plugin.json`, `skills/`, `agents/`, `hooks/`, and `settings.json` aligned with Claude plugin docs.
3. **Consistency over time**: Keep compatibility guidance consistent when it changes.

## Core Principles
1. **Problem-First Thinking**: Always identify the problem before proposing solutions
2. **Lightweight Methodology**: Favor simplicity over complex frameworks
3. **AI-Native Design**: Content designed for consumption by AI agents (following AgentSkills standard)
4. **Practical Guidance**: Focus on actionable skills and templates

---

## 🔄 Problem-Based SRS Iteration Guidelines

### Using the Methodology

When analyzing or working with this repository, **use the Problem-Based SRS methodology** to iterate on problems, needs, and requirements (both functional and non-functional).

**Load and follow the methodology from these skills:**

| Step | Skill |
|------|-------|
| 0. Business Context (CONTEXT) | `skills/business-context/SKILL.md` |
| 1. Customer Problems (WHY) | `skills/customer-problems/SKILL.md` |
| 2. Software Glance | `skills/software-glance/SKILL.md` |
| 3. Customer Needs (WHAT) | `skills/customer-needs/SKILL.md` |
| 4. Software Vision | `skills/software-vision/SKILL.md` |
| 5. Functional Requirements (HOW) | `skills/functional-requirements/SKILL.md` |
| Validation | `skills/zigzag-validator/SKILL.md` |
| Complexity (Optional) | `skills/complexity-analysis/SKILL.md` |
| Orchestrator | `skills/problem-based-srs/SKILL.md` |
| Agent | `agents/problem-based-srs/AGENT.md` |

### Artifact Naming Convention

- **Customer Problems**: `CP.{n}` or `CP.{n}.{m}` (e.g., CP.01, CP.01.1)
- **Customer Needs**: `CN.{cp}.{n}` (e.g., CN.01.1 traces to CP.01)
- **Functional Requirements**: `FR.{cp}.{cn}.{n}` (e.g., FR.01.1.1 traces to CN.01.1)
- **Non-Functional Requirements**: `NFR.{version}.md` (e.g., NFR.1.0.md)

---

## 🚀 Trunk-Based Development Workflow

This repository follows **trunk-based development**. All changes go directly to `main`.

### Git Workflow for Iterations

When creating or updating requirement artifacts:

```bash
git add .                                    # Stage all changes
git commit -m "<type>: <description>"        # Commit with message
git push origin main                         # Push to trunk
```

### Commit Message Convention

| Prefix | Use Case | Example |
|--------|----------|---------|
| `feat:` | New feature or requirement | `feat: Add CP.02 for mobile access` |
| `fix:` | Bug fix or correction | `fix: Correct FR.01.2.1 traceability` |
| `docs:` | Documentation updates | `docs: Update README with new workflow` |
| `refactor:` | Restructuring without changing behavior | `refactor: Reorganize skills` |

**Always confirm with the user before pushing to main.**

---

## Workflow Guidelines

### Before Taking Action
**CRITICAL: Always plan and get confirmation before executing tasks.**

1. **Understand the Request**: Clarify what the user wants to accomplish
2. **Create a Plan**: Present a clear plan of what will be changed/created/deleted
3. **Ask for Confirmation**: Wait for user approval before executing
4. **Execute**: Only after confirmation, proceed with the changes
5. **Verify**: Show what was done and confirm completion

**Example iteration workflow:**
```
User: "Analyze the codebase and identify testing requirements"

AI: "I'll analyze using Problem-Based SRS methodology.
     Loading: skills/problem-based-srs/SKILL.md
     
     [Applies 5-step process from source files]
     
     Plan:
     1. Save analysis to spec/NFR.2.0.md
     2. git add, commit, push to main
     
     Proceed? (yes/no)"
```

---

## When Working on This Repository

### Plugin Structure (Claude Code Standard)

This repository is structured as a Claude Code plugin:

```
Problem-Based-SRS/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest (name, version, metadata)
├── agents/
│   └── problem-based-srs/       # Agent orchestrator
│       └── AGENT.md
├── skills/
│   ├── problem-based-srs/       # Main skill for SRS methodology
│   │   ├── SKILL.md
│   │   └── references/          # Examples only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RafaelGorski/Problem-Based-SRS](https://github.com/RafaelGorski/Problem-Based-SRS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
