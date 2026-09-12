---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Claudex is a **Claude Code marketplace** that distributes skills and hooks through a plugin-based architecture. This is NOT a traditional application codebase - it's a marketplace distribution system following Anthropic's official plugin schema.

### Architecture Model

```
marketplace.json (Single Source of Truth)
    ↓
23 Plugins (1 per skill)
    ↓
Each plugin: plugin.json + skills/ + README.md
```

**Key Principle**: The `.claude-plugin/marketplace.json` file is the **single source of truth** for all marketplace metadata. Each plugin has its own `.claude-plugin/plugin.json` for per-plugin metadata.

## Repository Structure

Follows Anthropic's official `anthropics/claude-code/plugins/` pattern:

```
claudex/
├── .claude-plugin/
│   └── marketplace.json           # SINGLE SOURCE OF TRUTH
├── scripts/
│   ├── validate-marketplace.py    # Schema validation
│   └── validate-skills.py         # Skill quality validation
├── plugins/                       # 23 plugins (1 per skill)
│   ├── accessibility-audit/
│   │   ├── skills/
│   │   │   └── accessibility-audit/
│   │   │       ├── SKILL.md
│   │   │       ├── README.md
│   │   │       └── CHANGELOG.md
│   │   └── README.md
│   ├── cc-insights/               # Includes hooks
│   │   ├── skills/
│   │   │   └── cc-insights/
│   │   ├── hooks/
│   │   │   ├── hooks.json
│   │   │   └── extract-explanatory-insights.sh
│   │   └── README.md
│   └── ... (21 more plugins)
└── archive/                       # Archived skills (not in marketplace)
```

### Plugin Directory Structure

**Required structure for every plugin:**
```
plugin-name/
├── skills/
│   └── skill-name/
│       └── SKILL.md       # Agent manifest (frontmatter + workflow)
└── README.md              # Plugin documentation
```

Note: Plugin metadata (name, description) lives in marketplace.json only. No per-plugin plugin.json files are needed.

**Optional directories within skill:**
```
skill-name/
├── data/              # Reference materials, insights
├── examples/          # Sample outputs
├── scripts/           # Implementation code (Python/Bash)
├── reference/         # Standards, best practices
└── requirements.txt   # Python dependencies (if applicable)
```

## Marketplace Integration

### marketplace.json Schema (Anthropic Pattern)

```json
{
  "name": "claudex",
  "owner": {
    "name": "Connor",
    "email": "noreply@claudex.dev"
  },
  "metadata": {
    "version": "6.2.0",
    "description": "Skills and hooks for Claude Code"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "description": "Plugin description",
      "source": "./plugins/plugin-name"
    }
  ]
}
```

**Auto-Discovery**: Claude Code automatically discovers components from the plugin's directory structure:
- `skills/` - Skill definitions (SKILL.md files)
- `hooks/` - Hook configurations (hooks.json)
- `agents/` - Agent definitions
- `commands/` - Custom commands

Do NOT add `strict`, `skills`, or `hooks` fields to marketplace entries - Claude Code auto-discovers these from directory structure.

### Adding a New Skill/Plugin

1. **Create plugin directory**:
   ```bash
   mkdir -p plugins/skill-name/skills/skill-name
   ```

2. **Create skill files**:
   - `skills/skill-name/SKILL.md` with frontmatter

3. **Create plugin README.md**

4. **Update marketplace.json**:
   - Add plugin entry to `plugins` array with `name`, `description`, and `source`

5. **Validate**:
   ```bash
   python3 scripts/validate-marketplace.py
   python3 scripts/validate-skills.py plugins/skill-name
   ```

## Automation

### Sensei Agent

The Sensei Agent automates skill improvements based on evaluation feedback. It is triggered via GitHub issues with the `agent-ready` label.

**Workflow:**
1. Create issue with evaluation feedback containing `**Skill**: \`skill-name\``
2. Add `agent-ready` label
3. Agent reads feedback, improves skill, creates PR
4. Label changes to `agent-completed` on success

**Files:**
- `.github/workflows/run-sensei-agent.yml` - GitHub Actions workflow
- `scripts/automation/sensei_agent.py` - Python agent script
- `docs/automation/sensei-agent.md` - Full documentation

**Safety:**
- Writes restricted to `plugins/*/skills/*` directory only
- Max 20 turns before automatic stop
- Remove `agent-ready` label to cancel

**Testing locally:**
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
python scripts/automation/sensei_agent.py \
  --issue 123 --repo owner/claudex --skill skill-name --dry-run
```

**Required secret:** `ANTHROPIC_API_KEY` must be configured in repository settings.

## Development Commands

### Validation

```bash
# Validate marketplace.json schema
python3 scripts/validate-marketplace.py

# Validate all skills against Anthropic spec
python3 scripts/validate-skills.py plugins/

# Validate specific skill
python3 scripts/validate-skills.py plugins/skill-name/skills/skill-name
```

**Expected output:** ✅ passed, ⚠️ warnings, or ❌ errors. Exit code 0 = valid.

### Git Operations

```bash
# Create feature branch
git checkout -b feature/add-skill-name

# Conventional commit
git commit -m "feat: Add skill-name plugin"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cskiro/claudex](https://github.com/cskiro/claudex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
