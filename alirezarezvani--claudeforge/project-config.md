---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**ClaudeForge** is a comprehensive toolkit for automated CLAUDE.md creation, enhancement, and maintenance for Claude Code projects. The repository consists of three integrated components:

1. **Skill** (`claudeforge-skill`) - Core Python modules for analysis, generation, and validation
2. **Slash Command** (`/enhance-claude-md`) - Interactive multi-phase discovery workflow
3. **Guardian Agent** (`claude-md-guardian`) - Background maintenance agent

---

## What's New in v2.0.0

**Claude Code v2.1.4+ Support:**
- **Lifecycle Hooks**: Guardian agent uses SessionStart, PreToolUse, PostToolUse hooks
- **Modern Permissions**: Updated to `permissions:` array syntax
- **Hot-Reload**: Skills auto-reload when modified (no restart needed)
- **Fork-Safe Mode**: Guardian runs independently with `fork_safe: true`
- **Auto-Migration**: Seamless upgrade from v1.x with automatic backups

**Migration:** See `docs/MIGRATION_V2.md` for upgrading from v1.0.0.

---

## Architecture

### Component Interaction Flow

```
User Project
    ↓
/enhance-claude-md (Slash Command)
    ↓
[Phase 1: Discovery] → [Phase 2: Analysis] → [Phase 3: Task]
    ↓
claude-md-guardian (Agent) OR Direct Skill Invocation
    ↓
claudeforge-skill (Python Modules)
    ↓
workflow.py → analyzer.py → validator.py → template_selector.py → generator.py
    ↓
CLAUDE.md Created/Updated with 100% Native Format
```

### Python Module Architecture

The skill consists of 5 core modules (~2,190 lines):

**workflow.py (432 lines)** - `InitializationWorkflow` class
- Orchestrates interactive initialization for new projects
- Methods: `check_claude_md_exists()`, `generate_exploration_prompt()`, `analyze_discoveries()`
- Detects: project type, tech stack, team size, development phase, workflows
- Returns: Project context dictionary for template selection

**analyzer.py (382 lines)** - `CLAUDEMDAnalyzer` class
- Analyzes existing CLAUDE.md files
- Methods: `analyze_file()`, `detect_sections()`, `calculate_quality_score()`, `generate_recommendations()`
- Quality scoring: 0-100 based on length (25pts), completeness (25pts), formatting (20pts), specificity (15pts), modularity (15pts)
- Returns: Analysis report with quality score and actionable recommendations

**validator.py (429 lines)** - `BestPracticesValidator` class
- Validates against Anthropic guidelines
- Methods: `validate_length()`, `validate_structure()`, `validate_formatting()`, `validate_completeness()`, `validate_all()`
- Checks: file length (20-300 lines), required sections, markdown formatting, anti-patterns
- Returns: Validation report with pass/fail status and detailed issues

**template_selector.py (467 lines)** - `TemplateSelector` class
- Selects appropriate template based on context
- Methods: `select_template()`, `customize_template()`, `recommend_modular_structure()`
- Logic: Maps project type + team size → template complexity
- Returns: Template configuration with target line count and modular recommendation

**generator.py (480 lines)** - `ContentGenerator` class
- Generates new or enhanced CLAUDE.md content
- Methods: `generate_root_file()`, `generate_context_file()`, `generate_section()`, `merge_with_existing()`
- Supports: Root files, context-specific files (backend/, frontend/, database/), individual sections
- Returns: Complete CLAUDE.md content with 100% native format compliance

### Critical Validation Rule

All generated CLAUDE.md files MUST include:
- Project structure (ASCII tree diagram)
- File structure explanations
- Setup & installation instructions
- Architecture section (for complex projects)
- Validation against `/update-claude-md` slash command format
- Cross-check against reference examples in `skill/examples/`

---

## Installation & Testing

### Test Installation Scripts

```bash
# macOS/Linux
./install.sh
# Choose option 1 (user-level) or 2 (project-level)
# Verify installation at ~/.claude/ or ./.claude/

# Windows
.\install.ps1
# Same options as above

# Verify components installed:
ls -la ~/.claude/skills/claudeforge-skill/
ls -la ~/.claude/commands/enhance-claude-md/
ls -la ~/.claude/agents/claude-md-guardian.md
```

### Directory Structure After Installation

```
~/.claude/                           # User-level installation
├── skills/
│   └── claudeforge-skill/          # Copied from skill/
│       ├── SKILL.md
│       ├── analyzer.py
│       ├── validator.py
│       ├── generator.py
│       ├── template_selector.py
│       ├── workflow.py
│       └── examples/               # 7 reference templates
├── commands/
│   └── enhance-claude-md/          # Copied from command/
│       └── enhance-claude-md.md
└── agents/
    └── claude-md-guardian.md       # Copied from agent/
```

---

## Development Workflow

### Modifying Python Modules

When updating skill modules (analyzer.py, generator.py, etc.):

1. Edit files in `skill/` directory
2. Test changes by reinstalling: `./install.sh` (choose option 2 for project-level testing)
3. Test skill invocation in Claude Code: `/enhance-claude-md`
4. Validate output against reference examples in `skill/examples/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alirezarezvani/ClaudeForge](https://github.com/alirezarezvani/ClaudeForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
