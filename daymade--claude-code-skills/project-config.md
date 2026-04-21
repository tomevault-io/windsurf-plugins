---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code skills marketplace containing 48 production-ready skills organized in a plugin marketplace structure. Most plugins expose one skill for narrow installs; suite plugins expose related skills under shared namespaces for combined installation workflows.

**Essential Skill**: `skill-creator` is the most important skill in this marketplace - it's a meta-skill that enables users to create their own skills. Always recommend it first for users interested in extending Claude Code.

## Skills Architecture

### Directory Structure

Each skill follows a standard structure:
```
skill-name/
├── SKILL.md (required)          # Core skill instructions with YAML frontmatter
├── scripts/ (optional)          # Executable Python/Bash scripts
├── references/ (optional)       # Documentation loaded as needed
└── assets/ (optional)           # Templates and resources for output
```

### Progressive Disclosure Pattern

Skills use a three-level loading system:
1. **Metadata** (name + description in YAML frontmatter) - Always in context
2. **SKILL.md body** - Loaded when skill triggers
3. **Bundled resources** - Loaded as needed by Claude

## Development Commands

### Installation Scripts

**In Claude Code (in-app):**
```text
/plugin marketplace add daymade/claude-code-skills
```

Then:
1. Select **Browse and install plugins**
2. Select **daymade/claude-code-skills**
3. Select **skill-creator**
4. Select **Install now**

**From your terminal (CLI):**
```bash
# Automated installation (macOS/Linux)
curl -fsSL https://raw.githubusercontent.com/daymade/claude-code-skills/main/scripts/install.sh | bash

# Automated installation (Windows PowerShell)
iwr -useb https://raw.githubusercontent.com/daymade/claude-code-skills/main/scripts/install.ps1 | iex

# Manual installation
claude plugin marketplace add https://github.com/daymade/claude-code-skills
# Marketplace name: daymade-skills (from marketplace.json)
claude plugin install skill-creator@daymade-skills
```

### Skill Validation and Packaging

```bash
# Quick validation of a skill
cd skill-creator && uv run --with PyYAML python -m scripts.quick_validate ../skill-name

# Package a skill (includes automatic validation)
cd skill-creator && uv run --with PyYAML python -m scripts.package_skill ../skill-name [output-dir]

# Initialize a new skill from template
uv run python skill-creator/scripts/init_skill.py <skill-name> --path <output-directory>
```

### Testing Skills Locally

```bash
# Add local marketplace
claude plugin marketplace add https://github.com/daymade/claude-code-skills
# Marketplace name: daymade-skills (from marketplace.json)

# Install specific skill (start with skill-creator)
claude plugin install skill-creator@daymade-skills

# Test by copying to user skills directory
cp -r skill-name ~/.claude/skills/
# Then restart Claude Code
```

In Claude Code, use `/plugin ...` slash commands. In your terminal, use `claude plugin ...`.

### Git Operations

This repository uses standard git workflow, but **always stage files by name**,
never `git add -A` / `git add .`. Multiple agents may have unstaged changes in
the same worktree — a blanket stage piggybacks their work into your commit:

```bash
git status
git add path/to/file1 path/to/file2   # specific files only
git commit -m "message"
git push
```

## Skill Writing Requirements

### Writing Style

Use **imperative/infinitive form** (verb-first instructions) throughout all skill content:
- ✅ "Extract files from a repomix file using the bundled script"
- ❌ "You should extract files from a repomix file"

### YAML Frontmatter Requirements

Every SKILL.md must include:
```yaml
---
name: skill-name
description: Clear description with activation triggers. This skill should be used when...
---
```

### Privacy and Path Guidelines (Enforced by Pre-commit Hook)

Skills for public distribution must NOT contain:
- Absolute paths to user directories (`/home/username/`, `/Users/username/`)
- Personal usernames, company names, product names
- Phone numbers, personal email addresses
- OneDrive paths or environment-specific absolute paths
- Use relative paths within skill bundle or standard placeholders (`<workspace>/`, `<user_id>`)

**Four-layer defense system:**
1. **CLAUDE.md rules** (this section) — Claude avoids generating sensitive content
2. **Global PII Guard pre-commit hook** (`~/scripts/git-pii-guard/pre-commit`) — blocks staged PII/secrets and generated/local artifact paths
3. **Global PII Guard pre-push hook** (`~/scripts/git-pii-guard/pre-push`) — scans commits about to be pushed, catching bad local history before it hits GitHub
4. **gitleaks** (`.gitleaks.toml`) — deep scan with custom rules for this repo

PII Guard is enabled via `~/scripts/git-pii-guard/manage.sh enable <repo-path>`, which sets `core.hooksPath` to `~/scripts/git-pii-guard`.
For repo-specific additions:
- `.pii-patterns` — extra content regexes
- `.pii-path-patterns` — extra forbidden path regexes
- `.pii-allowpaths` — explicit path allowlist exceptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daymade/claude-code-skills](https://github.com/daymade/claude-code-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
