---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code marketplace plugin providing AI-powered content generation skills. Skills use Gemini Web API (reverse-engineered) for text/image generation and Chrome CDP for browser automation.

## Architecture

Skills are organized into three plugin categories in `marketplace.json`:

```
skills/
├── [content-skills]           # Content generation and publishing
│   ├── canghe-xhs-images/          # Xiaohongshu infographic series (1-10 images)
│   ├── canghe-cover-image/         # Article cover images (2.35:1 aspect)
│   ├── canghe-slide-deck/          # Presentation slides with outlines
│   ├── canghe-article-illustrator/ # Smart illustration placement
│   ├── canghe-comic/               # Knowledge comics (Logicomix/Ohmsha style)
│   ├── canghe-post-to-x/           # X/Twitter posting automation
│   └── canghe-post-to-wechat/      # WeChat Official Account posting
│
├── [ai-generation-skills]     # AI-powered generation backends
│   └── canghe-danger-gemini-web/   # Gemini API wrapper (text + image gen)
│
└── [utility-skills]           # Utility tools for content processing
    ├── canghe-danger-x-to-markdown/ # X/Twitter content to markdown
    └── canghe-compress-image/      # Image compression
```

**Plugin Categories**:
| Category | Description |
|----------|-------------|
| `content-skills` | Skills that generate or publish content (images, slides, comics, posts) |
| `ai-generation-skills` | Backend skills providing AI generation capabilities |
| `utility-skills` | Helper tools for content processing (conversion, compression) |

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
npx -y bun skills/canghe-danger-gemini-web/scripts/main.ts "Hello"

# Image generation
npx -y bun skills/canghe-danger-gemini-web/scripts/main.ts --prompt "A cat" --image cat.png

# From prompt files
npx -y bun skills/canghe-danger-gemini-web/scripts/main.ts --promptfiles system.md content.md --image out.png
```

## Key Dependencies

- **Bun**: TypeScript runtime (via `npx -y bun`)
- **Chrome**: Required for `canghe-danger-gemini-web` auth and `canghe-post-to-x` automation
- **No npm packages**: Self-contained TypeScript, no external dependencies

## Authentication

`canghe-danger-gemini-web` uses browser cookies for Google auth:
- First run opens Chrome for login
- Cookies cached in data directory
- Force refresh: `--login` flag

## Plugin Configuration

`.claude-plugin/marketplace.json` defines plugin metadata and skill paths. Version follows semver.

## Skill Loading Rules

**IMPORTANT**: When working in this project, follow these rules:

| Rule | Description |
|------|-------------|
| **Load project skills first** | MUST load all skills from `skills/` directory in current project. Project skills take priority over system/user-level skills with same name. |
| **Default image generation** | When image generation is needed, use `skills/canghe-image-gen/SKILL.md` by default (unless user specifies otherwise). |

**Loading Priority** (highest → lowest):
1. Current project `skills/` directory
2. User-level skills (`$HOME/.canghe-skills/`)
3. System-level skills

## Release Process

**IMPORTANT**: When user requests release/发布/push, ALWAYS use `/release-skills` workflow.

**Never skip**:
1. `CHANGELOG.md` + `CHANGELOG.zh.md` - Both must be updated
2. `marketplace.json` version bump
3. `README.md` + `README.zh.md` if applicable
4. All files committed together before tag

## Adding New Skills

**IMPORTANT**: All skills MUST use `canghe-` prefix to avoid conflicts when users import this plugin.

**REQUIRED READING**: Before creating a new skill, read the official [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

### Key Requirements from Official Best Practices

| Requirement | Details |
|-------------|---------|
| **Concise is key** | Claude is smart—only add context it doesn't have. Challenge each token. |
| **name field** | Max 64 chars, lowercase letters/numbers/hyphens only, no "anthropic"/"claude" |
| **description field** | Max 1024 chars, non-empty, MUST be third person, include what + when to use |
| **SKILL.md body** | Keep under 500 lines; use separate files for additional content |
| **Naming convention** | Gerund form preferred (e.g., `processing-pdfs`), but `canghe-` prefix required here |
| **References** | Keep one level deep from SKILL.md; avoid nested references |
| **No time-sensitive info** | Avoid dates/versions that become outdated |

### Steps

1. Create `skills/canghe-<name>/SKILL.md` with YAML front matter
   - Directory name: `canghe-<name>`
   - SKILL.md `name` field: `canghe-<name>`
2. Add TypeScript in `skills/canghe-<name>/scripts/`
3. Add prompt templates in `skills/canghe-<name>/prompts/` if needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freestylefly/canghe-skills](https://github.com/freestylefly/canghe-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
