---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This repository is a **Claude Code plugin** that bundles reusable skills for creating skills and plugins. The plugin is named **skills-toolkit** and provides Agent Skills that can be auto-activated by Claude or invoked directly with `/skills-toolkit:skill-name` slash commands.

**CRITICAL MINDSET:** Skills are instructions FOR CLAUDE, not documentation FOR PEOPLE. When evaluating or improving a skill, the question is always: "Will this help Claude understand and execute the task?" not "Will people find this easy to read?"

**Plugin structure:** This project is organized as a Claude Code plugin with `.claude-plugin/plugin.json` manifest and `skills/` directory for Agent Skills. Skills control invocation behavior through frontmatter (see [Control who invokes a skill](#control-who-invokes-a-skill) below).

## Skill Structure

Each skill is a directory containing:

```
skill-name/
├── SKILL.md                    # Required: metadata + instructions (frontmatter + body)
├── scripts/                    # Optional: executable code (Python, shell, etc.)
│   └── script.py
├── references/                 # Optional: documentation Claude loads into context
│   └── api.md
└── assets/                     # Optional: files used in output (images, templates)
    └── template.docx
```

### SKILL.md Format

SKILL.md is the complete instruction set Claude loads and follows when the skill is triggered.

**Frontmatter** (metadata Claude uses for skill discovery and activation):
```yaml
---
name: skill-name                    # lowercase, hyphens, ≤64 chars
description: >-                     # ≤1024 chars, specific trigger phrases
  What the skill does. Use when [trigger contexts/phrases].
version: 1.0.0                      # Optional: semantic version for tracking
allowed-tools: Read,Write,Bash(*)   # Optional: principle of least privilege
---
```

**Body** (instructions Claude executes):
- Clear, procedural guidance for the task
- Examples Claude can reference and adapt
- Important constraints and edge cases Claude must know
- Links to reference files for deeper context when needed
- Target <500 lines; offload detailed content to references/

**Key principle:** Every word in SKILL.md body is loaded when the skill triggers. Keep it focused on what Claude needs to execute the task correctly.

## Plugin Components

### Agent Skills (in `skills/`)
Skills are discoverable and invocable via both auto-activation and direct `/` commands:

- **skill-composer** - Create NEW Claude Code skills from scratch following best practices. Claude auto-activates when detecting skill creation tasks; users can invoke directly with `/skills-toolkit:skill-composer`.
- **skill-refiner** - Improve and validate EXISTING Claude Code skills for clarity, efficiency, and production readiness. Claude auto-activates when detecting skill refinement/validation tasks; users can invoke directly with `/skills-toolkit:skill-refiner`.
- **skill-tester** - Empirically test and benchmark skills using evaluation-driven development. Two modes: Quick Workflow (fast pass/fail) or Full Pipeline (baseline comparison, metrics, iteration tracking). Users can invoke with `/skills-toolkit:skill-tester`.
- **plugin-creator** - Create, convert, and validate Claude Code plugins. Claude auto-activates when detecting plugin-related tasks; users can invoke directly with `/skills-toolkit:plugin-creator`.
- **subagent-creator** - Create, validate, and refine Claude Code subagents. Claude auto-activates for subagent delegation tasks; users can invoke with `/skills-toolkit:subagent-creator`.
- **hook-creator** - Create, validate, and refine hooks for automating workflows. Claude auto-activates for hook-related work; users can invoke with `/skills-toolkit:hook-creator`.

No separate command files are needed—skills use frontmatter to control invocation behavior.

## Known Limitation: Knowledge Duplication & Future Refactoring

### Current State

This toolkit has intentional **knowledge duplication** that respects Claude's official architecture:

- `plugin-creator` includes summaries of skill/subagent/hook knowledge in `references/`
- These overlap with the full guidance in `skill-composer/`, `subagent-creator/`, and `hook-creator/`
- This duplication follows the **Bounded Scope Principle** (see `skills/skill-composer/references/self-containment-principle.md`)

**Why?** Claude's official architecture does not support skill-to-skill delegation as a first-class feature. Each skill must be completely self-contained within its directory structure. This is documented in [Claude Code Skills documentation](https://code.claude.com/docs/en/skills).

### Future Improvement Path

When Claude implements full support for `context: fork` skill execution ([Feature request on GitHub](https://github.com/anthropics/claude-code/issues/17283)), we can refactor for Single Responsibility Principle:

```yaml
# Future: When context: fork fully works for skills
---
name: plugin-creator
description: Create and organize plugins with proper structure
context: fork  # ← Delegate complex tasks to subagents
agent: general-purpose
---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [full-stack-biz/claude-skills-toolkit](https://github.com/full-stack-biz/claude-skills-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
