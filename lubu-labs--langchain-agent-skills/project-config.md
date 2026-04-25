---
trigger: always_on
description: `AGENTS.md` is the single source of truth for contributor and agent guidance in this repository. All workflow, tooling, and architecture rules live here. `CLAUDE.md` only points to this document.
---

# Repository Guidelines

`AGENTS.md` is the single source of truth for contributor and agent guidance in this repository. All workflow, tooling, and architecture rules live here. `CLAUDE.md` only points to this document.

## Project Overview
This repository contains a collection of LangChain/LangGraph/LangSmith/Deep Agents skills for AI coding assistants. Skills are modular, self-contained packages that extend an agent's capabilities with specialized knowledge, workflows, and tools for the LangChain ecosystem.

## Repository Structure
```
skills/
├── skill-creator/              # Meta-skill for creating new skills
├── deepagents-setup-configuration/ # Skill for Deep Agents initialization, configuration, and validation
├── deepagents-planning-todos/  # Skill for task planning and decomposition with write_todos
├── langgraph-project-setup/    # Skill for initializing LangGraph projects
├── langgraph-agent-patterns/   # Skill for multi-agent coordination patterns
├── langgraph-state-management/ # Skill for state schemas, reducers, persistence
├── langgraph-error-handling/   # Skill for retry, recovery, and escalation
├── langgraph-testing-evaluation/ # Skill for testing and evaluating agents
├── langsmith-trace-analyzer/   # Skill for fetching and analyzing LangSmith traces
├── langsmith-deployment/       # Skill for deploying agents to production
└── [1 planned skill]

Each skill follows this structure:
skill-name/
├── SKILL.md                 # Required: YAML frontmatter + markdown instructions
├── scripts/                 # Executable automation (Python/JavaScript/TypeScript)
├── references/              # Detailed documentation (loaded on-demand)
└── assets/                  # Templates, examples, schemas (optional)
```
- Package output (`*.skill`) is optional for distribution/export and should not be treated as source-of-truth.
- Marketplace/plugin paths should point to `skills/<skill-name>` directories in this repository.
- Roadmap and ordering are tracked in `PLAN.md`.

## Marketplace Configuration (Claude Code)
- Marketplace manifest: `.claude-plugin/marketplace.json`.
- This file lists installable skills, versions, and local paths for Claude Code Marketplace.
- Update it whenever you add/remove skills or bump versions.

## Development Environment

### Python Setup and Command Priority
**Always prioritize `uv` for all Python operations. Only fall back to `python3` if `uv` is not accessible.**

This applies to:
- Running scripts: `uv run script.py` (not `python3 script.py`)
- Managing environments: `uv venv`, `uv sync` (not `python -m venv`, `pip install`)
- Installing packages: prefer `uv add <package>` for project dependencies, and `uv pip install <package>` for ad-hoc environment installs

```bash
# Create virtual environment
uv venv --python=3.12

# Sync/install dependencies
uv sync

# Add dependency to the project
uv add pyyaml

# Or manually install specific packages in the current environment
uv pip install pyyaml
```

**Command Priority Order:**
1. **Preferred:** `uv run <script>` - Automatically manages environment
2. **Alternative:** Activate venv + `python3` - If uv unavailable
3. **Never use:** `python` - Always use `python3` or `uv run`

## Key Commands
**Initialize a new skill:**
```bash
# Preferred
uv run skills/skill-creator/scripts/init_skill.py <skill-name> --path skills/

# Fallback if uv not available
python3 skills/skill-creator/scripts/init_skill.py <skill-name> --path skills/
```

**Validate skill structure:**
```bash
# Preferred
uv run skills/skill-creator/scripts/quick_validate.py skills/<skill-name>/

# Fallback if uv not available
source .venv/bin/activate
python3 skills/skill-creator/scripts/quick_validate.py skills/<skill-name>/
```

**Package skill for distribution:**
```bash
# Preferred
uv run skills/skill-creator/scripts/package_skill.py skills/<skill-name>/ dist/

# Fallback if uv not available
source .venv/bin/activate
python3 skills/skill-creator/scripts/package_skill.py skills/<skill-name>/ dist/
```

## Skill Development Workflow
The canonical process for implementing skills is documented in `PLAN.md`. Follow this sequence:

1. **Initialize**: Run `uv run skills/skill-creator/scripts/init_skill.py <skill-name> --path skills/` to create skill directory structure
2. **Implement scripts**: Create Python and JavaScript versions of automation scripts
3. **Write references**: Create detailed documentation files for conditional loading
4. **Prepare assets**: Add templates, examples, or schemas
5. **Write SKILL.md**:
   - YAML frontmatter with comprehensive `description` (primary triggering mechanism)
   - Keep body under 500 lines
   - Link to reference files for detailed content
6. **Validate**: Run `uv run skills/skill-creator/scripts/quick_validate.py skills/<skill-name>/` to check structure
7. **Package**: Run `uv run skills/skill-creator/scripts/package_skill.py skills/<skill-name>/ dist/` to create optional `.skill` export artifacts
8. **Test**: Verify scripts execute correctly (use `uv run` for Python scripts) and skill triggers appropriately

## Skill Architecture Principles

### Progressive Disclosure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lubu-Labs/langchain-agent-skills](https://github.com/Lubu-Labs/langchain-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
