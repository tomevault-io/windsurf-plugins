---
trigger: always_on
description: This skill works with:
---

# Create Skill - Comprehensive Skill Creation Guide

## 🎯 PURPOSE: EXTENDING KAI'S CAPABILITIES THROUGH MODULAR SKILLS

**Skills are modular, self-contained packages that extend Claude's capabilities with specialized knowledge, workflows, and tools.**

This guide combines:
- Anthropic's official skill methodology
- PAI-specific patterns and conventions
- Best practices from existing skills
- Template-driven quality standards

## 📚 WHAT ARE SKILLS?

### Definition

Skills are contextual packages that:
1. **Extend capabilities**: Add specialized knowledge or workflows
2. **Load progressively**: Metadata → Instructions → Resources
3. **Activate intelligently**: Match user intent to skill descriptions
4. **Work independently**: Self-contained but inherit global context
5. **Follow standards**: Consistent structure across all skills

### Skills vs Slash Commands

**Skills**:
- Contextual knowledge and workflows
- Always available in system prompt
- Triggered by matching user intent
- Can reference slash commands

**Slash Commands**:
- Executable workflows
- Must be explicitly invoked
- Typically orchestrate multiple tools
- Live in `${PAI_DIR}/commands/`

**Relationship**: Skills often invoke slash commands (e.g., research skill calls `/conduct-research`)

## 🏗️ SKILL ARCHITECTURE

### Three-Layer Loading System

**Layer 1: Metadata** (Always Loaded)
```yaml
---
name: skill-name
description: Clear description with activation triggers
---
```
- Appears in `<available_skills>` in system prompt
- Used for intent matching
- Must be concise but complete

**Layer 2: SKILL.md Body** (Loaded When Activated)
- Quick reference instructions
- Core workflows
- Key commands
- Examples
- References to deeper resources

**Layer 3: Supporting Resources** (Loaded As Needed)
- CLAUDE.md (comprehensive context)
- Subdirectories (components, templates, docs)
- Scripts, references, assets

### Directory Structure Patterns

#### Simple Skill Structure
```
${PAI_DIR}/skills/fabric-patterns/
└── SKILL.md          # Everything in one file
```

**Use when:**
- Single focused capability
- Minimal context needed
- Quick reference suffices

#### Complex Skill Structure
```
${PAI_DIR}/skills/development/
├── SKILL.md                      # Quick reference
├── CLAUDE.md                     # Full methodology
├── primary-stack/                # Reusable components
│   ├── auth-setup.md
│   ├── stripe-billing.md
│   └── business-metrics.md
├── style-guide/                  # UI patterns
│   └── [design resources]
└── [other subdirectories]
```

**Use when:**
- Multi-step workflows
- Extensive methodology
- Multiple sub-components
- Deep context required

## ✍️ WRITING EFFECTIVE SKILLS

### SKILL.md Structure

```markdown
---
name: skill-name
description: What it does, when to use it, key methods. USE WHEN triggers...
---

# Skill Name

## When to Activate This Skill
- Trigger phrase 1
- Trigger phrase 2
- User intent description

## Core Workflow / Main Instructions
[Primary instructions in imperative form]

## Available Tools / Commands
[Key commands, tools, or methods]

## Examples
[Concrete usage examples]

## Supplementary Resources
For full context: `read ${PAI_DIR}/skills/[name]/CLAUDE.md`
For components: `read ${PAI_DIR}/skills/[name]/[subdirectory]/`
```

### Description Writing Guidelines

**Critical elements:**
1. **What it does**: Clear capability statement
2. **Key methods/tools**: Mention specific technologies
3. **Activation triggers**: "USE WHEN user says..." phrases
4. **Unique characteristics**: What makes this skill special

**Examples from PAI:**

**Good - research skill:**
```yaml
description: Multi-source comprehensive research using perplexity-researcher,
  claude-researcher, and gemini-researcher agents. Launches up to 10 parallel
  research agents for fast results. USE WHEN user says 'do research', 'research X',
  'find information about', 'investigate', 'analyze trends', 'current events',
  or any research-related request.
```
✅ Clear what it does (multi-source research)
✅ Mentions tools (3 researcher types)
✅ Lists explicit triggers
✅ Explains benefit (parallel, fast)

**Good - chrome-devtools skill:**
```yaml
description: Chrome DevTools MCP for web application debugging, visual testing,
  and browser automation. The ONLY acceptable way to debug web apps - NEVER use
  curl, fetch, or wget. Provides screenshots, console inspection, network monitoring,
  and DOM analysis.
```
✅ States purpose (debugging, testing)
✅ Strong negative trigger (never use curl)
✅ Lists capabilities
✅ Clear domain (web applications)

**Bad example:**
```yaml
description: A skill for development tasks
```
❌ Too vague
❌ No triggers
❌ No tools mentioned
❌ Unclear when to use

### Instruction Writing Standards

**Use imperative/infinitive form** (verb-first instructions):
- ✅ "Create directory structure"
- ✅ "Launch research agents in parallel"
- ✅ "Use Chrome DevTools for debugging"
- ❌ "You should create a directory"
- ❌ "We will launch research agents"

**Be specific and actionable:**
- ✅ "Run `bun dev` to start server"
- ✅ "Execute `/conduct-research` slash command"
- ❌ "Start the application"
- ❌ "Do research"

**Reference, don't duplicate:**
- ✅ "Use contacts from global context"
- ✅ "Follow global security rules"
- ✅ "See CLAUDE.md for full methodology"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielmiessler/PAIPlugin](https://github.com/danielmiessler/PAIPlugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
