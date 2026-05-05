---
trigger: always_on
description: This is the **AI Dev Standards** repository - a complete system for maintaining AI development standards with automatic installation and syncing.
---

# AI Assistant Rules for ai-dev-standards

## Project Overview

This is the **AI Dev Standards** repository - a complete system for maintaining AI development standards with automatic installation and syncing.

**Key Components:**
- **skills/** - 64 specialized skills (mvp-builder, rag-implementer, data-visualizer, etc.)
- **mcp-servers/** - 50 MCP server implementations (executable tools)
- **tools/** - 9 LangChain/CrewAI tools + 4 supporting scripts
- **components/** - 13 reusable React components and patterns
- **integrations/** - 6 third-party service integrations (OpenAI, Supabase, etc.)
- **templates/** - Config file templates (.cursorrules, .gitignore, etc.)
- **CLI/** - Command-line tool for automatic installation and syncing
- **installers/bootstrap/** - Auto-bootstrap system (`npx @ai-dev-standards/bootstrap`)
- **docs/** - Complete documentation (2500+ lines)

**Current Status:** ✅ Production Ready (v1.3.0)
**Coverage:** 92% skill-to-MCP parity (1.1:1 ratio) • 107 total resources

---

## Critical: Load Context First

Before performing ANY task in this repository or in projects referencing it:

1. ✅ Check if `ai-dev-standards/` is accessible
2. ✅ Read `meta/PROJECT-CONTEXT.md` (understand the system)
3. ✅ Read `meta/HOW-TO-USE.md` (learn navigation)
4. ✅ Review `meta/DECISION-FRAMEWORK.md` (when making technology choices)
5. ✅ Review `docs/SYSTEM-OVERVIEW.md` (understand the complete architecture)

**Time investment:** 3-5 minutes
**Value:** Prevents mistakes, enables effective use, ensures consistency

---

## Core Principles

### 1. Check Before Creating

**ALWAYS** search for existing solutions before building new:
- Search `meta/skill-registry.json` for relevant skills
- Check `standards/architecture-patterns/` for design approaches
- Review `playbooks/` for operational procedures
- Look in `components/` for reusable implementations
- Consult `examples/` for reference code

**Never invent when a standard exists.**

### 2. Follow Standards Always

**ALL code must follow:**
- Conventions in `standards/coding-conventions/` (when created)
- Best practices in `standards/best-practices/`
- Patterns in `standards/architecture-patterns/`
- Security guidelines (never compromise)

**No exceptions without documented reasoning.**

### 3. Use Skills for Guidance

For specialized tasks:
- Skills activate automatically based on context
- Can explicitly request: "Use the [skill-name] skill to..."
- Skills provide methodology, not rigid rules
- Adapt guidance to specific context

**Let skills augment capabilities.**

### 4. Apply Decision Framework

For ANY architectural or technical decision:
- Consult `meta/DECISION-FRAMEWORK.md`
- List requirements and constraints
- Consider trade-offs
- Document decision with reasoning
- Reference framework sections

**Document decisions, don't just make them.**

### 5. Use Playbooks for Procedures

Check `playbooks/` for step-by-step guides:
- Deployment procedures
- Release management
- Operational tasks
- Common workflows

**Follow playbooks when they exist.**

---

## Task Priority Order

When given any task:

```
1. Load Context
   └─ Read META files (PROJECT-CONTEXT, HOW-TO-USE, DECISION-FRAMEWORK)

2. Check PLAYBOOKS
   └─ Is there an existing procedure for this exact task?

3. Identify SKILLS
   └─ Search meta/skill-registry.json for relevant methodologies

4. Search COMPONENTS
   └─ Can I reuse existing code?

5. Review PATTERNS
   └─ What architectural approach fits?

6. Apply STANDARDS
   └─ What conventions must I follow?

7. Reference EXAMPLES
   └─ Are there similar implementations?

8. Implement
   └─ Build following guidance from above

9. Document
   └─ Record decisions and reference sources
```

---

## File Reference Format

When mentioning files from ai-dev-standards:

**Good examples:**
- "Based on skills/mvp-builder/SKILL.md, the P0/P1/P2 matrix..."
- "Following standards/architecture-patterns/rag-pattern.md:45-67..."
- "As outlined in meta/DECISION-FRAMEWORK.md:123, RAG is appropriate when..."

**Include:**
- Relative path from repository root
- Specific line numbers when referencing details
- Clear connection to current task

---

## Communication Style

### When Starting a Task

**Say:**
```
"I'll [task description]. Let me check our standards repository first...

✓ Loaded meta/PROJECT-CONTEXT.md
✓ Checked playbooks/ for existing procedures
✓ Identified relevant skill: [skill-name]

[Proceed with explanation of approach based on guidance]"
```

### When Making Decisions

**Say:**
```
"Based on meta/DECISION-FRAMEWORK.md ([section-name]), I recommend [choice] because:

Requirements:
- [requirement 1]
- [requirement 2]

Decision: [Choice]

Reasoning:
- [reason 1 citing framework]
- [reason 2 citing framework]

Trade-offs:
- [trade-off 1]
- [trade-off 2]"
```

### When Using Skills

**Say:**
```
"I'm using the [skill-name] skill (skills/[skill-name]/SKILL.md) which provides [key methodology].

[Apply skill guidance to specific task]"
```

### When Creating Something New

**Say:**
```
"I didn't find an existing [component/pattern] for this in:
- components/ (checked)
- standards/architecture-patterns/ (checked)
- playbooks/ (checked)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daffy0208/ai-dev-standards](https://github.com/daffy0208/ai-dev-standards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
