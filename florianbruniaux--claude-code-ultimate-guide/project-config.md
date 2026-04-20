---
trigger: always_on
description: This repository is the **comprehensive documentation for Claude Code** (Anthropic's CLI tool). It teaches users how to use Claude Code effectively through guides, examples, and templates.
---

# Claude Code Ultimate Guide - Project Context

## Purpose

This repository is the **comprehensive documentation for Claude Code** (Anthropic's CLI tool). It teaches users how to use Claude Code effectively through guides, examples, and templates.

**Meta-note**: This repo documents Claude Code, so its own configuration should be exemplary.

## Repository Structure

```
guide/                    # Core documentation
├── ultimate-guide.md     # Main guide (~20K lines, the reference)
├── cheatsheet.md         # 1-page printable summary
├── cowork.md             # Cowork redirect page
├── core/                 # Architecture, methodologies, releases, known-issues, visual-reference
├── security/             # security-hardening, sandbox-isolation, sandbox-native, production-safety, data-privacy
├── ecosystem/            # ai-ecosystem, mcp-servers-ecosystem, third-party-tools, remarkable-ai
├── roles/                # ai-roles, adoption-approaches, learning-with-ai, agent-evaluation
├── ops/                  # devops-sre, observability, ai-traceability
├── diagrams/             # Mermaid visual diagrams
└── workflows/            # Step-by-step workflow guides

examples/                 # Production-ready templates
├── agents/               # Custom agent templates
├── commands/             # Slash command templates
├── hooks/                # Event hook examples (bash/powershell)
├── skills/               # Skill module templates
└── scripts/              # Utility scripts (audit, health check)

machine-readable/         # For LLM consumption
├── reference.yaml        # Condensed index (~2K tokens)
└── llms.txt              # AI indexation file

whitepapers/              # Focused whitepapers (FR + EN)
├── fr/                   # 10 source files in French (.qmd)
└── en/                   # 10 translated files in English (.qmd)
# Published at: https://www.florian.bruniaux.com/guides

tools/                    # Interactive utilities
├── audit-prompt.md       # Setup audit prompt
└── onboarding-prompt.md  # Personalized learning prompt

docs/                     # Public documentation (tracked)
└── resource-evaluations/ # External resource evaluations (68 files)

claudedocs/               # Claude working documents (gitignored)
├── resource-evaluations/ # Research working docs (prompts, private audits)
└── *.md                  # Analysis reports, plans, working docs
```

## Key Files

| File | Purpose |
|------|---------|
| `VERSION` | Single source of truth for version (currently 3.34.1) |
| `guide/ultimate-guide.md` | The main reference (search here first) |
| `guide/cheatsheet.md` | Quick reference for daily use |
| `machine-readable/reference.yaml` | LLM-optimized index with line numbers |
| `CHANGELOG.md` | All changes with detailed descriptions |

## Commands

### Version Management
```bash
# Check version consistency across all docs
./scripts/sync-version.sh --check

# Fix version mismatches (updates from VERSION file)
./scripts/sync-version.sh

# Bump version
echo "3.7.0" > VERSION && ./scripts/sync-version.sh
```

### Whitepaper, Recap Cards & Guide Export

Full build commands (PDF/EPUB/recap-card), stack details, ebook versioning, and Typst template sync rules:

@docs/workflows/whitepaper-build.md

### Before Committing
```bash
# Verify versions are synchronized
./scripts/sync-version.sh --check
```

### Slash Commands (Maintenance)

Custom slash commands available in this project:

| Command | Description |
|---------|-------------|
| `/release <bump-type>` | Release guide version (CHANGELOG + VERSION + sync + commit + push) |
| `/update-infos-release [bump-type]` | Update Claude Code releases tracking + optional guide version bump |
| `/version` | Display current guide and Claude Code versions with stats |
| `/changelog [count]` | View recent CHANGELOG entries (default: 5) |
| `/sync` | Check guide/landing synchronization status |
| `/audit-agents-skills [path]` | Audit quality of agents, skills, and commands in .claude/ config |
| `/security-check` | Quick config check against known threats database (~30s) |
| `/security-audit` | Full 6-phase security audit with score /100 (2-5min) |
| `/update-threat-db` | Research & update threat intelligence database |

**Examples:**
```
/release patch                 # Bump patch + release (3.20.4 → 3.20.5)
/release minor                 # Bump minor + release (3.20.4 → 3.21.0)
/update-infos-release          # Update CC releases only
/update-infos-release patch    # Update CC + bump guide (3.9.11 → 3.9.12)
/update-infos-release minor    # Update CC + bump guide (3.9.11 → 3.10.0)
/version                       # Show versions and content stats
/changelog 10                  # Last 10 CHANGELOG entries
/sync                          # Check guide/landing sync status
/audit-agents-skills           # Audit current project
/audit-agents-skills --fix     # Audit + fix suggestions
/audit-agents-skills ~/other   # Audit another project
/security-check                # Quick scan config vs known threats
/security-audit                # Full audit with posture score /100
/update-threat-db              # Research + update threat-db.yaml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlorianBruniaux/claude-code-ultimate-guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
