---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A collection of agent skills for financial analysis and trading, following the [Agent Skills](https://agentskills.io) open standard. Skills are installable into Claude Code, Claude.ai, and other supported agents (Codex, Gemini CLI, GitHub Copilot, etc.).

## Repository structure

This repo is three things at once:
1. A **Claude Code plugin marketplace** (`.claude-plugin/marketplace.json` + `plugins/`)
2. An **Agent Skills** repository (the `SKILL.md` files inside `plugins/<group>/skills/`)
3. An **opencli plugin monorepo** (`opencli-plugin.json` at root + `opencli-plugins/`) — Node code for adapters that some skills depend on

Skills are organized into plugin groups by usage; opencli plugins are separate Node packages.

```
.claude-plugin/
  marketplace.json        # Marketplace definition — lists all 6 plugins
plugins/
  market-analysis/        # Stock analysis, earnings, correlations, options via yfinance
    plugin.json           # Plugin manifest for this group
    skills/
      <skill-name>/
        SKILL.md
        README.md
        references/
  social-readers/         # Social media research feeds (Twitter, Discord, LinkedIn, Telegram, YC)
    plugin.json
    skills/...
  data-providers/         # External API data (Adanos, Funda AI, Hormuz Strait, TradingView)
    plugin.json
    skills/...
  startup-tools/          # Startup analysis
    plugin.json
    skills/...
  ui-tools/               # Generative UI design system
    plugin.json
    skills/...
  skill-creator/          # Skill authoring, evaluation, and improvement
    plugin.json
    skills/...
opencli-plugin.json       # Top-level opencli MONOREPO manifest — declares sub-plugins
opencli-plugins/          # Source for opencli adapters (Node code, has tests)
  tradingview/            # TradingView desktop reader (drives the tradingview-reader skill)
    opencli-plugin.json   # Per-plugin manifest
    package.json          # Node package (type: module)
    *.js                  # one file per command (registers via cli({...}))
    lib/                  # shared helpers
    tests/                # node:test units
workspaces/               # Development workspaces (not distributed)
.agents/                  # Auto-generated mirror for agent distribution (do not edit directly)
.github/workflows/
  release-skills.yml      # Zips each skill and publishes as GitHub release on tag
  skill-lint.yml          # Lints all SKILL.md files
```

## How skills work

Each skill is a self-contained directory under `plugins/<group>/skills/`. The `SKILL.md` file is what Claude reads at runtime — it tells the model when to activate, what steps to follow, and where to find reference details.

### SKILL.md format

```markdown
---
name: skill-name
description: >
  Multi-line description that doubles as the trigger definition.
  Include specific phrases, keywords, and scenarios that should activate this skill.
---

# Skill Title

Step-by-step instructions organized as ## Step N sections.
Tables, code blocks, and formulas as needed.

## Reference Files

- `references/foo.md` — description
```

**Required frontmatter fields:** `name`, `description`

The `description` field is critical — it controls when the skill activates. Write it as a comprehensive trigger list, not a summary.

### Reference files

Markdown documents in `references/` containing detailed API references, code templates, formulas, or schema docs. The SKILL.md instructions tell the model to read specific reference files when needed, keeping the main instructions concise.

## Creating a new skill

1. Choose the appropriate plugin group (`market-analysis`, `social-readers`, `data-providers`, or `startup-tools`)
2. Create `plugins/<group>/skills/<skill-name>/` directory
3. Write `SKILL.md` with YAML frontmatter (`name`, `description`) and step-by-step instructions
4. Add reference files under `references/` for detailed API docs, code templates, or formulas that would bloat the main instructions
5. Add a `README.md` for the skill's GitHub page (description, triggers, platform, setup, reference file list)
6. Update the root `README.md` to list the new skill in the appropriate plugin group table
7. The skill will be auto-zipped and released on tag push via GitHub Actions

### Platform considerations

Skills that require shell access, network calls, or external binaries (e.g., twitter-cli, pip install) only work on **CLI-based agents** like Claude Code. They do **not** work on Claude.ai, which runs in a sandboxed environment that restricts network access and binaries.

Skills that only use Claude's built-in tools (e.g., `show_widget` for generative-ui) work on **Claude.ai**.

### Dynamic content with `!`command``

Skills can embed shell commands that Claude Code executes at skill invocation time, injecting the output inline. Use this for runtime environment checks (tool installation status, auth state, live data). Syntax: wrap in a fenced code block with `` !`command` ``.

Example — checking if a CLI tool is installed and authenticated:
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [himself65/finance-skills](https://github.com/himself65/finance-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
