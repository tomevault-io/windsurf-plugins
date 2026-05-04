---
trigger: always_on
description: > `AGENTS.md` and `GEMINI.md` are symlinks to this file for Codex CLI and Gemini CLI compatibility.
---

# claude-settings

> `AGENTS.md` and `GEMINI.md` are symlinks to this file for Codex CLI and Gemini CLI compatibility.

Multi-tool plugin marketplace. Each plugin under `plugins/` is independently installable on Claude Code, Codex CLI, Gemini CLI, and Cursor.

## Repo Structure

```
claude-settings/
  CLAUDE.md                              # this file (repo dev guide)
  AGENTS.md -> CLAUDE.md                 # Codex CLI reads this
  GEMINI.md -> CLAUDE.md                 # Gemini CLI reads this
  .claude/CLAUDE.md                      # user-facing global config (synced to ~/.claude/CLAUDE.md)
  .claude/settings.json                  # Claude Code settings
  .claude-plugin/marketplace.json        # Claude Code marketplace
  .agents/plugins/marketplace.json       # Codex CLI marketplace
  .cursor-plugin/marketplace.json        # Cursor marketplace
  .codex/config.toml                     # Codex CLI config
  .github/scripts/                       # repo maintenance scripts
    _helpers.sh                          # shared sync/zip functions
    sync-<vendor>-skills.sh              # per-vendor skill sync
    sync-versions.sh                     # version alignment
    release.sh                           # GitHub release creation
    validate_plugins.py                  # CI plugin validation
  plugins/
    <name>/
      .claude-plugin/plugin.json         # Claude Code manifest
      .codex-plugin/plugin.json          # Codex CLI manifest
      .cursor-plugin/plugin.json         # Cursor manifest
      gemini-extension.json              # Gemini CLI manifest
      skills/<skill>/SKILL.md            # universal across all tools
      agents/<agent>.md                  # Claude Code + Gemini + Cursor
      hooks/hooks.json + scripts/        # Claude Code + Gemini only
      commands/<cmd>.md                  # Claude Code only
```

## Cross-Tool Plugin Format Reference

### Plugin Manifests

Each plugin has 4 manifest files. Claude Code manifest is the source of truth; others are copies or subsets.

| Tool        | Path                                     | Format                                                                  |
| ----------- | ---------------------------------------- | ----------------------------------------------------------------------- |
| Claude Code | `.claude-plugin/plugin.json`             | JSON: name, version, description, author, homepage, repository, license |
| Codex CLI   | `.codex-plugin/plugin.json`              | JSON: same fields as Claude Code                                        |
| Cursor      | `.cursor-plugin/plugin.json`             | JSON: same fields as Claude Code                                        |
| Gemini CLI  | `gemini-extension.json` (at plugin root) | JSON: name, version, description only                                   |

Docs:

- Claude Code: https://code.claude.com/docs/en/plugins-reference
- Codex CLI: https://developers.openai.com/codex/plugins/build/
- Cursor: https://cursor.com/docs/reference/plugins
- Gemini CLI: https://geminicli.com/docs/extensions/reference/
- AGENTS.md spec: https://agents.md/
- Agent Skills spec: https://agentskills.io/specification

### Root Marketplace Files

| Tool        | Path                               | Notes                                                                           |
| ----------- | ---------------------------------- | ------------------------------------------------------------------------------- |
| Claude Code | `.claude-plugin/marketplace.json`  | local sources only (use sync scripts for external repos)                        |
| Codex CLI   | `.agents/plugins/marketplace.json` | local sources only; use `./`-prefixed `source.path` plus plugin `policy` fields |
| Cursor      | `.cursor-plugin/marketplace.json`  | local sources, needs `source` + `description`                                   |
| Gemini CLI  | none                               | per-plugin install: `gemini extensions install --path ./plugins/<name>`         |

#### Claude Code marketplace entry

```json
{
  "name": "<plugin-name>",
  "source": "./plugins/<plugin-name>",
  "description": "...",
  "version": "1.0.0",
  "keywords": ["keyword1", "keyword2"],
  "category": "<category>",
  "tags": ["tag1", "tag2"]
}
```

#### Codex CLI marketplace entry

```json
{
  "name": "<plugin-name>",
  "source": { "source": "local", "path": "./plugins/<plugin-name>" },
  "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
  "category": "<category>"
}
```

`source.path` must point to a local folder with a `./`-prefixed path relative to the marketplace root.

`policy.installation` values: `AVAILABLE`, `INSTALLED_BY_DEFAULT`, `NOT_AVAILABLE`.

`policy.authentication` controls whether auth happens on install or first use.

#### Codex CLI personal marketplace example

Use this only for generic Codex marketplace docs and maintainer examples. User-facing installation docs should stay specific to this repo's bundled marketplace.

```json
{
  "name": "personal-plugins",
  "interface": {
    "displayName": "Personal Plugins"
  },
  "plugins": [
    {
      "name": "my-plugin",
      "source": { "source": "local", "path": "./.codex/plugins/my-plugin" },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fcakyon/claude-codex-settings](https://github.com/fcakyon/claude-codex-settings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
