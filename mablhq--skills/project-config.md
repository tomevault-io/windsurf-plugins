---
trigger: always_on
description: This repo is the public home of mabl's agent skills. One repo, five install surfaces — all sharing **one plugin home, `plugins/mabl/`**. That directory holds the skills, the MCP config, and each surface's plugin manifest; there is exactly one copy of everything. The repo root holds only the per-surface marketplace files that point into it (plus the Copilot manifest, which has to live at the root — see below).
---

# CLAUDE.md

This repo is the public home of mabl's agent skills. One repo, five install surfaces — all sharing **one plugin home, `plugins/mabl/`**. That directory holds the skills, the MCP config, and each surface's plugin manifest; there is exactly one copy of everything. The repo root holds only the per-surface marketplace files that point into it (plus the Copilot manifest, which has to live at the root — see below).

Why a subdirectory and not the repo root? **Codex.** Its marketplace requires each plugin in a subdirectory (`codex plugin add` rejects the repo root as a plugin) and copies that subdir into its cache **without following symlinks** — so the skills and MCP config have to be real files inside the plugin dir, not links back to the root. Rather than keep a second copy in sync, `plugins/mabl/` *is* the home and the other four surfaces point at it.

- **OpenAI Codex plugin** (`mabl`) — manifest `plugins/mabl/.codex-plugin/plugin.json`, listed by `.agents/plugins/marketplace.json` (`source: ./plugins/mabl`). Reads `plugins/mabl/skills/` and `plugins/mabl/.mcp.json`. Codex reads the same `.mcp.json` shape as Claude (camelCase `mcpServers`, `type: http` remote servers with OAuth).
- **Claude Code plugin** (`mabl`) — manifest `plugins/mabl/.claude-plugin/plugin.json`; marketplace `.claude-plugin/marketplace.json` at the root with `source: ./plugins/mabl`. Claude reads skills and `.mcp.json` by convention at the plugin root (`plugins/mabl/`).
- **Cursor plugin** (`mabl`) — manifest `plugins/mabl/.cursor-plugin/plugin.json`; marketplace `.cursor-plugin/marketplace.json` at the root with `source: ./plugins/mabl`. Cursor reads MCP servers from `plugins/mabl/mcp.json` (note: not `.mcp.json` — Cursor only reads `mcp.json`).
- **GitHub Copilot / VS Code plugin** (`mabl`) — manifest is the **root `plugin.json`**, because VS Code's plugin loader checks for a manifest at the repo root. It points into the home: `"skills": "plugins/mabl/skills/"`, `"mcpServers": "plugins/mabl/.mcp.json"`.
- **`gh skill install` source** — skills discovered via the `skills/*/SKILL.md` convention, which `gh skill` finds even nested under a prefix (`plugins/mabl/skills/...`).

### Changing a skill? Bump the version and update the docs

When you change something a plugin consumer sees — a skill's behavior, a new or
removed skill, an MCP server — **bump `version`** (all manifests; see "Keep the
manifests and MCP files in sync" below), add a `CHANGELOG.md` entry (see
"Changelog" below), and refresh the skill's row in `README.md` if what it does
changed. Skip all three only for changes no consumer ever sees (internal
comments, CI-only tweaks).

### Keep the manifests and MCP files in sync

The four plugin manifests — `plugins/mabl/.claude-plugin/plugin.json`, `plugins/mabl/.cursor-plugin/plugin.json`, `plugins/mabl/.codex-plugin/plugin.json`, and the root `plugin.json` (Copilot) — describe the same plugin. When you bump `version` or change `name`/`description`/`author`, update **all four** (and the `version` in the three `marketplace.json` files, which isn't parity-checked). CI checks this parity.

The one remaining duplication is MCP config: `plugins/mabl/mcp.json` (Cursor) must be a byte-identical copy of `plugins/mabl/.mcp.json` (Claude/Copilot/Codex). Cursor refuses any other filename, so the two can't be collapsed — CI enforces they match.

CI validates the manifests and the MCP files (`.github/workflows/validate-plugin.yml`).

### Changelog

There's no separate release step here — merging to `main` ships the plugin.
So every PR that bumps `version` (see above) adds its own entry directly to
the top of `CHANGELOG.md`, no `[Unreleased]` staging section. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/):

```markdown
## [<version>] - <YYYY-MM-DD>
### Added
- Short, user-facing description of what changed and why it matters.
```

Use whichever categories apply — `Added`, `Changed`, `Fixed`, `Removed`,
`Security` — and only the ones you need.

Skip the entry only when a change has no effect on what a plugin consumer
sees (typo fixes in internal comments, CI-only changes, etc) — those PRs
don't bump `version` either.

## Writing PRs

This repo is public — external developers read our PRs. Write them short and human:

- Lead with what changed and why it matters, in plain sentences. The diff shows the rest.
- A few paragraphs, not a report. Skip the `### Testing` / `### Follow-up` headers and bullet dumps unless the PR is genuinely large.
- No AI boilerplate: no "This PR introduces...", no generated-by footer, no emoji section markers.
- Keep the one caveat that actually matters; drop the exhaustive list.

## Rules for every skill

### Skills must be self-contained


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mablhq/skills](https://github.com/mablhq/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
