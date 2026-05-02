---
trigger: always_on
description: > Universal guidelines for AI coding agents working on the Qodo Skills plugin project.
---

# Qodo Skills - Agent Guidelines

> Universal guidelines for AI coding agents working on the Qodo Skills plugin project.

## Project Overview

**Purpose**: Shift-left code review skills that bring Qodo's quality standards into local development workflow.

**Compatible Agents**: Claude Code, Cursor, Windsurf, Cline, and any agent supporting the [Agent Skills](https://agentskills.io) standard.

## Repository Structure

```
qodo-skills/
├── AGENTS.md                    # This file - universal guidelines
├── CLAUDE.md                    # Claude Code-specific directives
├── references/                  # Shared reference docs for all skills
│   └── usage-tracking.md        # Required HTTP headers for Qodo API calls
├── skills/                      # Agent Skills catalog
│   ├── qodo-get-rules/         # Fetch relevant Qodo coding rules
│   │   ├── SKILL.md            # Skill specification
│   │   ├── AGENTS.md           # Skill-specific guidelines
│   │   └── references/         # Skill-specific reference docs
│   └── qodo-pr-resolver/       # Fix PR review issues
│       ├── SKILL.md
│       └── AGENTS.md
```

## Technology Stack

- **Python 3.6+**: Core scripting (standard library only, zero external dependencies)
- **Bash/Shell**: Cross-platform wrappers (`.sh` for Unix, `.cmd` for Windows)
- **Git**: Repository detection and remote URL parsing
- **Agent Skills Standard**: SKILL.md format for skill definitions

## File Size Guidelines (CRITICAL)

**Keep all files digestible to agents:**

- **Maximum**: 500 lines per file
- **Ideal**: ~300 lines per file
- **Reason**: Frontier LLMs handle ~150-200 instructions consistently; file size limits prevent context overload

**When files exceed limits:**

1. **Split by scope**: Create hierarchical AGENTS.md/CLAUDE.md in subdirectories
2. **Use progressive disclosure**: Point to detailed docs instead of including everything
3. **Refactor code**: Break large scripts into smaller modules
4. **Extract common patterns**: Move shared code to utilities

**Examples of hierarchical structure:**

```
project/
├── AGENTS.md                    # Root guidelines (~200 lines)
├── skills/
│   ├── qodo-get-rules/
│   │   ├── AGENTS.md           # Skill-specific context (~150 lines)
│   │   └── SKILL.md            # Skill instructions (~250 lines)
│   └── qodo-pr-resolver/
│       ├── AGENTS.md           # Skill-specific context (~150 lines)
│       └── SKILL.md            # Skill instructions (~400 lines)
```

## Skill Creation Guidelines

### SKILL.md Format

Every skill requires a SKILL.md file with YAML frontmatter:

```markdown
---
name: qodo-my-skill
description: "Brief description (1-2 sentences) for skill discovery"
allowed-tools: ["Bash", "Read", "Edit", "Write"]
triggers:
  - "qodo.?my.?skill"
  - "invoke.my.skill"
---

# Skill Name

## Description
[Detailed explanation]

## Prerequisites
[Required tools, config, permissions]

## Instructions
[Step-by-step workflow]

## Configuration
[How to configure]

## Error Handling
[Common issues and solutions]
```

### Skill Naming Convention

**All skills must follow the `qodo-*` prefix pattern:**

- `qodo-get-rules` ✅
- `qodo-pr-resolver` ✅
- `get-qodo-rules` ❌ (old pattern, do not use)
- `my-skill` ❌ (missing prefix)

This makes skills identifiable at a glance and prevents naming collisions with other skill collections.

### YAML Frontmatter Fields

**Required:**
- `name`: Unique identifier using `qodo-*` prefix (lowercase, hyphens only)
- `description`: Brief description for discovery (1-2 sentences)

**Optional but STRONGLY RECOMMENDED:**
- `triggers`: Array of patterns for skill invocation (use regex-like patterns)
  - Example: `"qodo.?fix"` matches "qodo-fix", "qodo fix", "qodofix"
  - **Why**: Improves usability by allowing natural language invocation
  - **Best practice**: Include 2-3 common variations of skill name

- `allowed-tools`: Array of tool names (restrict for security)
  - Common: `["Bash", "Read", "Edit", "Write", "Grep", "Glob"]`
  - Advanced: `["WebFetch", "WebSearch", "Task"]`

### Instruction Writing Standards

**Do:**
- ✅ Be specific: Exact commands and code snippets
- ✅ Use examples: Concrete inputs/outputs
- ✅ Handle errors: Include edge cases and error messages
- ✅ Be step-by-step: Break complex workflows into numbered steps
- ✅ Use markdown formatting: Headers, code blocks, lists
- ✅ Test thoroughly: Verify in different scenarios

**Don't:**
- ❌ Be vague: Avoid "do the right thing" instructions
- ❌ Assume context: Explain domain-specific concepts
- ❌ Skip error handling: Always include graceful fallbacks
- ❌ Over-complicate: Keep it as simple as possible
- ❌ Forget edge cases: Consider what could go wrong

### Helper Scripts

Place scripts in `scripts/` directory:

**Template:**
```bash
#!/bin/bash
set -euo pipefail  # Exit on error, undefined var, pipe failure

# Description: What this script does
# Usage: ./helper.sh [args]

# Check prerequisites
if ! command -v required_tool &> /dev/null; then
    echo "Error: required_tool is not installed" >&2
    exit 1
fi

# Main logic
main() {
    # Your code here
    echo "Success message"
}

main "$@"
```

**Exit codes:**
- `0`: Success
- `1-255`: Error (use consistent codes)

## Cross-Platform Considerations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qodo-ai/qodo-skills](https://github.com/qodo-ai/qodo-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
