---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is **BMAD Skills** - a complete workflow ecosystem packaged as Claude Skills. It provides 16 integrated skills that guide users from idea to implementation through natural conversation. This is a **skills library repository**, not an application codebase.

## Core Architecture

### Two-Track System

The repository implements two complementary workflows:

1. **BMAD Track** (11 skills): End-to-end product development for Level 2-4 complexity
   - main-workflow-router → bmad-discovery-research → bmad-product-planning → bmad-ux-design → bmad-architecture-design → bmad-test-strategy → bmad-performance-optimization → bmad-observability-readiness → bmad-security-review → bmad-story-planning → bmad-development-execution
   - Handles: New products, complex features, multi-team coordination

2. **OpenSpec Track** (3 skills): Lightweight change management for Level 0-1 complexity
   - openspec-change-proposal → openspec-change-implementation → openspec-change-closure
   - Handles: Bug fixes, small features, quick changes

The **main-workflow-router** automatically routes work to the appropriate track based on complexity assessment.

### Self-Contained Bundle Architecture

Everything is contained in `.claude/skills/`:

```
.claude/skills/
├── [16 skill directories]     # Each with SKILL.md, REFERENCE.md, WORKFLOW.md, CHECKLIST.md
├── _core/                      # Shared resources (glossary, constraints, quality-gates)
├── _config/                    # Configuration (MANIFEST.json, STYLE-GUIDE.md)
├── _runtime/                   # Runtime workspace for OpenSpec changes
└── _docs/                      # Documentation organized by type
```

**Critical constraint:** All skills must reference resources using paths relative to `.claude/skills/` because the bundle is installed to various locations (`~/.claude/skills/` or `project/.claude/skills/`).

### Dual Manifest Locations

For marketplace compatibility:
- `meta/MANIFEST.json` - Required at root for marketplace tooling
- `.claude/skills/_config/MANIFEST.json` - Used by skills internally

**Both must stay in sync.** When updating skills, update both files.

## Key Commands

### Installation & Distribution

```bash
# Verify current installation
bash scripts/verify.sh [path]

# Install to ~/.claude/skills (global)
bash scripts/install-to-home.sh

# Install to current project
bash scripts/install-to-project.sh

# Create marketplace distribution bundle
bash scripts/package-bundle.sh
```

### Validation & Testing

```bash
# Validate a single skill
python .claude/skills/core-skill-creation/scripts/quick_validate.py .claude/skills/[skill-name]

# Validate all skill contracts
python .claude/skills/_core/tooling/lint_contracts.py

# Run static tests only (metadata, templates, contracts)
npm test

# Run specific test suite
pytest tests/test_skill_metadata.py
pytest tests/test_manifest_consistency.py
```

### Conversational Workflow Testing (Manual)

**Important:** BMAD Skills uses conversational workflows that cannot be automated with traditional E2E tests. Tests are performed manually through actual Claude conversations.

**Why No Automated E2E Tests:**
- Skills require multi-turn interactive conversations (questions/responses)
- Claude batch mode (`claude -p`) doesn't support interactive dialogue
- Workflow transitions depend on conversational context
- See `.project-archive/TESTING.md` for detailed explanation

**Manual Test Approach:**
```bash
# Test workflows through actual Claude conversations
claude  # Interactive mode

# Example workflow to test:
> "I want to create a budgeting app for students"
# Verify: Discovery skill activates
# Verify: PRD is generated in _runtime/workspace/artifacts/
# Verify: Content quality and structure

> "Design the architecture"
# Verify: Architecture skill activates
# Verify: Architecture doc generated
# Verify: Technical decisions documented

> "Break this down into developer stories"
# Verify: Story planning skill activates
# Verify: Stories generated in _runtime/workspace/stories/
# Verify: Each story has acceptance criteria
```

**What Manual Testing Validates:**
- ✅ Skills activate correctly based on conversational prompts
- ✅ Artifacts are generated in correct locations
- ✅ Artifact structure meets requirements (sections, frontmatter)
- ✅ Content quality (keywords, relevance, completeness)
- ✅ Multi-turn context maintenance across skill transitions
- ✅ Full workflow cycles (Discovery → Planning → Architecture → Stories)

**Documentation:**
- See `.project-archive/TESTING.md` for complete test strategy
- See `.project-archive/IMPROVEMENTS.md` for automated test coverage details

### OpenSpec Helper Scripts

```bash
# Create new OpenSpec change workspace
python .claude/skills/openspec-change-proposal/scripts/scaffold_change.py [change-id]

# Update execution log
python .claude/skills/openspec-change-implementation/scripts/update_execution_log.py [change-id] "message"

# Archive completed change
python .claude/skills/openspec-change-closure/scripts/archive_change.py [change-id]
```

### Metrics & Analysis

```bash
# Export activation metrics report

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bacoco/BMad-Skills](https://github.com/bacoco/BMad-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
