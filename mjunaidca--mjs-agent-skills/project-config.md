---
trigger: always_on
description: Use when [specific trigger condition].
---

# Claude Code Rules

> Operational directives for AI agents working in mjs-skills

---

## What This Repo Is

A personal Agent Skills library. Skills are frozen decisions—not tools, not utilities, not helpers. Each skill encodes judgment about what matters, what fails, and what works.

---

## Before You Do Anything

1. **Read the constitution**: `.specify/memory/constitution.md`
2. **Check existing skills**: `ls .claude/skills/`
3. **Understand the PRD**: `research/h3-skills-master-prd.md` for skill specifications
4. **Reference the spec**: `agentskills-standard/docs/specification.mdx` for Agent Skills format

---

## Task Context

**Your Surface:** You operate on a project level, providing guidance to users and executing development tasks via a defined set of tools.

**Your Success is Measured By:**
- All outputs strictly follow the user intent
- Skills comply with constitution principles and Agent Skills spec
- Prompt History Records (PHRs) created automatically and accurately
- ADR suggestions made intelligently for significant decisions
- All changes are small, testable, and reference code precisely

---

## Core Guarantees (Product Promise)

- Record every user input verbatim in a PHR after every user message
- PHR routing (all under `history/prompts/`):
  - Constitution → `history/prompts/constitution/`
  - Feature-specific → `history/prompts/<feature-name>/`
  - General → `history/prompts/general/`
- ADR suggestions: when an architecturally significant decision is detected, suggest: "📋 Architectural decision detected: <brief>. Document? Run `/sp.adr <title>`." Never auto-create ADRs; require user consent.

---

## Skill Development Rules

### Authoritative Standard

Follow the [Agent Skills Specification](agentskills-standard/docs/specification.mdx). When in doubt, the spec takes precedence. The constitution at `.specify/memory/constitution.md` adds project-specific constraints.

### Required Structure

```
.claude/skills/[skill-name]/
├── SKILL.md           # YAML frontmatter + instructions
└── scripts/
    └── verify.py      # Exit 0 (success) or 1 (failure)
```

### SKILL.md Template

```yaml
---
name: [gerund-form-name]
description: |
  [What it does in one sentence].
  Use when [specific trigger condition].
---

## Quick Start
[Immediate action for the 80% case]

## Instructions
1. [Step with command]
2. [Step with command]
3. `python scripts/verify.py`

## If Verification Fails
1. Run diagnostic: `[specific command]`
2. Check: `[what to look for]`
3. **Stop and report** — do not proceed with downstream steps
```

### verify.py Template

```python
#!/usr/bin/env python3
"""Verify [what this checks]."""
import subprocess
import sys

def main():
    result = subprocess.run([...], capture_output=True)

    if result.returncode == 0:
        print("✓ [Specific success message]")  # Keep under 100 chars
        sys.exit(0)
    else:
        print("✗ [Actionable error]. Run: [diagnostic command]")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

### Rules You Must Follow

#### Naming
- Use gerund form: `deploying-*`, `scaffolding-*`, `building-*`
- Max 64 characters, lowercase + hyphens only
- Exception: Pattern names can override if sharper (e.g., `kafka-exactly-once`)
- Test: Would someone search for this exact name?
- **BANNED**: `utils`, `helpers`, `setup`, `manage`, `handle`

#### Description
- MUST include "Use when [trigger]"
- Max 1024 characters (per Agent Skills spec)
- Use third person ("Deploys..." not "I deploy...")
- If collision possible, add "NOT when [exclusion]"
- **BANNED words**: "manage", "handle", "setup", "utilities", "helpers"

#### Token Discipline (Progressive Disclosure)

| Layer | Content | Budget | When Loaded |
|-------|---------|--------|-------------|
| 1. Metadata | name + description | ~100 tokens | Always (startup) |
| 2. Instructions | SKILL.md body | <5000 tokens | When skill activates |
| 3. Resources | scripts/, references/ | As needed | On demand only |

- SKILL.md: <500 lines, <5000 tokens
- verify.py output: <100 characters
- References: one level deep only

#### MCP Output Discipline
- NEVER inject raw MCP output into context
- ALWAYS filter/summarize before context entry
- Raw payloads stay in subprocess or temp files
- **BANNED patterns**: `print(full_response)`, `context += mcp_output`, `return transcript`

#### Verification
- Every skill MUST have verify.py
- On failure: exit 1 + actionable error + diagnostic command
- On success: exit 0 + minimal confirmation

#### Failure Escalation
- If verify.py fails twice: STOP
- Surface the diagnostic command
- Request human intervention
- Do NOT proceed with downstream steps

### Before Shipping Any Skill

```bash
# 1. Validate with skills-ref
skills-ref validate ./.claude/skills/[skill-name]

# 2. Structure check
ls .claude/skills/[skill-name]/SKILL.md
ls .claude/skills/[skill-name]/scripts/verify.py

# 3. Check frontmatter
head -20 .claude/skills/[skill-name]/SKILL.md
# Must have: name, description with "Use when"

# 4. Test verify.py
python .claude/skills/[skill-name]/scripts/verify.py
# Must exit 0 or 1 with message

# 5. Check line count
wc -l .claude/skills/[skill-name]/SKILL.md
# Should be <500 lines
```

### Routing Evaluation

Before declaring a skill complete, test routing:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mjunaidca/mjs-agent-skills](https://github.com/mjunaidca/mjs-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
