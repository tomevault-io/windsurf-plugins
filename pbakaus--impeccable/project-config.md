---
trigger: always_on
description: The vocabulary you didn't know you needed. 1 skill, 18 commands, and curated anti-patterns for impeccable style. Works with Cursor, Claude Code, Gemini CLI, and Codex CLI.
---

# Impeccable

The vocabulary you didn't know you needed. 1 skill, 18 commands, and curated anti-patterns for impeccable style. Works with Cursor, Claude Code, Gemini CLI, and Codex CLI.

## Repository Purpose

Maintain a **single source of truth** for design-focused skills and commands, then automatically transform them into provider-specific formats. Each provider has different capabilities (frontmatter, arguments, modular files), so we use a build system to generate appropriate outputs.

## Architecture: Option A (Feature-Rich Source)

We use a **feature-rich source format** that gets transformed for each provider:

- **Source files** (`source/`): Full metadata with YAML frontmatter, args, descriptions
- **Build system** (`scripts/`): Transforms source → provider-specific formats
- **Distribution** (`dist/`): Committed output files for 4 providers

### Why Option A?

Cursor doesn't support frontmatter or arguments (lowest common denominator). Instead of limiting all providers, we:
1. Author with full metadata in source files
2. Generate full-featured versions for providers that support it (Claude Code, Gemini, Codex)
3. Generate downgraded versions for Cursor (strip frontmatter, rely on appending)

## Repository Structure

```
impeccable/
├── source/                      # EDIT THESE! Single source of truth
│   ├── commands/                # Command definitions with frontmatter
│   │   └── normalize.md
│   └── skills/                  # Skill definitions with frontmatter
│       └── impeccable/
├── dist/                        # Generated outputs (committed for users)
│   ├── cursor/                  # Commands + Agent Skills
│   │   └── .cursor/
│   │       ├── commands/*.md
│   │       └── skills/*/SKILL.md
│   ├── claude-code/             # Full featured
│   │   └── .claude/
│   │       ├── commands/*.md
│   │       └── skills/*/SKILL.md
│   ├── gemini/                  # TOML commands + modular skills
│   │   ├── .gemini/
│   │   │   └── commands/*.toml
│   │   ├── GEMINI.md
│   │   └── GEMINI.*.md
│   └── codex/                   # Custom prompts + Agent Skills
│       └── .codex/
│           ├── prompts/*.md
│           └── skills/*/SKILL.md
├── api/                         # Vercel Functions (production)
│   ├── skills.js                # GET /api/skills
│   ├── commands.js              # GET /api/commands
│   └── download/
│       ├── [type]/[provider]/[id].js   # Individual downloads
│       └── bundle/[provider].js        # Bundle downloads
├── public/                      # Website for impeccable.style
│   ├── index.html               # Main page
│   ├── css/                     # Modular CSS (9 files)
│   │   ├── main.css             # Entry point with imports
│   │   ├── tokens.css           # Design system
│   │   └── ...                  # Component styles
│   └── app.js                   # Vanilla JS
├── server/                      # Bun server (local dev only)
│   ├── index.js                 # Serves website + API routes
│   └── lib/
│       └── api-handlers.js      # Shared API logic (used by both server & functions)
├── scripts/                     # Build system (Bun)
│   ├── build.js                 # Main orchestrator
│   ├── lib/
│   │   ├── utils.js             # Shared utilities
│   │   ├── zip.js               # ZIP generation
│   │   └── transformers/        # Provider-specific transformers
│   │       ├── cursor.js
│   │       ├── claude-code.js
│   │       ├── gemini.js
│   │       └── codex.js
├── README.md                    # End user documentation
├── DEVELOP.md                   # Contributor documentation
└── package.json                 # Bun scripts
```

## Website (impeccable.style)

**Tech Stack:**
- Vanilla JavaScript (no frameworks)
- Modern CSS with Bun's bundler (nesting, OKLCH colors, @import)
- **Local Development**: Bun server with native routes (`server/index.js`)
- **Production**: Vercel Functions with Bun runtime (`/api` directory)
- Deployed on Vercel with Bun runtime

**Dual Setup:**
- `/api` directory contains individual Vercel Functions for production
- `/server` directory contains monolithic Bun server for local development
- `/server/lib/api-handlers.js` contains shared logic used by both
- Zero duplication: API functions and dev server import the same handlers

**Design:**
- Editorial precision aesthetic
- Cormorant Garamond (display) + Instrument Sans (body)
- OKLCH color space for vibrant, perceptually uniform colors
- Editorial sidebar layout (title left, content right)
- Modular CSS architecture (9 files)

**API Endpoints** (Vercel Functions):
- `/` - Homepage (static HTML)
- `/api/skills` - JSON list of all skills
- `/api/commands` - JSON list of all commands
- `/api/download/[type]/[provider]/[id]` - Individual file download
- `/api/download/bundle/[provider]` - ZIP bundle download

## Source File Format

### Commands (`source/commands/*.md`)

```yaml
---
name: command-name
description: Clear description of what this command does
args:
  - name: argname
    description: Argument description
    required: false
---

Command prompt here. Use {{argname}} placeholders for arguments.
```

### Skills (`source/skills/*.md`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pbakaus/impeccable](https://github.com/pbakaus/impeccable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
