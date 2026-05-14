---
trigger: always_on
description: Generates boilerplate Three.js scene code with customizable options.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Instructions

Be concise and to the point. Sacrifice grammar and punctuation for clarity and conciseness.

## Repository Overview

This is a **dual-purpose repository**:
1. **Development Workspace** - Create and manage Claude skills in `.claude/skills/`
2. **Plugin Marketplace** - Distribute skills as plugins via `.claude/plugins/` and `.claude-plugin/marketplace.json`

Skills are modular, self-contained packages that extend Claude's capabilities by providing specialized knowledge, workflows, and tools. Each skill acts as an "onboarding guide" for specific domains or tasks.

The repository focuses on creating comprehensive skills for a design agency's web development toolstack, particularly 3D/WebGL and animation technologies, then packaging them as plugins with slash commands and specialized agents.

## Official Documentation Reference

**IMPORTANT**: All skills in this repository MUST follow Claude's official skill standards.

📚 **Official Claude Skill Documentation**: https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview

Key resources:
- Skill Overview: https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview
- Creating Skills: https://docs.claude.com/en/docs/agents-and-tools/agent-skills/creating-skills
- Skill Structure: https://docs.claude.com/en/docs/agents-and-tools/agent-skills/skill-structure

When creating or modifying skills, **always consult the official documentation** to ensure compliance with the latest standards.

## Quick Reference

**Skill Development Workflows**:

```bash
# Create a new skill
.claude/skills/skill-creator/scripts/init_skill.py my-new-skill --path .claude/skills

# Validate a skill
.claude/skills/skill-creator/scripts/quick_validate.py .claude/skills/my-new-skill

# Package a skill (automatically validates first)
.claude/skills/skill-creator/scripts/package_skill.py .claude/skills/my-new-skill

# Validate all skills
for skill in .claude/skills/*/; do
  .claude/skills/skill-creator/scripts/quick_validate.py "$skill"
done

# Make a script executable (required after creating new scripts)
chmod +x .claude/skills/my-skill/scripts/my_script.py
```

**Plugin Marketplace Workflows**:

```bash
# Generate individual plugin from skill
./scripts/marketplace/generate_plugin.py threejs-webgl

# Generate all individual plugins
./scripts/marketplace/generate_plugin.py --all

# Generate category bundle plugin
./scripts/marketplace/generate_bundle.py core-3d-animation

# Generate all bundles
./scripts/marketplace/generate_bundle.py --all

# Generate marketplace.json manifest
./scripts/marketplace/generate_marketplace.py

# Validate marketplace and all plugins
./scripts/marketplace/validate_marketplace.py
```

**User Installation Workflows**:

```bash
# Add marketplace to Claude Code
/plugin marketplace add freshtechbro/claudedesignskills

# Install individual plugin
/plugin install threejs-webgl

# Install bundle plugin
/plugin install core-3d-animation
```

**Skill Activation**:
- Skills auto-activate when Claude detects trigger keywords from the description
- Mention specific technologies: "Three.js", "GSAP", "React Three Fiber"
- Describe tasks: "create 3D scene", "scroll-driven animations", "physics-based motion"

## Repository Structure

```
claudeskills/
├── CLAUDE.md                      # This file - repository guidance
├── MARKETPLACE.md                 # Plugin marketplace documentation
├── README.md                      # Public-facing documentation
│
├── .claude/
│   ├── skills/                    # Skill development workspace
│   │   ├── skill-creator/         # Meta-skill for creating skills
│   │   ├── threejs-webgl/         # ✅ Complete (22 total)
│   │   ├── gsap-scrolltrigger/
│   │   └── ...                    # (20 more skills)
│   │
│   └── plugins/                   # Generated plugins for distribution
│       ├── individual/            # 22 individual skill plugins
│       │   ├── threejs-webgl/     # Each includes skills/, commands/, agents/
│       │   ├── gsap-scrolltrigger/
│       │   └── ...
│       │
│       └── bundles/               # 5 category bundle plugins
│           ├── core-3d-animation/
│           ├── extended-3d-scroll/
│           ├── animation-components/
│           ├── authoring-motion/
│           └── meta-skills/
│
├── .claude-plugin/
│   └── marketplace.json           # Marketplace manifest (27 plugins)
│
└── scripts/
    ├── skill-creator/             # (existing skill scripts)
    └── marketplace/               # Marketplace automation
        ├── generate_plugin.py     # Convert skill → plugin
        ├── generate_bundle.py     # Create category bundles
        ├── generate_marketplace.py # Build marketplace.json
        └── validate_marketplace.py # Validate all plugins
```

**Each skill directory contains**:
- `SKILL.md` - Main instructions
- `scripts/` - Automation utilities
- `references/` - Documentation
- `assets/` - Templates and starter projects

**Each plugin directory contains**:
- `.claude-plugin/plugin.json` - Plugin manifest
- `skills/<skill-name>/` - Skill content (copied from `.claude/skills/`)
- `commands/` - Slash commands (1-3 per plugin)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freshtechbro/claudedesignskills](https://github.com/freshtechbro/claudedesignskills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
