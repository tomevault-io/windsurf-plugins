---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository is a curated catalog of **semantic anchors** - well-defined terms, methodologies, and frameworks that serve as reference points when communicating with Large Language Models (LLMs). Semantic anchors act as shared vocabulary that triggers specific, contextually rich knowledge domains within an LLM's training data.

**Current Status:** The repository is undergoing a major redesign to become an interactive, bilingual website with treemap visualization, role-based filtering, and automated contribution workflow.

## First-Time Setup

After cloning, activate the shared Git hooks with one command (works on Unix/macOS and Windows Git Bash):

```bash
git config core.hooksPath .githooks
```

This installs the pre-commit hook that automatically syncs `skill/` into `plugins/semantic-anchors/skills/` on every commit. No other tools are required.

## Git Workflow & Fork Setup

**Development takes place in the fork until PRD implementation is complete.**

**Git Remotes:**
- `origin` → `raifdmueller/Semantic-Anchors` (fork for development)
- `upstream` → `LLM-Coding/Semantic-Anchors` (original repository)

**Workflow:**
1. All feature branches are created in the fork (`origin`)
2. PRs are created within the fork for review
3. After PRD implementation is complete, changes will be merged back to upstream
4. Use `git push origin <branch>` for all development work
5. Use `git fetch upstream` to sync with original repository if needed

**Issue References:**
- Issues are tracked in upstream repository: https://github.com/LLM-Coding/Semantic-Anchors/issues
- Reference issues in commits: `feat: implement X (#42)`

### Git Worktrees for Parallel Team Development

**When using agent teams, use Git Worktrees to enable true parallel development:**

**Setup:**
```bash
# Create worktree directory
mkdir -p ~/projects/Semantic-Anchors-worktrees

# Create worktree for each feature branch
git worktree add ../Semantic-Anchors-worktrees/feature-<name> -b feature/<name>
```

**Directory Structure:**
```
~/projects/
├── Semantic-Anchors/              # Main worktree (main branch)
└── Semantic-Anchors-worktrees/    # Feature branch worktrees
    ├── feature-mece-analysis/     # Teammate 1
    ├── feature-role-mapping/      # Teammate 2
    ├── feature-split-readme/      # Teammate 3
    └── feature-metadata-script/   # Teammate 4
```

**Team Workflow:**
1. Create worktrees for each feature branch
2. Spawn teammates with `working_directory` parameter pointing to their worktree
3. Each teammate works independently in their directory
4. Teammates commit and push to their feature branches
5. Create PRs from feature branches to main
6. After merge, cleanup: `git worktree remove <path>`

**Advantages:**
- No branch switching conflicts
- True parallel development
- Each teammate has isolated workspace
- Shared Git history (.git directory)
- Clean separation of concerns

## Project Documentation

All project documentation is located in the `docs/` directory:

- **`docs/PRD.md`**: Product Requirements Document with user personas, user stories, timeline
- **`docs/specs/`**: Detailed specifications
  - `01_use_cases.adoc`: 10 use cases with PlantUML diagrams
  - `02_api_specification.adoc`: Data models and API structure
  - `03_acceptance_criteria.adoc`: Gherkin scenarios (in German)
  - `adrs/`: Architecture Decision Records with Pugh matrices
- **`docs/arc42/`**: Complete architecture documentation (12 chapters)
- **`PROJECT_STATUS.md`**: Current implementation status and roadmap

**Always review these documents before starting work on implementation issues.**

## Current Repository Structure

```
Semantic-Anchors/
├── docs/                           # All documentation
│   ├── PRD.md                      # Product Requirements
│   ├── specs/                      # Specifications & ADRs
│   └── arc42/                      # Architecture (arc42 template)
├── README.adoc                     # Current catalog (will be split in Phase 1)
├── PROJECT_STATUS.md               # Implementation tracking
├── CLAUDE.md                       # This file
└── LICENSE
```

## Target Repository Structure (After Phase 1)

```
Semantic-Anchors/
├── docs/
│   ├── anchors/                    # Individual anchor files (60+)
│   │   ├── _template.adoc          # Template for new anchors
│   │   ├── tdd-london-school.adoc
│   │   └── ...
│   ├── categories/                 # Category include files
│   │   ├── testing-quality.adoc
│   │   └── ...
│   ├── roles/                      # Role include files
│   │   ├── software-developer.adoc
│   │   └── ...
│   ├── metadata/                   # Generated metadata
│   │   ├── categories.yml
│   │   ├── roles.yml
│   │   └── anchors.yml
│   └── ... (specs, arc42)
├── website/                        # Vite-based static site (Phase 2)
│   ├── src/
│   ├── public/
│   └── package.json
├── scripts/                        # Build scripts
│   ├── extract-metadata.js
│   └── generate-includes.js
└── ...
```

## File Format

All content is written in **AsciiDoc** format (.adoc), not Markdown. Key AsciiDoc patterns:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LLM-Coding/Semantic-Anchors](https://github.com/LLM-Coding/Semantic-Anchors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
