---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **Skillpacks Marketplace** - a modular collection of 16 professional skillpacks providing 144 production-ready skills for Claude Code across AI/ML, Python engineering, game development, security, documentation, UX design, and solution architecture.

**Status**: v1.2.0 - Production ready, CC BY-SA 4.0 licensed, publicly available

## Architecture

### Marketplace Structure

```plaintext
skillpacks/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog defining all 15 plugins
├── plugins/                       # 15 independent plugin directories
│   ├── [plugin-name]/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Plugin metadata (name, version, description)
│   │   └── skills/
│   │       └── [skill-name]/
│   │           └── SKILL.md      # Skill implementation
└── docs/
    └── future-*.md               # Future planning docs
```

### Plugin Categories

1. **Python Engineering (Axiom faction)** - 3 plugins, 24 skills
   - `axiom-python-engineering` (10 skills)
   - `axiom-system-archaeologist` (5 skills - router + specialists)
   - `axiom-solution-architect` (9 skills - router + specialists, forward design)

2. **AI/ML (Yzmir faction)** - 8 plugins, 70 skills
   - `yzmir-ai-engineering-expert` (router)
   - `yzmir-pytorch-engineering`
   - `yzmir-training-optimization`
   - `yzmir-deep-rl`
   - `yzmir-neural-architectures`
   - `yzmir-llm-specialist`
   - `yzmir-ml-production`
   - `yzmir-simulation-foundations`

3. **Game Development (Bravos faction)** - 2 plugins, 20 skills
   - `bravos-simulation-tactics`
   - `bravos-systems-as-experience`

4. **UX Design (Lyra faction)** - 1 plugin, 11 skills
   - `lyra-ux-designer`

5. **Security (Ordis faction)** - 1 plugin, 9 skills
   - `ordis-security-architect`

6. **Documentation (Muna faction)** - 1 plugin, 9 skills
   - `muna-technical-writer`

### Slash Commands (Router Skills)

**IMPORTANT**: All router skills (`using-X` skills) are available as slash commands in `.claude/commands/` due to skill context limits.

To use a router skill, invoke it as a slash command:

```
/ai-engineering      # Routes to AI/ML skills
/system-archaeologist # Routes to architecture analysis
/solution-architect  # Routes to forward solution design
/deep-rl            # Routes to RL algorithms
/python-engineering  # Routes to Python skills
```

**Why slash commands?** Router skills exceeded the context budget for skill discovery. Slash commands provide:
- No context limits
- Explicit user invocation
- Faster loading
- Better control flow

See [.claude/SLASH_COMMANDS.md](.claude/SLASH_COMMANDS.md) for complete list of all 15 router commands.

### Skill File Format

Each `SKILL.md` follows this structure:

- **Front matter**: YAML with `name` and `description`
- **Content**: Expert-level guidance with examples, patterns, and anti-patterns
- Skills range from 200-2000 lines of production-ready content

### Router Patterns

Several plugins use "using-X" router skills that direct users to appropriate specialized skills. **These are now available as slash commands** (see Slash Commands section above):

- `axiom-system-archaeologist/using-system-archaeologist/SKILL.md` → `/system-archaeologist` - Routes to architecture analysis specialists
- `yzmir-ai-engineering-expert/using-ai-engineering/SKILL.md` → `/ai-engineering` - Routes to all AI/ML packs
- `yzmir-deep-rl/using-deep-rl/SKILL.md` → `/deep-rl` - Routes to 12 RL algorithm skills
- Similar routers exist for all 15 plugins (see `.claude/SLASH_COMMANDS.md` for complete list)

## Installation & Testing

### Add to Claude Code

```bash
# Add marketplace
/plugin marketplace add tachyon-beep/skillpacks

# Browse available
/plugin

# Install specific pack
/plugin install yzmir-deep-rl
```

### Development Testing

When testing changes to skills:

1. Make edits to `plugins/[plugin-name]/skills/[skill-name]/SKILL.md`
2. If plugin is installed, changes take effect immediately
3. Test by asking Claude to use the skill

### Local Development Installation

```bash
# From skillpacks directory
/plugin marketplace add .
```

## Version Management

### Plugin Versioning

Each plugin has independent versioning in `.claude-plugin/plugin.json`:

```json
{
  "name": "yzmir-deep-rl",
  "version": "1.0.0",
  "description": "...",
  "category": "ai-ml"
}
```

### Marketplace Versioning

The marketplace catalog (`.claude-plugin/marketplace.json`) coordinates all 13 plugins:

- Lists all plugins with their source paths
- Maintains marketplace metadata (version, homepage)
- Uses `"pluginRoot": "./plugins"` to locate plugin directories

## Working With This Repository

### Adding a New Skill

1. Navigate to appropriate plugin: `plugins/[plugin-name]/skills/`
2. Create directory: `mkdir new-skill-name`
3. Create skill file: `new-skill-name/SKILL.md`
4. Add YAML front matter:

   ```yaml
   ---
   name: new-skill-name
   description: Brief description for skill discovery
   ---
   ```

5. Write skill content following existing skill patterns
6. Test with Claude Code

### Creating a New Plugin


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tachyon-beep/skillpacks](https://github.com/tachyon-beep/skillpacks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
