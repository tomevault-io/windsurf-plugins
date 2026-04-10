---
trigger: always_on
description: **Repository**: https://github.com/secondsky/claude-skills
---

# Claude Skills - Project Context

**Repository**: https://github.com/secondsky/claude-skills
**Purpose**: Production-ready skills for Claude Code CLI
**Owner**: Claude Skills Maintainers
**Status**: Active Development | 170 Plugins | 167 Skills Complete
**Last Updated**: 2026-04-08
**Last Audit**: 2026-04-08 (dependency audit)

---

## ⚠️ CRITICAL: Repository Policy - NEVER TOUCH JEZWEB REPO

**🚨 ABSOLUTE RULE: The `jezweb/claude-skills` repository must NEVER be modified under ANY circumstances! 🚨**

**ONLY work in the official repository: https://github.com/secondsky/claude-skills**

### ❌ FORBIDDEN ACTIONS (jezweb repo):
- ❌ NEVER push commits to `jezweb` remote
- ❌ NEVER create PRs to `jezweb/claude-skills`
- ❌ NEVER create branches targeting `jezweb`
- ❌ NEVER make ANY changes to `jezweb` repository
- ❌ NEVER use `git push jezweb` or `git push upstream` (both point to jezweb)
- ❌ NEVER use `gh pr create` without explicitly specifying `--repo secondsky/claude-skills`

### ✅ REQUIRED ACTIONS (secondsky repo):
- ✅ ALWAYS use `origin` remote pointing to `secondsky/claude-skills`
- ✅ ALWAYS push to: `git push origin <branch>`
- ✅ ALWAYS create PRs to: `secondsky/claude-skills` via `gh pr create --repo secondsky/claude-skills`
- ✅ ALWAYS verify remote before pushing: `git remote -v` (check origin = secondsky)

### Why This Matters:
The `jezweb/claude-skills` is a PERSONAL FORK that must remain untouched. All development happens in `secondsky/claude-skills`. Pushing to jezweb would corrupt the fork and cause serious issues.

**If user says "push" or "create PR" without specifying repo, ALWAYS default to secondsky/claude-skills.**

---

## ⚠️ CRITICAL: Plugin Development Prerequisites

### Install Official Plugin-Dev Toolkit (REQUIRED)

```bash
/plugin install plugin-dev@claude-code-marketplace
```

**What it provides**:
- Hook development → `hook-development` skill
- MCP integration → `mcp-integration` skill
- Plugin structure → `plugin-structure` skill
- Agent creation → `agent-creator` agent
- Command creation → `command-development` skill
- Skill creation → `skill-development` skill

**What our documentation provides**:
- Marketplace management (unique to this repository)
- System prompt budget optimization (production issue, 15k char limit)
- Batch operations for 169 skills
- Repository-specific workflows

See [PLUGIN_DEV_BEST_PRACTICES.md](docs/guides/PLUGIN_DEV_BEST_PRACTICES.md) for repository-specific guidance.

---

## What This Repository Is

This is a curated collection of **production-tested Claude Code skills** for building modern web applications. Skills are modular capabilities that extend Claude's knowledge in specific domains, enabling faster development with fewer errors.

**Focus**: Claude Code CLI skills (not claude.ai web interface)

**Target Audience**: Developers building with Cloudflare, React, Tailwind v4, and AI integrations.

---

## Quick Navigation

**👋 First Time Here?** → Read [START_HERE.md](docs/getting-started/START_HERE.md)
**🔨 Building a Skill?** → See [QUICK_WORKFLOW.md](docs/getting-started/QUICK_WORKFLOW.md)
**✅ Verifying Work?** → Check [ONE_PAGE_CHECKLIST.md](docs/getting-started/ONE_PAGE_CHECKLIST.md)
**📂 Need Project Structure?** → See [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md)

---

## Codebase Exploration

**📖 For detailed project structure, see [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md)** (generated with codemap)

Use the `codemap` CLI tool ([github.com/JordanCoin/codemap](https://github.com/JordanCoin/codemap)) to quickly understand the project structure:

```bash
# Generate project tree with file stats
codemap .

# Show dependency flow (imports/exports)
codemap --deps .

# Files changed vs main branch
codemap --diff

# Check impact of a file (who imports it)
codemap --importers lib/accounting/ledger.ts

# Limit tree depth
codemap --depth 2 .

# Filter by extension
codemap --only ts,tsx .
```

Install: `brew tap JordanCoin/tap && brew install codemap`

## Skill Review Process

When reviewing skills, use manual review with the ONE_PAGE_CHECKLIST:

**Manual Review Steps**:
1. Check against [ONE_PAGE_CHECKLIST.md](docs/getting-started/ONE_PAGE_CHECKLIST.md)
2. Verify package versions: `scripts/check-versions.sh`
3. Test installation: `./scripts/install-skill.sh <skill-name>`
4. Validate YAML frontmatter and structure
5. Check compliance with [claude-code-skill-standards.md](docs/reference/claude-code-skill-standards.md)

**Advanced Validation**:
Use official plugin-dev toolkit: `/plugin install plugin-dev@claude-code-marketplace`

---

## ⚠️ CRITICAL: Manual Review & Refactoring Process

**ALWAYS use MANUAL approaches when reviewing and refactoring skills.**

### What This Means:

**✅ ALLOWED:**
- Using existing scripts in `scripts/` directory (e.g., `review-skill.sh`, `check-versions.sh`)
- Using standard tools: Read, Edit, Write, Grep, Glob, Bash
- Manual analysis and judgment
- Reading files to understand structure
- Using ONE_PAGE_CHECKLIST.md for manual review

**❌ FORBIDDEN:**
- Creating NEW Python/shell scripts to automate refactoring
- Using sed/awk to programmatically rewrite large sections
- Batch processing multiple files without human review of each change
- Auto-generating content via scripts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/secondsky) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
