---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the **Universal Skills Manager** skill, which acts as a centralized skill manager for AI capabilities across multiple AI coding tools (Gemini CLI, Google Anti-Gravity, OpenCode, Claude Code, Cline, Cursor, etc.).

The skill enables:
- **Discovery**: Searching for skills from multiple sources — SkillsMP.com (curated, AI semantic search), SkillHub (community skills, no API key required), and ClawHub (versioned skills, semantic search, no API key required)
- **Installation**: Installing skills from GitHub or ClawHub to User-level (global) or Project-level (local) scopes
- **Synchronization**: Copying/syncing skills across different AI tools
- **Consistency**: Maintaining version consistency across installed locations
- **Cloud Packaging**: Creating ready-to-upload ZIP files for claude.ai/Claude Desktop/ChatGPT with embedded API keys

## Architecture

This is a **skill definition repository** containing the Universal Skills Manager in the `universal-skills-manager/` subfolder. It is not a traditional codebase with source files.

### Repository Structure

```
universal-skills-manager/
├── README.md                       # Installation & usage documentation
├── CLAUDE.md                       # This file - technical context
├── SECURITY.md                     # Security policy and vulnerability reporting
├── specs.md                        # Technical specification for install script
├── docs/
│   ├── TECHNICAL.md                # Technical reference (APIs, scripts, security details)
│   ├── SECURITY_SCANNING.md        # Security scanner reference
│   ├── scan_skill-security-analysis.md  # Full security analysis of scanner
│   └── remediation-final-code-review.md # Code review of security hardening
├── tests/
│   ├── conftest.py                 # Test fixtures (scanner, tmp_skill helpers)
│   ├── test_scan_skill.py          # Scanner test suite (65 tests)
│   └── test_sync_skills.py         # Sync reporter test suite (63 tests)
└── universal-skills-manager/       # The skill folder
    ├── SKILL.md                    # Skill definition and logic
    └── scripts/
        ├── install_skill.py        # Python helper for downloading skills from GitHub
        ├── scan_skill.py           # Security scanner (20+ detection categories)
        ├── sync_skills.py          # Read-only sync status reporter across AI tools
        └── validate_frontmatter.py # Cloud platform YAML frontmatter validator
```

### Skill Structure

The `SKILL.md` file follows this format:
- **Frontmatter**: YAML metadata (name, description, homepage, metadata block with runtime requirements)
- **Documentation**: Markdown content describing when to use the skill, capabilities, operational rules
- **Implementation details**: Instructions for the AI agent on how to execute skill functionality

### Supported Tool Ecosystem

The skill manages skills across these AI tools and their respective paths:

| Tool | User Scope (Global) | Project Scope (Local) |
|------|---------------------|----------------------|
| Gemini CLI / Codex | `~/.agents/skills/` | `./.agents/skills/` |
| Google Anti-Gravity | `~/.gemini/antigravity/skills/` | `./.antigravity/extensions/` |
| OpenCode | `~/.config/opencode/skills/` | `./.opencode/skills/` |
| OpenClaw | `~/.openclaw/workspace/skills/` | `./.openclaw/skills/` |
| CC-Claw | `~/.cc-claw/workspace/skills/` | N/A (daemon, no project scope) |
| Claude Code | `~/.claude/skills/` | `./.claude/skills/` |
| block/goose | `~/.config/goose/skills/` | `./.goose/agents/` |
| Roo Code | `~/.roo/skills/` | `./.roo/skills/` |
| Cursor | `~/.cursor/skills/` | `./.cursor/skills/` |
| Cline | `~/.cline/skills/` | `./.cline/skills/` |

*Note: Gemini CLI (v0.30+) and OpenAI Codex both read `~/.agents/skills/`. Gemini CLI also reads `~/.gemini/skills/` but gives `.agents/` higher precedence. We install to `~/.agents/skills/` only to avoid duplicate-skill conflicts.*

### Cloud Platform Support (claude.ai, Claude Desktop, ChatGPT)

For claude.ai, Claude Desktop, and ChatGPT, skills must be uploaded as ZIP files through their respective web UIs. The skill includes a "Package for Cloud Upload" capability that:
1. Prompts user for their SkillsMP API key (optional — SkillHub works without one)
2. Creates `config.json` with the embedded key
3. Validates frontmatter against the Agent Skills spec
4. Generates a ZIP file ready for upload

**Upload paths:**
- **claude.ai / Claude Desktop**: Settings → Capabilities → Upload Skill
- **ChatGPT**: Profile → Skills → New skill → Upload from your computer

ChatGPT Skills are in beta and available on Business, Enterprise, Edu, Teachers, and Healthcare plans. All three platforms follow the same [Agent Skills specification](https://agentskills.io/specification) for SKILL.md frontmatter.

The hybrid API key discovery checks:
1. `$SKILLSMP_API_KEY` environment variable (Claude Code)
2. `config.json` in skill directory (claude.ai/Claude Desktop/ChatGPT)
3. Source selection prompt: offer SkillsMP (with key), SkillHub (no key needed), or ClawHub (no key needed) as fallback

## Key Concepts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jacob-bd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
