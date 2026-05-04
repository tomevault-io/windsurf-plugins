---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code marketplace plugin providing AI-powered content generation skills. Skills use Gemini Web API (reverse-engineered) for text/image generation and Chrome CDP for browser automation.

## Architecture

Skills are organized into one plugin category in `marketplace.json`:

```
skills/
└── [image-generation-skills]  # AI image generation
    ├── baoyu-image-gen/            # Core image generation (OpenAI/Google/DashScope/Replicate)
    ├── baoyu-xhs-images/          # Xiaohongshu infographic series (1-10 images)
    ├── baoyu-cover-image/         # Article cover images (2.35:1 aspect)
    ├── baoyu-infographic/         # Professional infographics (21 layouts, 20 styles)
    ├── baoyu-slide-deck/          # Presentation slides (8-30 slides)
    ├── baoyu-comic/               # Knowledge comics (multi-page)
    ├── baoyu-article-illustrator/ # Article illustrations
    └── baoyu-danger-gemini-web/   # Backup image generation (reverse API)
```

**Plugin Categories**:
| Category | Description |
|----------|-------------|
| `image-generation-skills` | AI image generation skills for various content types |

Each skill contains:
- `SKILL.md` - YAML front matter (name, description) + documentation
- `scripts/` - TypeScript implementations
- `prompts/system.md` - AI generation guidelines (optional)

## Running Skills

All scripts run via Bun (no build step):

```bash
npx -y bun skills/<skill>/scripts/main.ts [options]
```

Examples:
```bash
# Text generation
npx -y bun skills/baoyu-danger-gemini-web/scripts/main.ts "Hello"

# Image generation
npx -y bun skills/baoyu-danger-gemini-web/scripts/main.ts --prompt "A cat" --image cat.png

# From prompt files
npx -y bun skills/baoyu-danger-gemini-web/scripts/main.ts --promptfiles system.md content.md --image out.png
```

## Key Dependencies

- **Bun**: TypeScript runtime (via `npx -y bun`)
- **API Keys**: OpenAI / Google / DashScope / Replicate (configured via `.env`)
- **No npm packages**: Self-contained TypeScript, no external dependencies

## Plugin Configuration

`.claude-plugin/marketplace.json` defines plugin metadata and skill paths. Version follows semver.

## Skill Loading Rules

**IMPORTANT**: When working in this project, follow these rules:

| Rule | Description |
|------|-------------|
| **Load project skills first** | MUST load all skills from `skills/` directory in current project. Project skills take priority over system/user-level skills with same name. |
| **Default image generation** | When image generation is needed, use `skills/baoyu-image-gen/SKILL.md` by default (unless user specifies otherwise). |

**Loading Priority** (highest → lowest):
1. Current project `skills/` directory
2. User-level skills (`$HOME/.baoyu-skills/`)
3. System-level skills

## Release Process

**IMPORTANT**: When user requests release/发布/push, ALWAYS use `/release-skills` workflow.

**Never skip**:
1. `CHANGELOG.md` + `CHANGELOG.zh.md` - Both must be updated
2. `marketplace.json` version bump
3. `README.md` + `README.zh.md` if applicable
4. All files committed together before tag

## Adding New Skills

**IMPORTANT**: All skills MUST use `baoyu-` prefix to avoid conflicts when users import this plugin.

**REQUIRED READING**: Before creating a new skill, read the official [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

### Key Requirements from Official Best Practices

| Requirement | Details |
|-------------|---------|
| **Concise is key** | Claude is smart—only add context it doesn't have. Challenge each token. |
| **name field** | Max 64 chars, lowercase letters/numbers/hyphens only, no "anthropic"/"claude" |
| **description field** | Max 1024 chars, non-empty, MUST be third person, include what + when to use |
| **SKILL.md body** | Keep under 500 lines; use separate files for additional content |
| **Naming convention** | Gerund form preferred (e.g., `processing-pdfs`), but `baoyu-` prefix required here |
| **References** | Keep one level deep from SKILL.md; avoid nested references |
| **No time-sensitive info** | Avoid dates/versions that become outdated |

### Steps

1. Create `skills/baoyu-<name>/SKILL.md` with YAML front matter
   - Directory name: `baoyu-<name>`
   - SKILL.md `name` field: `baoyu-<name>`
2. Add TypeScript in `skills/baoyu-<name>/scripts/`
3. Add prompt templates in `skills/baoyu-<name>/prompts/` if needed
4. **Choose the appropriate category** and register in `marketplace.json`:
   - `image-generation-skills`: For AI image generation skills
   - If none fit, create a new category with descriptive name
5. **Add Script Directory section** to SKILL.md (see template below)

### Choosing a Category

| If your skill... | Use category |
|------------------|--------------|
| Generates images (infographics, comics, slides) | `image-generation-skills` |
| Provides AI generation backend | `image-generation-skills` |

**Creating a new category**: If the skill doesn't fit existing categories, add a new plugin object to `marketplace.json` with:
- `name`: Descriptive kebab-case name (e.g., `analytics-skills`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XY121718/baoyu-skills](https://github.com/XY121718/baoyu-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
