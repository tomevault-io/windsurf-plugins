---
trigger: always_on
description: **chewiesoft-marketplace** — a dual-platform plugin marketplace owned by **j0ruge**. Distributes skills and commands for **Claude Code** and **Cursor**, developed exclusively by the owner.
---

# CLAUDE.md

## Project Overview

**chewiesoft-marketplace** — a dual-platform plugin marketplace owned by **j0ruge**. Distributes skills and commands for **Claude Code** and **Cursor**, developed exclusively by the owner.

For Claude Code, this repository follows the [Claude Code Plugin Marketplace](https://code.claude.com/docs/en/plugins-reference) specification.
For Cursor, skills are installed via the interactive `install.py` script at the repo root.

## Marketplace Structure

```
skills/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog (lists all plugins)
├── plugins/                      # Plugin directories (one per plugin)
│   └── .gitkeep
├── .claude/
│   ├── commands/                 # Speckit commands (SDD dev tools, NOT marketplace content)
│   └── settings.json             # Team distribution config
├── .specify/                     # SDD framework (dev tooling, NOT marketplace content)
├── install.py                    # Interactive installer (Claude Code + Cursor)
└── CLAUDE.md
```

### Key files

- **`.claude-plugin/marketplace.json`** — The marketplace catalog. Lists all available plugins with their source paths. This is the entry point Claude Code reads when registering the marketplace.
- **`plugins/`** — Each subdirectory is a self-contained plugin with its own `.claude-plugin/plugin.json` manifest.
- **`.claude/settings.json`** — Registers this repo as a known marketplace for team auto-discovery.
- **`install.py`** — Interactive installer. Handles both Claude Code (shows installation commands) and Cursor (copies and adapts skills to `~/.cursor/skills/` or `.cursor/skills/`).

### Platform Compatibility

Each `plugin.json` includes a `platforms` field declaring which platforms support the plugin:
- `["claude-code", "cursor"]` — available on both platforms
- `["claude-code"]` — Claude Code only (e.g., `statusline`)

## Adding a New Plugin

1. Create a directory under `plugins/`:
   ```
   plugins/my-plugin/
   ├── .claude-plugin/
   │   └── plugin.json        # Plugin manifest (name, description, author, etc.)
   ├── commands/               # Command definitions (.md files)
   ├── skills/                 # Skill definitions (SKILL.md files)
   ├── agents/                 # Agent definitions (.md files) [optional, auto-discovered]
   └── hooks/hooks.json        # Hook configuration [optional, auto-discovered]
   ```

2. Create `plugins/my-plugin/.claude-plugin/plugin.json`:
   ```json
   {
     "name": "my-plugin",
     "description": "What this plugin does",
     "author": {
       "name": "Chewiesoft"
     },
     "version": "1.0.0",
     "keywords": ["relevant", "tags"],
     "platforms": ["claude-code", "cursor"],
     "commands": "./commands",
     "skills": "./skills"
   }
   ```
   Do **not** declare `agents` or `hooks` as a directory: `"agents": "./agents"` and `"hooks": "./hooks"`
   fail `claude plugin install` with `agents: Invalid input` / `hooks: Invalid input` (validated 2026-09-30).
   `agents/*.md` and `hooks/hooks.json` are discovered automatically. If you must declare them, the valid
   forms are a list of files (`"agents": ["./agents/reviewer.md"]`) and the file path
   (`"hooks": "./hooks/hooks.json"`). Check with `claude plugin validate plugins/<name>` before publishing.

   Set `"platforms": ["claude-code"]` if the plugin uses Claude Code-specific APIs (e.g., status line, MCP tools) and cannot work in Cursor.

3. Register the plugin in `.claude-plugin/marketplace.json`:
   ```json
   {
     "plugins": [
       {
         "name": "my-plugin",
         "source": "./plugins/my-plugin",
         "version": "1.0.0",
         "description": "Brief plugin description shown in Discover tab",
         "category": "development",
         "keywords": ["relevant", "tags"],
         "platforms": ["claude-code", "cursor"]
       }
     ]
   }
   ```

4. If the plugin supports Cursor, add a mapping entry to `CURSOR_SKILL_MAP` in `install.py` so the installer knows how to copy and adapt the content.
   - For skills: add an entry with `"source_type": "skill"` pointing to the skill directory.
   - For commands: add an entry with `"source_type": "command"` and a `"cursor_description"` for Cursor's trigger-based discovery.

## Skill description guidelines

The `description` field in SKILL.md frontmatter and `marketplace.json` is the
primary trigger mechanism. When the combined skill listing exceeds Claude
Code's `skillListingBudgetFraction` (default 1% of context), descriptions are
**dropped silently** and the skills lose their trigger. Keep descriptions tight
so this never happens.

### DO
- Keep `description` ≤ 350 characters (hard cap: 500). Fits one line in
  `/skills` output.
- Open with one sentence describing the function — not "Use this skill when…".
- Follow with 1–2 distinctive capabilities that disambiguate from neighbors.
- End with a single trigger list of ≤ 8 keywords, prefixed `Triggers — `.
- Pick **one language** per description (English preferred for cross-team
  reach). Body of the skill can stay bilingual.
- Mirror the exact same description in `SKILL.md` frontmatter, `plugin.json` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [j0ruge/skills](https://github.com/j0ruge/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
